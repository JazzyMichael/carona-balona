<script>
import { onMount } from 'svelte';
import { WiredCard } from 'wired-card';
import { WiredLink } from 'wired-link';
import LineChart from './LineChart.svelte';

const url = 'https://pomber.github.io/covid19/timeseries.json';
let data;
let lastUpdate;
let totalConfirmed;
let totalDeaths;

const addCommas = (num) => {
	return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
}

const getData = async () => {
	let data;
	if (localStorage.getItem('data')) {
		data = JSON.parse(localStorage.getItem('data'));
		localStorage.removeItem('data');
		console.log('cached', data);
	} else {
		const res = await fetch(url);
		data = await res.json();
		localStorage.setItem('data', JSON.stringify(data));
		console.log('fetched', data);
	}
	return data;
}

onMount(async () => {
	data = await getData();

	lastUpdate = Object.values(data).reduce((acc, val) => {
		return val[val.length - 1].date > acc ? val[val.length - 1].date : acc;
	}, '');
	
	totalConfirmed = addCommas(Object.values(data).reduce((acc, val) => {
		return val[val.length - 1].confirmed + acc;
	}, 0));

	totalDeaths = addCommas(Object.values(data).reduce((acc, val) => {
		return val[val.length - 1].deaths + acc;
	}, 0));
});


</script>

<div class="header">
	<h2 style="color: #ff3e00; font-weight: 400;">🍺 CARONA VIRUS 🤢</h2>
	
	{#if lastUpdate}
		<p style="font-size: 0.9em">Last Updated: {lastUpdate}</p>
	{:else}
		<p style="font-size: 0.9em">   </p>
	{/if}

	<wired-card elevation="3" fill="#1a83e6" style="padding: 2em">
		<span style="color: white">{totalConfirmed} Cases</span>
	</wired-card>

	<wired-card elevation="3" fill="#ff3e00" style="padding: 2em">
		<span style="color: white">{totalDeaths} Deaths</span>
	</wired-card>
</div>

<LineChart countries={data}></LineChart>

<wired-card elevation="3" style="margin: 4em auto; display: block; text-align: center;">
	<p>*Data comes from the Novel Coronavirus COVID-19 Data Repository by Johns Hopkins Center for Systems Science and Engineering. It is then cleansed and converted to JSON using an open-source project developed by <wired-link href="https://pomb.us/" target="_blank">"Pomber"</wired-link> before being interpretted by this Svelte App! Source code coming soon.</p>
	<p><wired-link href="https://www.arcgis.com/apps/opsdashboard/index.html#/bda7594740fd40299423467b48e9ecf6" target="_blank">Click Here</wired-link> to view the official Johns Hopkins Dashboard.</p>
</wired-card>

<style>
	.header {
		text-align: center;
		margin: 1em auto 3em;
	}

	.chart-filters {
 		display: flex;
		justify-content: space-around;
		align-items: center;
		flex-wrap: wrap;
		width: 95%;
		max-width: 666px;
		margin: 1em auto;
	}

	.chart-card {
		width: 95%;
		margin: auto;
		display: block;
		box-sizing: border-box;
	}

	/* only chart-specific styles below */
	.chart {
		height: 400px;
		padding: 1em 0 1.5em 36px;
	}

	.grid-line {
		position: relative;
		display: block;
	}

	.grid-line.horizontal {
		width: calc(100% + 2em);
		left: -2em;
		border-bottom: 1px dashed #ccc;
	}

	.grid-line span {
		position: absolute;
		left: 0;
		bottom: 2px;
		font-family: sans-serif;
		font-size: 14px;
		color: #999;
	}

	.x-label {
		position: absolute;
		width: 4em;
		left: -2em;
		bottom: -22px;
		font-family: sans-serif;
		font-size: 14px;
		color: #999;
		text-align: center;
	}

	path.data {
		stroke: rgba(0,0,0,0.2);
		stroke-linejoin: round;
		stroke-linecap: round;
		stroke-width: 1px;
		fill: none;
	}

	.highlight {
		stroke: #ff3e00;
		fill: none;
		stroke-width: 2;
	}

	.annotation {
		position: absolute;
		white-space: nowrap;
		bottom: 1em;
		line-height: 1.2;
		background-color: rgba(255,255,255,0.9);
		padding: 0.2em 0.4em;
		border-radius: 2px;
	}

	.annotation-point {
		position: absolute;
		width: 10px;
		height: 10px;
		background-color: #ff3e00;
		border-radius: 50%;
		transform: translate(-50%,-50%);
	}

	.annotation strong {
		display: block;
		font-size: 20px;
	}

	.annotation span {
		display: block;
		font-size: 14px;
	}
	/* only chart styles above */
</style>
