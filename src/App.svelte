<script>
	import { onMount } from 'svelte';
	import axios from 'axios';

	import Chart from './Chart.svelte';
	import Line from './Line.svelte';

	const DATA_SOURCE = 'https://corona.elezioni.io/data';
	const START_DATE = '2020-12-25'; // First Friday of the year
	const CHARTS = [
		{
			id: 'cases',
			label: 'Highest Number of<br /><span>New Cases</span>',
			label2: 'Highest Number<br />of <span>New Cases</span>',
			title: 'New Cases',
		},
		{
			id: 'hospitals',
			label: 'Highest Increment of<br /><span>Hospital Bed occupancy</span>',
			label2: 'Highest Number of <span>Hospital Bed occupancy</span>',
			title: 'Hospital Bed occupancy',
		},
		{
			id: 'icus',
			label: 'Highest Increment of<br /><span>ICU Beds occupancy</span>',
			label2: 'Highest Number<br />of <span>ICU Beds occupancy</span>',
			title: 'ICU Bed occupancy',
		},
		{
			id: 'fatalities',
			label: 'Highest Number of<br /><span>Fatalities</span>',
			label2: 'Highest Number of <span>Fatalities</span>',
			title: 'Fatalities',
		},
		{
			id: 'tests',
			label: 'Highest Number of<br /><span>Tests Performed</span>',
			label2: 'Highest Number of <span>Tests Performed</span>',
			title: 'Tests',
		},
	];

	Date.prototype.getWeek = function (dowOffset) {
		/*getWeek() was developed by Nick Baicoianu at MeanFreePath: http://www.meanfreepath.com */

		dowOffset = typeof(dowOffset) == 'int' ? dowOffset : 0; //default dowOffset to zero
		let newYear = new Date(this.getFullYear(),0,1);
		let day = newYear.getDay() - dowOffset; //the day of week the year begins on
		day = (day >= 0 ? day : day + 7);
		let daynum = Math.floor((this.getTime() - newYear.getTime() - 
		(this.getTimezoneOffset()-newYear.getTimezoneOffset())*60000)/86400000) + 1;
		let weeknum;
		//if the year starts before the middle of a week
		if(day < 4) {
				weeknum = Math.floor((daynum+day-1)/7) + 1;
				if(weeknum > 52) {
					let nYear = new Date(this.getFullYear() + 1,0,1);
					let nday = nYear.getDay() - dowOffset;
					nday = nday >= 0 ? nday : nday + 7;
					/*if the next year starts before the middle of
						the week, it is week #1 of that year*/
					weeknum = nday < 4 ? 1 : 53;
				}
		}
		else {
				weeknum = Math.floor((daynum+day-1)/7);
		}
		return weeknum;
	};
	
	let data;
	let data2020 = {
		cases: [],
		fatalities: [],
		tests: [],
		icus: [],
		hospitals: [],
	};
	let data2021 = {
		cases: [],
		fatalities: [],
		tests: [],
		icus: [],
		hospitals: [],
	};

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

				// console.log('italy_filtered', italy_filtered);

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
				// console.log('italy', italy);

				italy.cases.shift();
				italy.hospitals.shift();
				italy.icus.shift();
				italy.fatalities.shift();
				italy.tests.shift();
				data = JSON.parse(JSON.stringify(italy));
				let week = -10;
				let refYear = 0;

				tmpCases = {};
				tmpFatalities = {};
				tmpIcus = {};
				tmpHospitals = {};
				tmpTests = {};

				rawData.italy.global.forEach((d, i) => {
					const currentDay = new Date(d.datetime);
					const year = currentDay.getFullYear();
					const thisWeek = currentDay.getWeek();
					if (week !== thisWeek || year !== refYear) {
						if (week > 0) {
							if (refYear === 2020) {
								data2020.cases.push(tmpCases);
								data2020.fatalities.push(tmpFatalities);
								data2020.hospitals.push(tmpHospitals);
								data2020.icus.push(tmpIcus);
								data2020.tests.push(tmpTests);
							} else if (refYear === 2021) {
								data2021.cases.push(tmpCases);
								data2021.fatalities.push(tmpFatalities);
								data2021.hospitals.push(tmpHospitals);
								data2021.icus.push(tmpIcus);
								data2021.tests.push(tmpTests);
							}
						}
						refYear = year;
						week = thisWeek;

						tmpCases = {
							datetime: currentDay,
							value: 0,
							week,
						};
						tmpFatalities = {
							datetime: currentDay,
							value: 0,
							week,
						};
						tmpHospitals = {
							datetime: currentDay,
							value: 0,
							week,
						};
						tmpIcus = {
							datetime: currentDay,
							value: 0,
							week,
						};
						tmpTests = {
							datetime: currentDay,
							value: 0,
							week,
						};
					}

					tmpCases.value = (() => {
						if (d.new_tested_positive) {
							return tmpCases.value + d.new_tested_positive;
						}
						return tmpCases.value;
					})();
					tmpFatalities.value = (() => {
						if (d.deaths) {
							return tmpFatalities.value + ((i === 0) ? d.deaths : d.deaths - (rawData.italy.global[i - 1].deaths));
						}
						return tmpFatalities.value;
					})();
					tmpHospitals.value = (() => {
						if (d.hospital) {
							return tmpHospitals.value + d.hospital;
						}
						return tmpHospitals.value;
					})();
					tmpIcus.value = (() => {
						if (d.icu) {
							return tmpIcus.value + d.icu;
						}
						return tmpIcus.value;
					})();
					tmpTests.value = (() => {
						if (d.tested) {
							return tmpTests.value + ((i === 0) ? d.tested : d.tested - (rawData.italy.global[i - 1].tested));
						}
						return tmpTests.value;
					})()
				});

				console.log(data, data2020, data2021);

      }).catch((e) => {
				console.log("error", e);
			});
		}

		fetchData();
  });
