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
      const socket = io('http://localhost:3000');
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
                    backgroundColor: '#009FB7',
                    borderColor: '#009FB7',
                    data: data.sensor1_humidity,
                  },
                  {
                    label: 'Sensor 2',
                    backgroundColor: '#04F06A',
                    borderColor: '#04F06A',
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
                  ticks: {
                  stepSize: 5,
                  },
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
        if (humidityAtasChart && humidityAtasChart.data) {
          let combinedData = (humidityAtasChart.data.labels as string[]).map((timestamp, index) => ({
            timestamp,
            sensor1_humidity: humidityAtasChart?.data.datasets[0].data[index] as number,
            sensor2_humidity: humidityAtasChart?.data.datasets[1].data[index] as number,
          }));

        const newCombinedData = newData.timestamps.map((timestamp: string, index: number) => ({
          timestamp,
          sensor1_humidity: newData.sensor1_humidity[index],
          sensor2_humidity: newData.sensor2_humidity[index],
        }));

        combinedData = combinedData.concat(newCombinedData);

        combinedData = combinedData
          .sort((a,b) => new Date(a.timestamp).getTime() - new Date(b.timestamp).getTime())
          .filter((item, index, self) =>
            index === self.findIndex((t) => t.timestamp === item.timestamp)
          );

          const sortedLabelsH = combinedData.map(item => item.timestamp);
          const sortedSensor1DataH = combinedData.map(item => item.sensor1_humidity);
          const sortedSensor2DataH = combinedData.map(item => item.sensor2_humidity);
  
          humidityAtasChart.data.labels = sortedLabelsH;
          humidityAtasChart.data.datasets[0].data = sortedSensor1DataH;
          humidityAtasChart.data.datasets[1].data = sortedSensor2DataH;

          humidityAtasChart.update();
        }
      };
  
      onMounted(async () => {
        try {
          const response = await axios.get('http://localhost:3000/api/sensor-data');
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
  