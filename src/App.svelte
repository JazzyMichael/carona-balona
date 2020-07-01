<script>
import { onMount } from 'svelte';
import { WiredCard } from 'wired-card';
import { WiredButton } from 'wired-button';
import { WiredToggle } from 'wired-toggle';
import { WiredLink } from 'wired-link';
import LineChart from './LineChart.svelte';
import AreaChart from './AreaChart.svelte';
import Top10 from './Top10.svelte';
import Header from './Header.svelte';
import Footer from './Footer.svelte';

const url = 'https://pomber.github.io/covid19/timeseries.json';
let data;
let lastUpdate;
let totalConfirmed;
let totalDeaths;
let selectedCountry = {
	name: 'US',
	dailyCases: [],
	dailyDeaths: [],
	casesTicks: [],
	deathsTicks: []
};

const format = (num = '', str = '') => {
	const value = num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
	const descriptor = num && str ? ` ${str}` : '';
	return value + descriptor;
}

function selectCountry(name) {
	if (!name) name = selectedCountry.name;
	selectedCountry.dailyCases = data[name].map((obj, index) => {
		const [year, month, day] = obj.date.split('-');
		const raw = new Date(year, month-1, day);
        const epoch = raw.getTime() / 1000;
		return {
			y: index === 0 ? obj.confirmed : Math.max(obj.confirmed - data[name][index-1].confirmed, 0),
			x: epoch,
			date: obj.date,
			confirmed: obj.confirmed,
			xDisplay: `${month}/${day}`
		};
	}).filter(day => day.x > 1582500000);

	selectedCountry.dailyDeaths = data[name].map((obj, index) => {
		const [year, month, day] = obj.date.split('-');
		const raw = new Date(year, month-1, day);
        const epoch = raw.getTime() / 1000;
		return {
			y: index === 0 ? obj.deaths : Math.max(obj.deaths - data[name][index-1].deaths, 0),
			x: epoch,
			date: obj.date,
			deaths: obj.deaths,
			xDisplay: `${month}/${day}`
		};
	}).filter(day => day.x > 1582500000);

	const highestNewCases = selectedCountry.dailyCases.reduce((acc, val) => Math.max(acc, val.y), 0);
	const highestNewDeaths = selectedCountry.dailyDeaths.reduce((acc, val) => Math.max(acc, val.y), 0);

	selectedCountry.casesTicks = [1];
	selectedCountry.deathsTicks = [1];

	if (highestNewCases < 10000) selectedCountry.casesTicks.push(5000);
	for (let i = 1; i * 10000 < highestNewCases + 10000; i++) {
		selectedCountry.casesTicks.push(i * 10000);
	}

	if (highestNewDeaths < 1000) selectedCountry.deathsTicks.push(500);
	for (let i = 1; i * 1000 < highestNewDeaths + 1000; i++) {
		selectedCountry.deathsTicks.push(i * 1000);
	}
};

onMount(async () => {
	data = await fetch(url).then(res => res.json());
	selectCountry('US');

	lastUpdate = Object.values(data).reduce((acc, val) => {
		return val[val.length - 1].date > acc ? val[val.length - 1].date : acc;
	}, '');
	
	totalConfirmed = Object.values(data).reduce((acc, val) => {
		return val[val.length - 1].confirmed + acc;
	}, 0);

	totalDeaths = Object.values(data).reduce((acc, val) => {
		return val[val.length - 1].deaths + acc;
	}, 0);

	const theme = localStorage.getItem('theme');
	document.body.classList.add(!theme ? 'light' : theme);
});

</script>

<svelte:head>
	<style>
		.light {
			--bg: #f5f5f5;
			--text: black;
			--svg-filter: invert(0%);
		}

		.dark {
			--bg: black;
			--text: #f5f5f5;
			--svg-filter: invert(100%);
		}

		html, body {
			overflow-x: hidden;
			color: var(--text);
			background: var(--bg);
		}
	</style>
</svelte:head>

<Header/>

<div class="stat-cards">
	<wired-card elevation="3" fill="#3367d6" class="stat-card">
		<span>{format(totalConfirmed, 'Cases')}</span>
	</wired-card>

	<wired-card elevation="3" fill="#ea7075" class="stat-card">
		<span>{format(totalDeaths, 'Deaths')}</span>
	</wired-card>
</div>

