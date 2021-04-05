<script>
	import { onMount } from 'svelte';
	import axios from 'axios';

	const DATA_SOURCE = 'https://corona.elezioni.io/data';
	const START_DATE = '2020-12-25'; // First Friday of the year
	
	let data;
	let italy = {
		cases: [],
		fatalities: [],
		tests: [],
		icus: [],
		hospitals: [],
	};

	onMount(async () => {
		console.log('mount');

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

        Promise.all([getItalia(), getRegioni(), getProvince(), getRegioni2020()])
            .then((results) => {
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
                data = JSON.parse(JSON.stringify(rawData));

				const italy_filtered = data.italy.global.filter(d => d.datetime >= START_DATE);

				console.log(italy_filtered);

				let tmpCases = {};
				let tmpFatalities = {};
				let tmpIcus = {};
				let tmpHospitals = {};
				let tmpTests = {};

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
				console.log(italy);
            })
			.catch((e) => {
				console.log("error", e);
			});
    });
</script>

<main>
	<div class="wrapper">
		<h1>Weekly new cases since Jan. 1st, 2021</h1>
		<div class="chart" id="newCases"></div>
	</div>
	<pre>{JSON.stringify(italy)}</pre>
</main>

<style>

	.wrapper {
		border: 1px solid #444;
		display: block;
		height: 650px;
		margin: 40px auto;
		position: relative;
		width: 650px;
	}

	h1 {
		display: block;
		font-size: 16px;
		font-weight: 400;
		height: 30px;
		line-height: 30px;
		margin:0;
		text-align: center;
	}

	.chart {
		display: block;
		height: 620px;
		position: relative;
		width: 650px;
	}
</style>