</script>

<main>
	<div class="wrapper">
		<h1>Italy: Weekly Covid19 Progression since January 1<span>st</span>, 2021</h1>
		{#if data !== undefined}
			{#each CHARTS as chart, i}
				<div class="chart" id="wrapper-{chart.id}">
					<Chart id="{chart.id}" data="{data}" label="{chart.label}" position="{i}" charts="{CHARTS.length}" />
				</div>
			{/each}
		{/if}
		<h4>Data from Italian Civil Protection.
			Weeks are computed from Friday to Friday, the day the Italian Government and the Italian Scientific Committee review COVID-Related Data.</h4>
	</div>
	<br />
	<br />
	<br />
	<br />
	<hr />
	<br />
	<br />
	<br />
	<br />
	<div class="wrapper wrapper2">
		<h1>Italy: Comparing Weekly Covid19 Progression Years <span class="y2020">2020</span> vs <span class="y2021">2021</span></h1>
		{#if data !== undefined}
			{#each CHARTS as chart, i}
				<div class="chart chart2" id="wrapper-{chart.id}">
					<Line id="{chart.id}" data2020="{data2020}" data2021="{data2021}" label="{chart.label2}" position="{i}" charts="{CHARTS.length}" />
				</div>
			{/each}
		{/if}
		<h4>Data from Italian Civil Protection.</h4>
	</div>

	<div class="assets">
		<svg>
			<defs>
				<pattern
					id="stripes"
					width="5px"
					height="5px"
					patternUnits="userSpaceOnUse"
					patternContentUnits="userSpaceOnUse"
					viewBox="0 0 100 100"
					preserveAspectRatio="none"
					patternTransform="rotate(-45)"
				>
					<line
						x1="0"
						x2="100"
						y1="46"
						y2="46"
						stroke-width="20"
						stroke="#444"
					/>
					<line
						x1="0"
						x2="100"
						y1="92"
						y2="92"
						stroke-width="20"
						stroke="#444"
					/>
					<line
						x1="46"
						x2="46"
						y1="0"
						y2="100"
						stroke-width="20"
						stroke="#444"
					/>
					<line
						x1="92"
						x2="92"
						y1="0"
						y2="100"
						stroke-width="20"
						stroke="#444"
					/>
				</pattern>
				<pattern
					id="stripes2"
					width="5px"
					height="5px"
					patternUnits="userSpaceOnUse"
					patternContentUnits="userSpaceOnUse"
					viewBox="0 0 140 140"
					preserveAspectRatio="none"
					patternTransform="rotate(-45)"
				>
					<line
						x1="0"
						x2="140"
						y1="70"
						y2="70"
						stroke-width="30"
						stroke="#444"
					/>
				</pattern>
				<pattern
					id="stripes3"
					width="5px"
					height="5px"
					patternUnits="userSpaceOnUse"
					patternContentUnits="userSpaceOnUse"
					viewBox="0 0 140 140"
					preserveAspectRatio="none"
					patternTransform="rotate(45)"
				>
					<line
						x1="0"
						x2="140"
						y1="70"
						y2="70"
						stroke-width="30"
						stroke="#444"
					/>
				</pattern>
				<pattern
					id="dots"
					width="3px"
					height="3px"
					patternUnits="userSpaceOnUse"
					patternContentUnits="userSpaceOnUse"
					viewBox="0 0 5 5"
					preserveAspectRatio="none"
					patternTransform="rotate(45)"
				>
					<circle
						r="1"
						cx="3"
						cy="3"
						fill="#9a9a9a"
						stroke="transparent"
						stroke-width="0" />
				</pattern>
			</defs>
		</svg>
	</div>
</main>

<style>

	.wrapper {
		background: #fff;
		border: 1px solid #444;
		display: block;
		height: auto;
		margin: 40px auto;
		position: relative;
		width: 750px;
	}

	.wrapper2 {
		width: 950px;
	}

	h1 {
		display: block;
		font-size: 18px;
		font-weight: 400;
		height: 50px;
		letter-spacing: .1em;
		line-height: 30px;
		margin:0;
		text-align: center;
		text-transform: uppercase;
	}

	h4 {
		display: block;
		font-size: 10px;
		font-weight: 400;
		letter-spacing: .1em;
		line-height: 1.25;
		margin: 0;
		padding: 20px 40px;
		text-align: center;
	}

	h1 span {
		text-transform: lowercase;
	}

	.chart {
		/* border-bottom: 1px solid #444; */
		display: block;
		height: auto;
		min-height: 150px;
		position: relative;
		width: 750px;
	}

	.chart2 {
		width: 950px;
	}

	.chart:nth-child(odd) {
		background: #fcfcfc;
	}

	.chart:last-child {
		border-bottom: inherit;
	}

	.assets {
		height: 0;
		opacity: 0;
		overflow: hidden;
		width: 0;
	}

	.y2020 {
		color: #9a9a9a;
		font-weight: 700;
	}

	.y2021 {
		font-weight: 700;
	}
</style>