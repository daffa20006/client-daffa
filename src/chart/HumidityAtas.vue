<template>
    <div>
      <canvas id="HumidityAtas" ref="chartRef" width="300" height="100"></canvas>
    </div>
  </template>
  
  <script lang="ts">
  import axios from 'axios';
  import { ref, onMounted } from 'vue';
  import { Chart, registerables } from 'chart.js';
  import 'chartjs-adapter-date-fns'; // Import the date-fns adapter
  import io from 'socket.io-client';
  // @ts-ignore
  import annotationPlugin from 'chartjs-plugin-annotation';
  
  Chart.register(...registerables, annotationPlugin);
  
  export default {
    name: 'HumidityAtas',
    setup() {
      const chartRef = ref<HTMLCanvasElement | null>(null);
      const socket = io('http://localhost:1000');
      let humidityAtasChart: Chart | undefined;
  
      const createChart = (data: any) => {
        console.log('Creating chart with data:', data);
        if (chartRef.value) {
          const ctx = chartRef.value.getContext('2d');
          if (ctx) {
            humidityAtasChart = new Chart(ctx, {
              type: 'line',
              data: {
                labels: data.timestamps,
                datasets: [
                  {
                    label: 'Sensor 1',
                    backgroundColor: (context) => {
                    const value = context.raw as number;
                    return value > 65 ? '#DE1A1A' : '#009FB7';
                  },
                  borderColor: (context) => {
                    const value = context.raw as number;
                    return value > 65 ? '#DE1A1A' : '#009FB7';
                  },
                    data: data.sensor1_humidity,
                  },
                  {
                    label: 'Sensor 2',
                    backgroundColor: (context) => {
                    const value = context.raw as number;
                    return value > 65 ? '#DE1A1A' : '#04F06A';
                  },
                  borderColor: (context) => {
                    const value = context.raw as number;
                    return value > 65 ? '#DE1A1A' : '#04F06A';
                  },
                    data: data.sensor2_humidity,
                  },
                ],
              },
              options: {
                responsive: true,
                aspectRatio: 3,
                scales: {
                  x: {
                    type: 'time',
                    time: {
                      unit: 'minute',
                      tooltipFormat: 'yyyy-MM-dd HH:mm',
                      displayFormats: {
                        minute: 'HH:mm',
                      },
                    },
             //     ticks: {
             //     stepSize: 5,
             //     },
                    title: {
                      display: true,
                      text: 'Waktu',
                    },
                  },
                  y: {
                    beginAtZero: true,
                    title: {
                      display: true,
                      text: 'Kelembapan (%RH)',
                    },
                  },
                },
                plugins: {
                annotation: {
                  annotations: {
                    threshold: {
                      type: 'line',
                      yMin: 65,
                      yMax: 65,
                      borderColor: '#DE1A1A',
                      borderWidth: 2,
                      borderDash: [6, 6],
                      label: {
                        content: 'Threshold 65°C',
                        display: true,
                        position: 'end',
                      },
                    },
                  },
                },
              },
              },
            });
          } else {
            console.error('Failed to get context for chart');
          }
        } else {
          console.error('chartRef.value is null');
        }
      };
  
      const updateChart = (newData: any) => {
        console.log('Updating chart with data:', newData);
        if (humidityAtasChart) {
          // Merge new data with existing data
          const existingLabelsH = humidityAtasChart.data.labels as string[];
          const existingSensor1DataH = humidityAtasChart.data.datasets[0].data as number[];
          const existingSensor2DataH = humidityAtasChart.data.datasets[1].data as number[];

          const newLabelsH = newData.timestamps.filter(hum => !existingLabelsH.includes(hum));
          const newSensor1DataH = newData.sensor1_humidity.filter(hum => !existingSensor1DataH.includes(hum));        
          const newSensor2DataH = newData.sensor2_humidity.filter(hum => !existingSensor2DataH.includes(hum));

          const allLabelsH = existingLabelsH.concat(newLabelsH);
          const allSensor1DataH = existingSensor1DataH.concat(newSensor1DataH);
          const allSensor2DataH = existingSensor2DataH.concat(newSensor2DataH);
  
          humidityAtasChart.data.labels = allLabelsH;
          humidityAtasChart.data.datasets[0].data = allSensor1DataH;
          humidityAtasChart.data.datasets[1].data = allSensor2DataH;
          humidityAtasChart.update();
        }
      };
  
      onMounted(async () => {
        try {
          const response = await axios.get('http://localhost:1000/api/sensor-data');
          const data = {
            timestamps: response.data.map((row: any) => row.timestamp),
            sensor1_humidity: response.data.map((row: any) => row.sensor1_humidity),
            sensor2_humidity: response.data.map((row: any) => row.sensor2_humidity),
          };
          console.log('Fetched data from API:', data);
          createChart(data);
  
          socket.on('sensorData', (data: any) => {
            const updatedDataH = {
              timestamps: Array.isArray(data) ? data.map((row: any) => row.timestamp) : [data.timestamp],
              sensor1_humidity: Array.isArray(data) ? data.map((row: any) => row.sensor1_humidity) : [data.sensor1_humidity],
              sensor2_humidity: Array.isArray(data) ? data.map((row: any) => row.sensor2_humidity) : [data.sensor2_humidity],
            };
            console.log('Received new data from socket:', updatedDataH);
            updateChart(updatedDataH);
          });
        } catch (error) {
          console.error('Error fetching data:', error);
        }
      });
  
      return {
        chartRef,
      };
    },
  };
  </script>
  