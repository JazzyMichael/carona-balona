<script>
import { WiredListbox } from 'wired-listbox';
import { WiredItem } from 'wired-item';
import { WiredCard } from 'wired-card';
import { WiredProgress } from 'wired-progress';

export let countries;
export let totalConfirmed = 0;
export let totalDeaths = 0;

let selected;
let top = {};
let total = {};
$: list = selected ? top[selected] : [];

const onSelect = (event) => {
    if (event && event.target && event.target.value && event.target.value.value) {
        selected = event.target.value.value;
    }
}

const addCommas = (num) => {
	if (!num) return
	return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
}

const decimalToPercentage = (num) => {
    const str = (num * 100).toString();
    let [left, right] = str.split('.');
    return `${left}.${right.substring(0, 2)}`;
}

const convertData = (dataObj) => {
    if (!dataObj) return [];
    return Object.entries(dataObj).map(([name, dataArr]) => {
        const confirmed = dataArr[dataArr.length - 1].confirmed;
        const deaths = dataArr[dataArr.length - 1].deaths;
        const ratio = Math.floor(+deaths / +confirmed * 100);
        return { name, confirmed, deaths, ratio };
    });
}

$: {
    countries = convertData(countries);
    top = {
        confirmed: [...countries].sort((a, b) => b.confirmed - a.confirmed).slice(0, 10),
        deaths: [...countries].sort((a, b) => b.deaths - a.deaths).slice(0, 10),
        ratio: [...countries].filter(x => x.confirmed > 100).sort((a, b) => b.ratio - a.ratio).slice(0, 10)
    }
    total = {
        confirmed: totalConfirmed,
        deaths: totalDeaths
    }
    selected = 'confirmed';
}

</script>

<div class="container">
    <div style="display: flex; justify-content: space-around; align-items: center;">
        <h3>Top 10</h3>

        <wired-listbox horizontal="true" selected="confirmed" on:selected={onSelect} class="listbox">
            <wired-item value="confirmed">Confirmed</wired-item>
            <wired-item value="deaths">Deaths</wired-item>
            <wired-item value="ratio">Ratio</wired-item>
        </wired-listbox>
    </div>

    {#if selected}
        <wired-card style="width: 100%; box-sizing: border-box; display: block; margin: 1em auto;">
            {#each list as country}
                <div style="width: 95%; display: flex; justify-content: space-between; margin: 1em 0.4em 0;">
                    <span>{country.name}</span>
                    {#if total[selected]}
                        <span>{decimalToPercentage(country[selected] / total[selected])}%</span>
                    {:else if selected === 'ratio'}
                        <span>{addCommas(country.deaths)}/{addCommas(country.confirmed)}</span>
                    {/if}
                </div>
                <wired-progress class="progress" value={country[selected]} max={selected === 'ratio' ? 100 : top[selected][0][selected]} percentage={selected === 'ratio'}></wired-progress>
            {/each}
        </wired-card>
        <!-- {#if selected === 'ratio'}
            <p style="text-align: center"><i>*ratio includes countries with at least 100 confirmed cases</i></p>
        {/if} -->
    {/if}
</div>

<style>
.progress {
    --wired-progress-color: #e48f73;
    width: 300px;
    max-width: 98%;
    display: block;
    margin: auto;
}

.listbox {
    --wired-item-selected-color: #e48f73;
    --wired-item-selected-bg: #fff1f4;
}

@media (min-width: 900px) {
    .container {
        margin-right: 1.5em;
        margin-left: 3em;
    }
}
</style>
