<template>
    <div>
      <canvas id="TemperatureBawahHistory" ref="chartRef" width="400" height="100"></canvas>
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
    name: 'TemperatureBawahHistory',
    setup() {
      const chartRef = ref<HTMLCanvasElement | null>(null);
      const socket = io('http://localhost:3000');
      let temperatureBawahChart: Chart | undefined;
  
      const createChart = (data: any) => {
        console.log('Creating chart with data:', data);
        if (chartRef.value) {
          const ctx = chartRef.value.getContext('2d');
          if (ctx) {
            temperatureBawahChart = new Chart(ctx, {
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
                    data: data.sensor3_temperature,
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
                    data: data.sensor4_temperature,
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
                    data: data.sensor5_temperature,
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
                    data: data.sensor6_temperature,
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
                      text: 'Temperature (°C)',
                    },
                  },
                },
                plugins: {
                  annotation: {
                    annotations: {
                      threshold: {
                        type: 'line',
                        yMin: 60,
                        yMax: 60,
                        borderColor: '#DE1A1A',
                        borderWidth: 2,
                        borderDash: [6, 6],
                        label: {
                          content: 'Threshold 60°C',
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
          if (temperatureBawahChart && temperatureBawahChart.data) {
            // Merge new data with existing data
            let combinedData = (temperatureBawahChart.data.labels as string []).map((timestamp, index) =>({
            timestamp,
            sensor3_temperature: temperatureBawahChart?.data.datasets[0].data[index] as number,
            sensor4_temperature: temperatureBawahChart?.data.datasets[1].data[index] as number,
            sensor5_temperature: temperatureBawahChart?.data.datasets[2].data[index] as number,
            sensor6_temperature: temperatureBawahChart?.data.datasets[3].data[index] as number,
          }));

        const newCombinedData = newData.timestamps.map((timestamp: string, index: number) => ({
          timestamp,
          sensor3_temperature: newData.sensor3_temperature[index],
          sensor4_temperature: newData.sensor4_temperature[index],
          sensor5_temperature: newData.sensor5_temperature[index],
          sensor6_temperature: newData.sensor6_temperature[index],
        }));

        combinedData = combinedData.concat(newCombinedData);

        // Urutkan data berdasarkan timestamp dan hapus duplikasi
        combinedData = combinedData
          .sort((a, b) => new Date(a.timestamp).getTime() - new Date(b.timestamp).getTime())
          .filter((item, index, self) =>
            index === self.findIndex((t) => t.timestamp === item.timestamp)
          );

          // Pisahkan kembali data ke dalam array yang berbeda
          const sortedLabels2 = combinedData.map(item => item.timestamp);
          const sortedSensor3Data = combinedData.map(item => item.sensor3_temperature);
          const sortedSensor4Data = combinedData.map(item => item.sensor4_temperature);
          const sortedSensor5Data = combinedData.map(item => item.sensor5_temperature);
          const sortedSensor6Data = combinedData.map(item => item.sensor6_temperature);  

          // Update data chart
          temperatureBawahChart.data.labels = sortedLabels2;
          temperatureBawahChart.data.datasets[0].data = sortedSensor3Data;
          temperatureBawahChart.data.datasets[1].data = sortedSensor4Data;
          temperatureBawahChart.data.datasets[2].data = sortedSensor5Data;
          temperatureBawahChart.data.datasets[3].data = sortedSensor6Data;

          temperatureBawahChart.update();
        }
      };
  
      onMounted(async () => {
        try {
          const response = await axios.get('http://localhost:3000/api/copy-data');
          const data = {
            timestamps: response.data.map((row: any) => row.timestamp),
            sensor3_temperature: response.data.map((row: any) => row.sensor3_temperature),
            sensor4_temperature: response.data.map((row: any) => row.sensor4_temperature),
            sensor5_temperature: response.data.map((row: any) => row.sensor5_temperature),
            sensor6_temperature: response.data.map((row: any) => row.sensor6_temperature),

          };
          console.log('Fetched data from API:', data);
          createChart(data);
  
          socket.on('sensorDataHistory', (data: any) => {
            const updatedData2 = {
              timestamps: Array.isArray(data) ? data.map((row: any) => row.timestamp) : [data.timestamp],
              sensor3_temperature: Array.isArray(data) ? data.map((row: any) => row.sensor3_temperature) : [data.sensor3_temperature],
              sensor4_temperature: Array.isArray(data) ? data.map((row: any) => row.sensor4_temperature) : [data.sensor4_temperature],
              sensor5_temperature: Array.isArray(data) ? data.map((row: any) => row.sensor5_temperature) : [data.sensor5_temperature],
              sensor6_temperature: Array.isArray(data) ? data.map((row: any) => row.sensor6_temperature) : [data.sensor6_temperature],
            };
            console.log('Received new data from socket:', updatedData2);
            updateChart(updatedData2);
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
  