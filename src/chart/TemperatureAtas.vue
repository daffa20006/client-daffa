<template>
  <div>
    <canvas id="TemperatureAtas" ref="chartRef" width="300 px" height="100 px"></canvas>
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
  name: 'TemperatureAtas',
  setup() {
    const chartRef = ref<HTMLCanvasElement | null>(null);
    const socket = io('http://localhost:1000');
    let temperatureAtasChart: Chart | undefined;

    const createChart = (data: any) => {
      console.log('Creating chart with data:', data);
      if (chartRef.value) {
        const ctx = chartRef.value.getContext('2d');
        if (ctx) {
          temperatureAtasChart = new Chart(ctx, {
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
                  data: data.sensor1_temperature,
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
                  data: data.sensor2_temperature,
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
      if (temperatureAtasChart && temperatureAtasChart.data) {
        const existingLabels = temperatureAtasChart.data.labels as string[];
        const existingSensor1Data = temperatureAtasChart.data.datasets[0].data as number[];
        const existingSensor2Data = temperatureAtasChart.data.datasets[1].data as number[];

        // Filter data baru untuk memastikan tidak ada duplikasi
        const newLabelsToAdd = newData.timestamps.filter(label => !existingLabels.includes(label));
        const newSensor1DataToAdd = newData.sensor1_temperature.filter(temp => !existingSensor1Data.includes(temp));
        const newSensor2DataToAdd = newData.sensor2_temperature.filter(temp => !existingSensor2Data.includes(temp));

        // Gabungkan data baru dengan data yang ada
        const allLabels = existingLabels.concat(newLabelsToAdd);
        const allSensor1Data = existingSensor1Data.concat(newSensor1DataToAdd);
        const allSensor2Data = existingSensor2Data.concat(newSensor2DataToAdd);

        // Update data chart
        temperatureAtasChart.data.labels = allLabels;
        temperatureAtasChart.data.datasets[0].data = allSensor1Data;
        temperatureAtasChart.data.datasets[1].data = allSensor2Data;

        // Update chart
        temperatureAtasChart.update();
      }
    };

    onMounted(async () => {
      try {
        const response = await axios.get('http://localhost:1000/api/sensor-data');
        const data = {
          timestamps: response.data.map((row: any) => row.timestamp),
          sensor1_temperature: response.data.map((row: any) => row.sensor1_temperature),
          sensor2_temperature: response.data.map((row: any) => row.sensor2_temperature),
        };
        console.log('Fetched data from API:', data);
        createChart(data);

        socket.on('sensorData', (data: any) => {
          const updatedData = {
            timestamps: Array.isArray(data) ? data.map((row: any) => row.timestamp) : [data.timestamp],
            sensor1_temperature: Array.isArray(data) ? data.map((row: any) => row.sensor1_temperature) : [data.sensor1_temperature],
            sensor2_temperature: Array.isArray(data) ? data.map((row: any) => row.sensor2_temperature) : [data.sensor2_temperature],
          };
          console.log('Received new data from socket:', updatedData);
          updateChart(updatedData);
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