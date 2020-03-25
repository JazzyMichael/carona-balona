<script>
import { WiredInput } from 'wired-input';
import { WiredSlider } from 'wired-slider';
import { WiredCard } from 'wired-card';
import * as Pancake from '@sveltejs/pancake';

export let countries = [];

let x1 = +Infinity;
let x2 = -Infinity
let y1 = +Infinity;
let y2 = -Infinity;

let closest;
let filter;
let range;
let minRange = 0;
let maxRange;

$: regex = filter ? new RegExp(filter.value, 'i') : null;
$: filtered = regex ? countries.filter(country => regex.test(country.name)) : countries;
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

const addCommas = (num) => {
    return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
}

const onSearch = () => {
    regex = filter ? new RegExp(filter.value, 'i') : null;
}

const onSlide = () => {
    let startingIndex = range.value || 0;
    x1 = countries[0].data[startingIndex].x;
}

const setDimensions = (arr) => {
	if (!arr || !arr.length) return;

	maxRange = arr[0].data.length - 5;

    arr.forEach(country => {
        country.data.forEach(d => {
            if (d.x < x1) x1 = d.x;
            if (d.x > x2) x2 = d.x;
            if (d.y < y1) y1 = d.y;
            if (d.y > y2) y2 = d.y;
        });
    });
}

const convertData = (dataObj) => {
    return Object.entries(dataObj).map(([name, dataArr]) => ({
        name,
        data: dataArr.map(obj => {
            const [year, month, day] = obj.date.split('-');
            const raw = new Date(year, month-1, day);
            const epoch = raw.getTime() / 1000;
            return {
                x: epoch,
                y: obj.confirmed,
                date: obj.date,
                deaths: obj.deaths,
                xDisplay: `${month}/${day}`
            }
        })
    }));
}

$: {
    countries = convertData(countries);
    setDimensions(countries);
}

</script>

<div style="flex-grow: 1; margin-bottom: 4em;">

<div class="chart-filters">
    <wired-input class="search-input" bind:this={filter} placeholder="Search" on:input={onSearch}></wired-input>

    <div>
        <p style="text-align: center; margin-bottom: 0; color: gray;">Stretch</p>
        <wired-slider class="range-slider" value="0" min={minRange} max={maxRange} bind:this={range} on:change={onSlide}></wired-slider>
    </div>
</div>

<h3 class="chart-title">Countries</h3>

<wired-card elevation="3" class="chart-card">
    <div class="chart">
        <Pancake.Chart {x1} {x2} y1={y1} y2={y2}>
            <Pancake.Grid horizontal count={6} let:value>
                <div class="grid-line horizontal"><span>{value}</span></div>
            </Pancake.Grid>

            <Pancake.Grid vertical count={3} let:value>
                <span class="x-label">{new Date(+value * 1000).toLocaleString('en', { month: 'short' })}</span>
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
                        <span>{addCommas(closest.y)} cases | {addCommas(closest.deaths)} deaths</span>
                    </div>
                </Pancake.Point>
            {/if}

            <Pancake.Quadtree data={points} bind:closest/>
        </Pancake.Chart>
    </div>
</wired-card>

</div>

<style>
.chart-filters {
	display: flex;
	justify-content: space-around;
	align-items: center;
	flex-wrap: wrap;
	width: 95%;
	max-width: 666px;
	margin: 1em auto;
}

.search-input {
	display: block;
	margin: 1em auto 2em;
}

.range-slider {
	display: block;
	margin: 0 auto 1em;
}

.chart-title {
	width: 92%;
	margin: 1em auto;
	display: block;
}

.chart-card {
	width: 95%;
	margin: auto;
	display: block;
	box-sizing: border-box;
}

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
</style>
