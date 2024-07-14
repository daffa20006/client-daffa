<template>
  <main id="Home-page">
    <h1>HISTORY DATA</h1>
    <div class="charts">
      <div class="chart-1">
        <h3>Monitoring Bagian Atas</h3>
        <TemperatureAtasHistory/>
        <HumidityAtasHistory/>
      </div>
    </div>
    <div class="charts">
      <div class="chart-2">
        <h3>Monitoring Bagian Bawah</h3>
        <temperatureBawahHistory/>
        <HumidityBawahHistory/>
      </div>
    </div>
    <div class="tombol_3">
      <button @click="DeleteData">Delete History</button> 
    </div>
  </main>
</template>

<script>
import HumidityAtasHistory from '../chart_history/HumidityAtasHistory.vue';
import TemperatureAtasHistory from '../chart_history/TemperatureAtasHistory.vue';
import TemperatureBawahHistory from '../chart_history/TemperatureBawahHistory.vue';
import HumidityBawahHistory from '../chart_history/HumidityBawahHistory.vue';
import axios from 'axios';

export default {
  name: 'Home',
  components: {TemperatureAtasHistory, HumidityAtasHistory, TemperatureBawahHistory, HumidityBawahHistory},
  methods: {
    DeleteData() {
      axios.post('http://localhost:3000/delete-data')
      .then(response => {
        if (response.data.success) {
          alert('Data telah dihapus');
        } else {
          alert('Data gagal dihapus')
        }
        })
        .catch(error => {
          console.error('Error:', error);
          alert('Ada Error');
        });
      }
    }
  }
</script>



<style>
#about,
    TemperatureBawah, TemperatureAtas, HumidityBawah, HumidityAtas {
  display: flex;
  flex-wrap: wrap;
}
.chart-container {
  width: 50%;
  height: 400px;
  position: relative
}

.charts{
  display: grid;
  grid-template-columns: 2fr 1fr;
  grid-gap: 10px;
  width: 145%;
  padding: 15px;
  padding-top: 10;
  padding-left: 0;
}

.chart-1{
  background: #fff;
  padding: 10px;
  border-radius: 5px;
  box-shadow: - 10px 25px rgba(0, 0, 0, 0.08);
}

.chart-2{
  background: #fff;
  padding: 10px;
  border-radius: 5px;
  box-shadow: - 10px 25px rgba(0, 0, 0, 0.08);
}

.tombol_3{
	background: #ffffff;
	padding: 10px;
	border-radius: 0px;
	box-shadow: - 10px 10px rgba(255, 0, 0, 0.08);
	width: 8%;
	outline: 1px solid rgb(13, 6, 52);
  }
</style>