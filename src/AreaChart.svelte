<script>
    import { scaleLinear } from 'd3-scale';
	
	export let title = '';
	export let data;
	export let data2 = null;
	export let yTicks;

	const xTicks = [
		{ display: 'Mar', timestamp: 1583038800 },
		{ display: 'May', timestamp: 1588305600 },
		{ display: 'July', timestamp: 1593576000 },
		{ display: 'Sept', timestamp: 1598932800 },
		{ display: 'Nov', timestamp: 1604203200 },
		{ display: 'Jan', timestamp: 1609477200 },
		{ display: 'Mar', timestamp: 1614574800 },
		{ display: 'May', timestamp: 1619841600 }
	];

	const padding = { top: 20, right: 15, bottom: 20, left: 25 };

	let width = 700;
    let height = 200;

	$: xScale = scaleLinear()
		.domain([minX, maxX])
		.range([padding.left, width - padding.right]);

	$: yScale = scaleLinear()
		.domain([Math.min.apply(null, yTicks), Math.max.apply(null, yTicks)])
		.range([height - padding.bottom, padding.top]);

	$: minX = data && data.length ? data[0].x : 0;
	$: maxX = data && data.length ? data[data.length - 1].x : 0;
	$: path = data ? `M${data.map(p => `${xScale(p.x)},${yScale(p.y)}`).join('L')}` : '';
	$: area = data ? `${path}L${xScale(maxX)},${yScale(0)}L${xScale(minX)},${yScale(0)}Z` : '';
	
	$: path2 = data2 ? `M${data2.map(p => `${xScale(p.x)},${yScale(p.y)}`).join('L')}` : '';
	$: area2 = data2 ? `${path2}L${xScale(maxX)},${yScale(0)}L${xScale(minX)},${yScale(0)}Z` : '';

	const addCommas = (num) => {
    return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
}

</script>

{#if data && data.length}
<div class="chart-container">

	<h2 style="text-align: center">{ title }</h2>

	<div class="chart" bind:clientWidth={width} bind:clientHeight={height}>
		<svg>
			<g class="axis y-axis" transform="translate(0, {padding.top})">
				{#each yTicks as tick}
					<g class="tick tick-{tick}" transform="translate(0, {yScale(tick) - padding.bottom})">
						<line x2="100%"></line>
						<text y="-4">{addCommas(tick)}</text>
					</g>
				{/each}
			</g>

			<g class="axis x-axis">
				{#each xTicks as tick}
					<g class="tick tick-{ tick.timestamp }" transform="translate({xScale(tick.timestamp)},{height})">
						<line y1="-{height}" y2="-{padding.bottom}" x1="0" x2="0"></line>
						<text y="-2">{ tick.display }</text>
					</g>
				{/each}
			</g>

			<path class="path-area" d={area}></path>
			<path class="path-line" d={path}></path>

			<path class="path-area2" d={area2}></path>
			<path class="path-line2" d={path2}></path>
		</svg>
	</div>

</div>
{/if}

<style>
	.chart-container {
		width: 100%;
		margin: auto;
	}

	.chart {
		width: 100%;
		margin-left: auto;
		margin-right: auto;
	}

	svg {
		position: relative;
		width: 100%;
		height: 200px;
		overflow: visible;
	}

	.tick {
		font-size: .725em;
		font-weight: 200;
	}

	.tick line {
		stroke: #aaa;
		stroke-dasharray: 2;
	}

	.tick text {
		fill: #666;
		text-anchor: start;
	}

	.tick.tick-0 line {
		stroke-dasharray: 0;
	}

	.x-axis .tick text {
		text-anchor: middle;
	}

	.path-line {
		fill: none;
		stroke: rgb(0,100,100);
		stroke-linejoin: round;
		stroke-linecap: round;
		stroke-width: 2;
	}

	.path-area {
		fill: rgba(0,100,100,0.2);
	}

	.path-line2 {
		fill: none;
		stroke: rgb(0, 18, 100);
		stroke-linejoin: round;
		stroke-linecap: round;
		stroke-width: 2;
	}

	.path-area2 {
		fill: rgba(0, 18, 100, 0.2);
	}
</style>
