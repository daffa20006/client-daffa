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
      const socket = io('http://localhost:1000');
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
                      text: 'Temperature (°C)',
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
        if (temperatureBawahChart) {
          // Merge new data with existing data
          const existingLabels2 = temperatureBawahChart.data.labels as string[];
          const existingSensor3Data = temperatureBawahChart.data.datasets[0].data as number[];
          const existingSensor4Data = temperatureBawahChart.data.datasets[1].data as number[];
          const existingSensor5Data = temperatureBawahChart.data.datasets[2].data as number[];
          const existingSensor6Data = temperatureBawahChart.data.datasets[3].data as number[];
          
          const newLabels2 = newData.timestamps.filter(temp => !existingLabels2.includes(temp));
          const newSensor3Data = newData.sensor3_temperature.filter(temp => !existingSensor3Data.includes(temp));
          const newSensor4Data = newData.sensor4_temperature.filter(temp => !existingSensor4Data.includes(temp));
          const newSensor5Data = newData.sensor5_temperature.filter(temp => !existingSensor5Data.includes(temp));
          const newSensor6Data = newData.sensor6_temperature.filter(temp => !existingSensor6Data.includes(temp));

          const allLabels2 = existingLabels2.concat(newLabels2);
          const allSensor3Data = existingSensor3Data.concat(newSensor3Data);
          const allSensor4Data = existingSensor4Data.concat(newSensor4Data);
          const allSensor5Data = existingSensor5Data.concat(newSensor5Data);
          const allSensor6Data = existingSensor6Data.concat(newSensor6Data);
  
          temperatureBawahChart.data.labels = allLabels2;
          temperatureBawahChart.data.datasets[0].data = allSensor3Data;
          temperatureBawahChart.data.datasets[1].data = allSensor4Data;
          temperatureBawahChart.data.datasets[2].data = allSensor5Data;
          temperatureBawahChart.data.datasets[3].data = allSensor6Data;
          temperatureBawahChart.update();
        }
      };
  
      onMounted(async () => {
        try {
          const response = await axios.get('http://localhost:1000/api/copy-data');
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
  