<template>
  <div>
    <canvas id="TemperatureAtasHistory" ref="chartRef" width="400 px" height="100 px"></canvas>
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
  name: 'TemperatureAtasHistory',
  setup() {
    const chartRef = ref<HTMLCanvasElement | null>(null);
    const socket = io('http://localhost:3000');
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
        if (temperatureAtasChart && temperatureAtasChart.data) {
          // Gabungkan data lama dan baru
            let combinedData = (temperatureAtasChart.data.labels as string[]).map((timestamp, index) => ({
            timestamp,
            sensor1_temperature: temperatureAtasChart?.data.datasets[0].data[index] as number,
            sensor2_temperature: temperatureAtasChart?.data.datasets[1].data[index] as number,
        }));

    const newCombinedData = newData.timestamps.map((timestamp: string, index: number) => ({
      timestamp,
      sensor1_temperature: newData.sensor1_temperature[index],
      sensor2_temperature: newData.sensor2_temperature[index],
    }));

    combinedData = combinedData.concat(newCombinedData);

    // Urutkan data berdasarkan timestamp dan hapus duplikasi
    combinedData = combinedData
      .sort((a, b) => new Date(a.timestamp).getTime() - new Date(b.timestamp).getTime())
      .filter((item, index, self) =>
        index === self.findIndex((t) => t.timestamp === item.timestamp)
      );

    // Pisahkan kembali data ke dalam array yang berbeda
    const sortedLabels = combinedData.map(item => item.timestamp);
    const sortedSensor1Data = combinedData.map(item => item.sensor1_temperature);
    const sortedSensor2Data = combinedData.map(item => item.sensor2_temperature);

    // Update data chart
    temperatureAtasChart.data.labels = sortedLabels;
    temperatureAtasChart.data.datasets[0].data = sortedSensor1Data;
    temperatureAtasChart.data.datasets[1].data = sortedSensor2Data;

    // Update chart
    temperatureAtasChart.update();
  } else {
    console.error('temperatureAtasChart is undefined');
  }
};

    onMounted(async () => {
      try {
        const response = await axios.get('http://localhost:3000/api/copy-data');
        const data = {
          timestamps: response.data.map((row: any) => row.timestamp),
          sensor1_temperature: response.data.map((row: any) => row.sensor1_temperature),
          sensor2_temperature: response.data.map((row: any) => row.sensor2_temperature),
        };
        console.log('Fetched data from API:', data);
        createChart(data);

        socket.on('sensorDataHistory', (data: any) => {
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
