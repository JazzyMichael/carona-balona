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
    selected = event.target.value.value;
}

const decimalToPercentage = (num) => {
    const str = (num * 100).toString();
    const [left, right] = str.split('.');
    return `${left}.${right.substring(0, 2)}%`
}

const convertData = (dataObj) => {
    if (!dataObj) return [];
    return Object.entries(dataObj).map(([name, dataArr]) => ({
        name,
        confirmed: dataArr[dataArr.length - 1].confirmed,
        deaths: dataArr[dataArr.length - 1].deaths
    }));
}

$: {
    countries = convertData(countries);
    top = {
        confirmed: [...countries].sort((a, b) => b.confirmed - a.confirmed).slice(0, 10),
        deaths: [...countries].sort((a, b) => b.deaths - a.deaths).slice(0, 10)
    }
    total = {
        confirmed: totalConfirmed,
        deaths: totalDeaths
    }
    selected = 'confirmed';
}

</script>

<div>

<div style="display: flex; justify-content: space-around; align-items: center;">
    <h3>Top 10</h3>

    <wired-listbox horizontal="true" selected="confirmed" on:selected={onSelect} class="listbox">
        <wired-item value="confirmed">Confirmed Cases</wired-item>
        <wired-item value="deaths">Deaths</wired-item>
    </wired-listbox>
</div>

<br>

{#if selected}
    <wired-card style="width: 100%; box-sizing: border-box;">
        {#each list as country}
            <div style="width: 95%; display: flex; justify-content: space-between; margin: 1em 0.4em 0;">
                <span>{country.name}</span>
                <span>({decimalToPercentage(country[selected] / total[selected])})</span>
            </div>
            <wired-progress class="progress" value={country[selected]} max={top[selected][0][selected]}></wired-progress>
        {/each}
    </wired-card>
{/if}

</div>

<style>

.progress {
    --wired-progress-color: #dc00321a;
    width: 300px;
    max-width: 98%;
    display: block;
    margin: auto;
}

.listbox {
    --wired-item-selected-color: darkred;
    --wired-item-selected-bg: #dc00321a;
}



</style>
