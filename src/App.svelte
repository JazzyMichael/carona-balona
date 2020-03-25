<script>
	import { onMount } from 'svelte';
	import { WiredInput } from 'wired-input';
	import { WiredSlider } from 'wired-slider';
	import { WiredCard } from 'wired-card';
	import { WiredLink } from 'wired-link';
	import * as Pancake from '@sveltejs/pancake';

	const url = 'https://pomber.github.io/covid19/timeseries.json';
	let data;

	let confirmed = [];
	
	let x1 = +Infinity;
	let x2 = -Infinity
	let y1 = +Infinity;
	let y2 = -Infinity;

	let closest;
	let filter;

	let totalConfirmed;
	let totalDeaths;

	let range;
	let minRange = 0;
	let maxRange;

	onMount(async () => {
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

		for (let [name, arr] of Object.entries(data)) {
			confirmed.push({
				name,
				data: arr.map(obj => {
					const [year, month, day] = obj.date.split('-');
					const raw = new Date(year, month-1, day);
					const epoc = raw.getTime() / 1000;
					return {
						date: obj.date,
						x: epoc,
						y: obj.confirmed,
						deaths: obj.deaths,
						xDisplay: `${month}/${day}`
					}
				})
			});
		}

		confirmed.forEach(country => {
			country.data.forEach(d => {
				if (d.x < x1) x1 = d.x;
				if (d.x > x2) x2 = d.x;
				if (d.y < y1) y1 = d.y;
				if (d.y > y2) y2 = d.y;
			});
		});
		confirmed = confirmed;

		maxRange = confirmed[0].data.length - 5;
		
		totalConfirmed = confirmed.reduce((acc, val) => {
			return val.data[val.data.length - 1].y + acc;
		}, 0);

		totalDeaths = confirmed.reduce((acc, val) => {
			return val.data[val.data.length - 1].deaths + acc;
		}, 0);
	});

	$: regex = filter ? new RegExp(filter.value, 'i') : null;
	$: filtered = regex ? confirmed.filter(country => regex.test(country.name)) : confirmed;
	$: points = filtered.reduce((acc, country) => {
		return acc.concat(country.data.map(d => ({
			x: d.x,
			y: d.y,
			country,
			date: d.date,
			deaths: d.deaths,
			xDisplay: d.xDisplay
		})));
	}, []);;

	const handleFilter = () => {
		regex = filter ? new RegExp(filter.value, 'i') : null;
	}

	const handleSlide = () => {
		let startingIndex = range.value || 0;
		x1 = confirmed[0].data[startingIndex].x;
	}
</script>

<div class="header">
	<h2>🍺 Carona Virus 🤢</h2>
	<wired-card elevation="3" style="padding: 1em 1.5em">
		<h3>Confirmed Cases: {totalConfirmed}</h3>
		<h3>Deaths: {totalDeaths}</h3>
		<p>Last Updated: {new Date(+x2 * 1000).toLocaleString('en').split(',')[0]}</p>
	</wired-card>
</div>
<br>
<p style="text-align: center; margin-bottom: 0; color: gray;">Zoom</p>
<wired-slider style="display: block; margin: 0 auto 1em;" value="0" min={minRange} max={maxRange} bind:this={range} on:change={handleSlide}></wired-slider>

<wired-input class="input" type="text" bind:this={filter} placeholder="Filter" on:input={handleFilter}></wired-input>
<br>
<wired-card elevation="3" style="width: 100%; box-sizing: border-box; max-width: 1234px; margin: auto;">
<div class="chart">
	<Pancake.Chart {x1} {x2} y1={y1} y2={y2}>
		<Pancake.Grid horizontal count={6} let:value>
			<div class="grid-line horizontal"><span>{value}</span></div>
		</Pancake.Grid>

		<Pancake.Grid vertical count={3} let:value>
			<span class="x-label">{new Date(+value * 1000).toLocaleString('en', { month: 'long' })}</span>
		</Pancake.Grid>

		<Pancake.Svg>
			{#each filtered as country}
				<Pancake.SvgLine data={country.data} let:d>
					<path class="data" {d}></path>
				</Pancake.SvgLine>
			{/each}

			{#if closest}
				<Pancake.SvgLine data={closest.country.data} let:d>
					<path class="highlight" {d}></path>
				</Pancake.SvgLine>
			{/if}
		</Pancake.Svg>

		{#if closest}
			<Pancake.Point x={closest.x} y={closest.y}>
				<span class="annotation-point"></span>
				<div class="annotation" style="transform: translate(-{100 * ((closest.x - x1) / (x2 - x1))}%,0)">
					<strong>{closest.country.name}</strong>
					<span>{closest.date}</span>
					<span>{closest.y} cases | {closest.deaths} deaths</span>
				</div>
			</Pancake.Point>
		{/if}

		<Pancake.Quadtree data={points} bind:closest/>
	</Pancake.Chart>
</div>
</wired-card>

<wired-card elevation="3" style="margin: 4em auto; text-align: center;">
	<p>*Data comes from the Novel Coronavirus COVID-19 Data Repository by Johns Hopkins Center for Systems Science and Engineering. It is then cleansed and converted to JSON using an open-source project developed by <wired-link href="https://https://pomb.us/" target="_blank">"Pomber"</wired-link> before being interpretted by this Svelte App! Source code coming soon.</p>
	<p><wired-link href="https://www.arcgis.com/apps/opsdashboard/index.html#/bda7594740fd40299423467b48e9ecf6" target="_blank">Click Here</wired-link> to view the official Johns Hopkins Dashboard.</p>
</wired-card>

<style>
	.header {
		text-align: center;
		margin: 1em auto;
	}

	h2 {
		color: #ff3e00;
		text-transform: uppercase;
		font-weight: 300;
	}

	.input {
		display: block;
		margin: 1em auto;
		width: 85%;
		max-width: 250px;
	}

	.chart {
		height: 400px;
		padding: 1em 0 1.5em 36px;
	}

	input {
		font-size: inherit;
		font-family: inherit;
		padding: 0.5em;
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
</style>