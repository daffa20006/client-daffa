<template>
    <div>
      <canvas id="HumidityBawahHistory" ref="chartRef" width="400" height="100"></canvas>
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
    name: 'HumidityBawahHistory',
    setup() {
      const chartRef = ref<HTMLCanvasElement | null>(null);
      const socket = io('http://localhost:1000');
      let humidityBawahChart: Chart | undefined;
  
      const createChart = (data: any) => {
        console.log('Creating chart with data:', data);
        if (chartRef.value) {
          const ctx = chartRef.value.getContext('2d');
          if (ctx) {
            humidityBawahChart = new Chart(ctx, {
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
                    data: data.sensor3_humidity,
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
                    data: data.sensor4_humidity,
                  },
                  {
                    label: 'Sensor 3',
                    backgroundColor: (context) => {
                    const value = context.raw as number;
                    return value > 65 ? '#DE1A1A' : '#FED766';
                  },
                  borderColor: (context) => {
                    const value = context.raw as number;
                    return value > 65 ? '#DE1A1A' : '#FED766';
                  },
                    data: data.sensor5_humidity,
                  },
                  {
                    label: 'Sensor 4',
                    backgroundColor: (context) => {
                    const value = context.raw as number;
                    return value > 65 ? '#DE1A1A' : '#F8C7CC';
                  },
                  borderColor: (context) => {
                    const value = context.raw as number;
                    return value > 65 ? '#DE1A1A' : '#F8C7CC';
                  },
                    data: data.sensor6_humidity,
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
        if (humidityBawahChart) {
          // Merge new data with existing data
          const existingLabelsH2 = humidityBawahChart.data.labels as string[];
          const existingSensor3HData = humidityBawahChart.data.datasets[0].data as number[];
          const existingSensor4HData = humidityBawahChart.data.datasets[1].data as number[];
          const existingSensor5HData = humidityBawahChart.data.datasets[2].data as number[];
          const existingSensor6HData = humidityBawahChart.data.datasets[3].data as number[];

          const newLabelsH2 = newData.timestamps.filter(hum => !existingLabelsH2.includes(hum));
          const newSensor3HData = newData.sensor3_humidity.filter(hum => !existingSensor3HData.includes(hum));
          const newSensor4HData = newData.sensor4_humidity.filter(hum => !existingSensor4HData.includes(hum));
          const newSensor5HData = newData.sensor5_humidity.filter(hum => !existingSensor5HData.includes(hum));   
          const newSensor6HData = newData.sensor6_humidity.filter(hum => !existingSensor6HData.includes(hum));

          const allLabelsH2 = existingLabelsH2.concat(newLabelsH2);
          const allSensor3HData = existingSensor3HData.concat(newSensor3HData);
          const allSensor4HData = existingSensor4HData.concat(newSensor4HData);
          const allSensor5HData = existingSensor5HData.concat(newSensor5HData);
          const allSensor6HData = existingSensor6HData.concat(newSensor6HData);
  
          humidityBawahChart.data.labels = allLabelsH2;
          humidityBawahChart.data.datasets[0].data = allSensor3HData;
          humidityBawahChart.data.datasets[1].data = allSensor4HData;
          humidityBawahChart.data.datasets[2].data = allSensor5HData;
          humidityBawahChart.data.datasets[3].data = allSensor6HData;
          humidityBawahChart.update();
        }
      };
  
      onMounted(async () => {
        try {
          const response = await axios.get('http://localhost:1000/api/copy-data');
          const data = {
            timestamps: response.data.map((row: any) => row.timestamp),
            sensor3_humidity: response.data.map((row: any) => row.sensor3_humidity),
            sensor4_humidity: response.data.map((row: any) => row.sensor4_humidity),
            sensor5_humidity: response.data.map((row: any) => row.sensor5_humidity),
            sensor6_humidity: response.data.map((row: any) => row.sensor6_humidity),

          };
          console.log('Fetched data from API:', data);
          createChart(data);
  
          socket.on('sensorDataHistory', (data: any) => {
            const updatedData2H = {
              timestamps: Array.isArray(data) ? data.map((row: any) => row.timestamp) : [data.timestamp],
              sensor3_humidity: Array.isArray(data) ? data.map((row: any) => row.sensor3_humidity) : [data.sensor3_humidity],
              sensor4_humidity: Array.isArray(data) ? data.map((row: any) => row.sensor4_humidity) : [data.sensor4_humidity],
              sensor5_humidity: Array.isArray(data) ? data.map((row: any) => row.sensor5_humidity) : [data.sensor5_humidity],
              sensor6_humidity: Array.isArray(data) ? data.map((row: any) => row.sensor6_humidity) : [data.sensor6_humidity],
            };
            console.log('Received new data from socket:', updatedData2H);
            updateChart(updatedData2H);
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
  