<div style="margin-top: 5vh; text-align: center;">
	{#if data && selectedCountry.name}
		<select name="country-daily-select" id="country-select" bind:value={selectedCountry.name} on:change={() => selectCountry()}>
			{#each Object.keys(data) as country}
				<option value={country}>{ country }</option>
			{/each}
		</select>
	{/if}
</div>

{#if selectedCountry && selectedCountry.name && selectedCountry.dailyCases && selectedCountry.dailyDeaths}
<div class="daily-charts">
	<AreaChart title="{selectedCountry.name} Cases per Day" data={selectedCountry.dailyCases} data2={selectedCountry.dailyDeaths} yTicks={selectedCountry.casesTicks}></AreaChart>
	<AreaChart title="{selectedCountry.name} Deaths per Day" data={selectedCountry.dailyDeaths} yTicks={selectedCountry.deathsTicks}></AreaChart>
</div>
{/if}

<div class="charts-container">
	<LineChart countries={data}></LineChart>
	<Top10 countries={data} totalConfirmed={totalConfirmed} totalDeaths={totalDeaths}></Top10>
</div>

<wired-card class="disclaimer-card">
	<p>Data comes from the Novel Coronavirus COVID-19 Data Repository by Johns Hopkins Center for Systems Science and Engineering. It updates automatically every day. Last updated {lastUpdate}.</p>

	<wired-button elevation="3" class="source-button">
		<a class="source-link" href="https://github.com/CSSEGISandData/COVID-19" target="_blank">
			<svg class="svg-image" role="img" height="40" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><title>GitHub Icon</title><path d="M12 .297c-6.63 0-12 5.373-12 12 0 5.303 3.438 9.8 8.205 11.385.6.113.82-.258.82-.577 0-.285-.01-1.04-.015-2.04-3.338.724-4.042-1.61-4.042-1.61C4.422 18.07 3.633 17.7 3.633 17.7c-1.087-.744.084-.729.084-.729 1.205.084 1.838 1.236 1.838 1.236 1.07 1.835 2.809 1.305 3.495.998.108-.776.417-1.305.76-1.605-2.665-.3-5.466-1.332-5.466-5.93 0-1.31.465-2.38 1.235-3.22-.135-.303-.54-1.523.105-3.176 0 0 1.005-.322 3.3 1.23.96-.267 1.98-.399 3-.405 1.02.006 2.04.138 3 .405 2.28-1.552 3.285-1.23 3.285-1.23.645 1.653.24 2.873.12 3.176.765.84 1.23 1.91 1.23 3.22 0 4.61-2.805 5.625-5.475 5.92.42.36.81 1.096.81 2.22 0 1.606-.015 2.896-.015 3.286 0 .315.21.69.825.57C20.565 22.092 24 17.592 24 12.297c0-6.627-5.373-12-12-12"/></svg>
			<span style="margin-left: 1em; color: #3367d6;">Data Source</span>
		</a>
	</wired-button>

	<wired-button elevation="3" class="source-button">
		<a class="source-link" href="https://github.com/Jappzy/carona-balona" target="_blank">
			<svg class="svg-image" role="img" height="40" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><title>GitHub Icon</title><path d="M12 .297c-6.63 0-12 5.373-12 12 0 5.303 3.438 9.8 8.205 11.385.6.113.82-.258.82-.577 0-.285-.01-1.04-.015-2.04-3.338.724-4.042-1.61-4.042-1.61C4.422 18.07 3.633 17.7 3.633 17.7c-1.087-.744.084-.729.084-.729 1.205.084 1.838 1.236 1.838 1.236 1.07 1.835 2.809 1.305 3.495.998.108-.776.417-1.305.76-1.605-2.665-.3-5.466-1.332-5.466-5.93 0-1.31.465-2.38 1.235-3.22-.135-.303-.54-1.523.105-3.176 0 0 1.005-.322 3.3 1.23.96-.267 1.98-.399 3-.405 1.02.006 2.04.138 3 .405 2.28-1.552 3.285-1.23 3.285-1.23.645 1.653.24 2.873.12 3.176.765.84 1.23 1.91 1.23 3.22 0 4.61-2.805 5.625-5.475 5.92.42.36.81 1.096.81 2.22 0 1.606-.015 2.896-.015 3.286 0 .315.21.69.825.57C20.565 22.092 24 17.592 24 12.297c0-6.627-5.373-12-12-12"/></svg>
			<span style="margin-left: 1em; color: #3367d6;">Front-End Source</span>
		</a>
	</wired-button>
</wired-card>

<Footer/>

<style>

.stat-cards {
	display: flex;
	justify-content: center;
	margin: 2em auto;
}

.stat-card {
	padding: 2em;
	margin: 0.2em;
	text-align: center;
	color: white;
}

.daily-charts {
	display: grid;
	grid-gap: 3em;
	grid-template-columns: 1fr;
	margin: 2vh 2em 12vh;
}

@media (min-width: 800px) {
	.daily-charts {
		grid-template-columns: 1fr 1fr;
	}
}

.charts-container {
	display: flex;
	justify-content: space-around;
	align-items: flex-start;
	flex-wrap: wrap;
}

.disclaimer-card {
	margin: 4em auto 2em;
	display: block;
	text-align: center;
}

.source-button {
	margin: 2em 1em 4em;
	width: fit-content;
}

.source-link {
	display: flex;
	align-items: center;
	padding: 0.5em;
	color: #3934c1;
}

.svg-image {
	filter: var(--svg-filter);
	-webkit-filter: var(--svg-filter);
}

</style>
