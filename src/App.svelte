<script>
	import { onMount } from 'svelte';
	import axios from 'axios';

	import Chart from './Chart.svelte';

	const DATA_SOURCE = 'https://corona.elezioni.io/data';
	const START_DATE = '2020-04-24'; // First Friday of the year
	const CHARTS = [
		{
			id: 'cases',
			title: 'New Cases',
		},
		{
			id: 'hospitals',
			title: 'Hospital Occupancy'
		},
		{
			id: 'icus',
			title: 'ICU Occupancy',
		},
		{
			id: 'fatalities',
			title: 'Fatalities',
		},
		{
			id: 'tests',
			title: 'Tests Performed',
		},
	];
	
	let data;

	onMount(async () => {
		const rawData = {
			epicenters: [],
			generated: '',
			italy: {
				global: [],
				provinces: [],
				regions: [],
			},
			tested: [],
		};

		const italy = {
			cases: [],
			fatalities: [],
			tests: [],
			icus: [],
			hospitals: [],
		};

		const fetchData = async () => {
			const getItalia = () => {
					return axios.get(`${DATA_SOURCE}/italia`);
			}

			const getRegioni = () => {
					return axios.get(`${DATA_SOURCE}/regioni`);
			}

			const getRegioni2020 = () => {
					return axios.get(`${DATA_SOURCE}/regioni2020`);
			}

			const getProvince = () => {
					return axios.get(`${DATA_SOURCE}/province`);
			}

      Promise.all([getItalia(), getRegioni(), getProvince(), getRegioni2020()]).then((results) => {
				if (!results[0].data.error) {
						rawData.italy.global = results[0].data.data.italy.global;
						rawData.tested = results[0].data.data.tested;
						rawData.generated = results[0].data.data.generated;
				}
				if (!results[1].data.error) {
						rawData.italy.regions = results[1].data.data.italy.regions;
						rawData.generated = results[0].data.data.generated;
				}
				if (!results[2].data.error) {
						rawData.italy.provinces = results[2].data.data.italy.provinces;
						rawData.generated = results[0].data.data.generated;
				}
				if (!results[3].data.error) {
						rawData.italy.regions = rawData.italy.regions.concat(results[3].data.data.italy.regions).sort((a,b) => a.datetime > b.datetime ? 1 : -1);
						rawData.generated = results[0].data.data.generated;
				}
				const italy_filtered = rawData.italy.global.filter(d => d.datetime >= START_DATE);
				let tmpCases = {};
				let tmpFatalities = {};
				let tmpIcus = {};
				let tmpHospitals = {};
				let tmpTests = {};

				console.log('italy_filtered', italy_filtered);

				italy_filtered.forEach((d, i) => {
					const startDay = new Date(d.datetime);
					const thisDay = startDay.getDay();
					if (thisDay === 5) {
						if (i > 0) {
							italy.cases.push(tmpCases);
							italy.fatalities.push(tmpFatalities);
							italy.icus.push(tmpIcus);
							italy.hospitals.push(tmpHospitals);
							italy.tests.push(tmpTests);
						}
						const endDay = new Date(d.datetime);
						endDay.setDate(endDay.getDate() + 6);
						tmpCases = {
							avg: d.new_tested_positive,
							endDay,
							startDay,
							value: d.new_tested_positive,
						};
						tmpFatalities = {
							avg: (i === 0) ? d.deaths : d.deaths - (italy_filtered[i -1].deaths),
							endDay,
							startDay,
							value: (i === 0) ? d.deaths : d.deaths - (italy_filtered[i -1].deaths),
						};
						tmpIcus = {
							avg: (i === 0) ? d.icu : d.icu - (italy_filtered[i -1].icu),
							endDay,
							startDay,
							value: (i === 0) ? d.icu : d.icu - (italy_filtered[i -1].icu),
						};
						tmpHospitals = {
							avg: (i === 0) ? d.hospital : d.hospital - (italy_filtered[i -1].hospital),
							endDay,
							startDay,
							value: (i === 0) ? d.hospital : d.hospital - (italy_filtered[i -1].hospital),
						};
						tmpTests = {
							avg: (i === 0) ? d.tested : d.tested - (italy_filtered[i -1].tested),
							endDay,
							startDay,
							value: (i === 0) ? d.tested : d.tested - (italy_filtered[i -1].tested),
						};
					} else {
						tmpCases.avg = (tmpCases.avg + d.new_tested_positive) / 7;
						tmpCases.value = tmpCases.value + d.new_tested_positive;
						tmpFatalities.avg = (tmpFatalities.avg + d.deaths - (italy_filtered[i -1].deaths)) / 7;
						tmpFatalities.value = tmpFatalities.value + d.deaths - (italy_filtered[i -1].deaths);
						tmpIcus.avg = (tmpIcus.avg + d.icu - (italy_filtered[i -1].icu)) / 7;
						tmpIcus.value = tmpIcus.value + d.icu - (italy_filtered[i -1].icu);
						tmpHospitals.avg = (tmpHospitals.avg + d.hospital - (italy_filtered[i -1].hospital)) / 7;
						tmpHospitals.value = tmpHospitals.value + d.hospital - (italy_filtered[i -1].hospital);
						tmpTests.avg = (tmpTests.avg + d.tested - (italy_filtered[i -1].tested)) / 7;
						tmpTests.value = tmpTests.value + d.tested - (italy_filtered[i -1].tested);
					}
				});
				console.log('italy', italy);

				italy.cases.shift();
				italy.hospitals.shift();
				italy.icus.shift();
				italy.fatalities.shift();
				italy.tests.shift();

				data = JSON.parse(JSON.stringify(italy));
      }).catch((e) => {
				console.log("error", e);
			});
		}

		fetchData();
  });
</script>

<main>
	<div class="wrapper">
		<h1>Weekly Progression since May 2020</h1>
		{#if data !== undefined}
			{#each CHARTS as chart, i}
				<div class="chart" id="wrapper-{chart.id}">
					<Chart id="{chart.id}" data="{data}" label="{chart.title}" position="{i}" charts="{CHARTS.length}" />
				</div>
			{/each}
		{/if}
	</div>
</main>

<style>

	.wrapper {
		border: 1px solid #444;
		display: block;
		height: auto;
		margin: 40px auto;
		position: relative;
		width: 750px;
	}

	h1 {
		display: block;
		font-size: 16px;
		font-weight: 300;
		height: 50px;
		letter-spacing: .1em;
		line-height: 30px;
		margin:0;
		text-align: center;
		text-transform: uppercase;
	}

	h1 span {
		text-transform: lowercase;
	}

	.chart {
		display: block;
		height: 175px;
		position: relative;
		width: 750px;
	}
</style>