<template>
    <div>
      <canvas id="HumidityBawah" ref="chartRef" width="400" height="100"></canvas>
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
    name: 'HumidityBawah',
    setup() {
      const chartRef = ref<HTMLCanvasElement | null>(null);
      const socket = io('http://localhost:3000');
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
                    backgroundColor: '#009FB7',
                    borderColor: '#009FB7',
                    data: data.sensor3_humidity,
                  },
                  {
                    label: 'Sensor 2',
                    backgroundColor: '#04F06A',
                    borderColor: '#04F06A',
                    data: data.sensor4_humidity,
                  },
                  {
                    label: 'Sensor 3',
                    backgroundColor: '#FED766',
                    borderColor: '#FED766',
                    data: data.sensor5_humidity,
                  },
                  {
                    label: 'Sensor 4',
                    backgroundColor: '#F8C7CC',
                    borderColor: '#F8C7CC',
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
        if (humidityBawahChart && humidityBawahChart.data) {
          // Merge new data with existing data
          let combinedData = (humidityBawahChart.data.labels as string []).map((timestamp, index) => ({
            timestamp,
            sensor3_humidity: humidityBawahChart?.data.datasets[0].data[index] as number,
            sensor4_humidity: humidityBawahChart?.data.datasets[1].data[index] as number,
            sensor5_humidity: humidityBawahChart?.data.datasets[2].data[index] as number,
            sensor6_humidity: humidityBawahChart?.data.datasets[3].data[index] as number,
          }));

          const newCombinedData = newData.timestamps.map((timestamp: string, index: number) => ({
          timestamp,
          sensor3_humidity: newData.sensor3_humidity[index],
          sensor4_humidity: newData.sensor4_humidity[index],
          sensor5_humidity: newData.sensor5_humidity[index],
          sensor6_humidity: newData.sensor6_humidity[index],
        }));   
        
          combinedData = combinedData.concat(newCombinedData);

          // Urutkan data berdasarkan timestamp dan hapus duplikasi
          combinedData = combinedData
            .sort((a, b) => new Date(a.timestamp).getTime() - new Date(b.timestamp).getTime())
            .filter((item, index, self) =>
              index === self.findIndex((t) => t.timestamp === item.timestamp)
            );

          const sortedLabelsH2 = combinedData.map(item => item.timestamp);
          const sortedSensor3HData = combinedData.map(item => item.sensor3_humidity);
          const sortedSensor4HData = combinedData.map(item => item.sensor4_humidity);
          const sortedSensor5HData = combinedData.map(item => item.sensor5_humidity);
          const sortedSensor6HData = combinedData.map(item => item.sensor6_humidity);
  
          humidityBawahChart.data.labels = sortedLabelsH2;
          humidityBawahChart.data.datasets[0].data = sortedSensor3HData;
          humidityBawahChart.data.datasets[1].data = sortedSensor4HData;
          humidityBawahChart.data.datasets[2].data = sortedSensor5HData;
          humidityBawahChart.data.datasets[3].data = sortedSensor6HData;
          humidityBawahChart.update();
        }
      };
  
      onMounted(async () => {
        try {
          const response = await axios.get('http://localhost:3000/api/sensor-data');
          const data = {
            timestamps: response.data.map((row: any) => row.timestamp),
            sensor3_humidity: response.data.map((row: any) => row.sensor3_humidity),
            sensor4_humidity: response.data.map((row: any) => row.sensor4_humidity),
            sensor5_humidity: response.data.map((row: any) => row.sensor5_humidity),
            sensor6_humidity: response.data.map((row: any) => row.sensor6_humidity),

          };
          console.log('Fetched data from API:', data);
          createChart(data);
  
          socket.on('sensorData', (data: any) => {
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
  