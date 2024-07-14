<template>
	<main id="Home-page">
	  <h1>DASHBOARD</h1>
	  <TimerTes/>
	  <p>Real-Time Data Charts</p>
	  <div class="charts">
		<div class="chart-1">
		  <h3>Monitoring Bagian Atas</h3>
		  <TemperatureAtas/>
		  <HumidityAtas/>
		</div>
	  </div>
	  <div class="charts">
		<div class="chart-2">
		  <h3>Monitoring Bagian Bawah</h3>
		  <temperatureBawah/>
		  <HumidityBawah/>
		</div>
	  </div>
	  <div class="tombol_3">
		<button @click="StartData">Mulai</button> 
	  </div>
	  <div class="tombol">
		<button @click="CopyData">Save Data</button> 
	  </div>
	  <div class="tombol_2">
		<button @click="ResetData">Reset Data</button> 
	  </div>
	</main>
  </template>
  
  <script>
  import HumidityAtas from '../chart/HumidityAtas.vue';
  import TemperatureAtas from '../chart/TemperatureAtas.vue';
  import TemperatureBawah from '../chart/TemperatureBawah.vue';
  import HumidityBawah from '../chart/HumidityBawah.vue';
  import TimerTes from '../timer/TimerTes.vue';
  import axios from 'axios';
  
  export default {
	name: 'Home',
	components: {TemperatureAtas, HumidityAtas, TemperatureBawah, HumidityBawah, TimerTes},
	methods: {
	  CopyData() {
		axios.post('http://localhost:3000/copy-data')
		.then(response => {
		  if (response.data.success) {
			alert('Data telah Disimpan');
		  } else {
			alert('Data gagal disimpan')
		  }
		  })
		  .catch(error => {
			console.error('Error:', error);
			alert('Ada Error');
		  });
		},
	  ResetData() {
		axios.post('http://localhost:3000/reset-data')
		.then(response => {
		  if (response.data.success) {
			alert('Data telah direset');
		  } else {
			alert('Data gagal direset')
		  }
		  })
		  .catch(error => {
			console.error('Error:', error);
			alert('Ada Error');
		  });
		},
	  StartData() {
		axios.post('http://localhost:3000/start-data', {
        	command: 'start'
      	  }, {
        	headers: {
          		'Content-Type': 'application/json'
        	}
      	})
      	.then(response => {
        	console.log('Response from backend:', response.data);
      		})
      	.catch(error => {
        	console.error('Error:', error);
      	});
	   },
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
  
  .tombol{
	background: #ffffff;
	padding: 10px;
	border-radius: 0px;
	box-shadow: - 10px 10px rgba(255, 0, 0, 0.08);
	width: 6.5%;
	outline: 1px solid rgb(13, 6, 52);
	margin-bottom: 10px;
	text-align: center;
  }
  
  .tombol_2{
	background: #ffffff;
	padding: 10px;
	border-radius: 0px;
	box-shadow: - 10px 10px rgba(255, 0, 0, 0.08);
	width: 6.5%;
	outline: 1px solid rgb(13, 6, 52);
	text-align: center;
  }

  .tombol_3{
	background: #ffffff;
	padding: 10px;
	border-radius: 0px;
	box-shadow: - 10px 10px rgba(255, 0, 0, 0.08);
	width: 6.5%;
	outline: 1px solid rgb(13, 6, 52);
	margin-bottom: 10px;
	text-align: center;
  }
  
  
  </style>