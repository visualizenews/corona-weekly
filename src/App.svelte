<script>
	import { onMount } from 'svelte';
	import axios from 'axios';

	const dataSource = 'https://corona.elezioni.io/data';
	const startDate = '2021-01-01'; // First Friday of the year
	
	let data;

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
            return axios.get(`${dataSource}/italia`);
        }

        const getRegioni = () => {
            return axios.get(`${dataSource}/regioni`);
        }

        const getRegioni2020 = () => {
            return axios.get(`${dataSource}/regioni2020`);
        }

        const getProvince = () => {
            return axios.get(`${dataSource}/province`);
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
            })
			.catch((e) => {
				console.log("error", e);
			});
    });


		
</script>

<main>
	<pre>{JSON.stringify(data)}</pre>
</main>

<style>
</style>