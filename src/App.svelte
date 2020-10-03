<svelte:head>
	<link rel="stylesheet" href="./assets/css/normalize.css">
	<link rel="stylesheet" href="./assets/css/leaflet.css">
</svelte:head>

<script>
	import L from 'leaflet';
	import { onMount } from 'svelte';

	const regex = /^[a-zA-Z0-9][a-zA-Z0-9-]{1,61}[a-zA-Z0-9](?:\.[a-zA-Z]{2,})+$/gm;
	let map;
	let marker;
	let formIP = '';
	let ip = '';
	let location = '';
	let timezone = '';
	let isp = '';
	const customMarker = L.icon({
		iconUrl: '/assets/img/icon-location.svg',
		shadowUrl: '',
		iconSize: [46, 56],
		iconAnchor: [23, 56]
	});

	onMount(async () => {
    await fetch('https://api.ipify.org?format=json')
      .then(r => r.json())
      .then(async (data) => {
				ip = data.ip;
				
				getIPInfo(ip);
      });
	});

	const updateMapPosition = (lat, lng) => {
		map.setView([lat + 0.001, lng], 16);
		marker.setLatLng([lat, lng]);
	}
	
	const getIPInfo = async (ip) => {
		await fetch('https://geo.ipify.org/api/v1?apiKey=' + __myapp.env.API_KEY + '&ipAddress=' + ip)
			.then(r => r.json())
			.then(data => {
				location = data.location.city + ', ' + data.location.region + ' ' + data.location.postalCode;
				timezone = data.location.timezone;
				isp = data.isp;

				updateMapPosition(data.location.lat, data.location.lng);
			});
	};

	const getDomainIP = async (domain) => {
		await fetch('https://dns.google/resolve?name=' + domain)
			.then(r => r.json())
			.then(data => {
				getIPInfo(data.Answer[0].data);
			});
	}

	const handleSubmit = (e) => {
		e.preventDefault();
		const r = regex.exec(formIP)

		if(r === null) {
			getIPInfo(formIP);
		} else {
			getDomainIP(formIP);
		}
	};

	const createMap = (container) => {
		let m = L.map(container).setView([51.505, -0.09], 16);

		L.tileLayer('https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png',
      {
        attribution: `&copy;<a href="https://www.openstreetmap.org/copyright" target="_blank">OpenStreetMap</a>,&copy;<a href="https://carto.com/attributions" target="_blank">CARTO</a>`,
        maxZoom: 20,
      }
    ).addTo(m);

		return m;
	};

	const createMarker = () => {
  let m = L.marker([0, 0], {icon: customMarker}).addTo(map);

		return m;
	};

	const useMap = (container) => {
		map = createMap(container);
		marker = createMarker();

		return {
			destroy: () => {
				map.remove();
				marker.remove();
			}
		};
	};

	const resizeMap = () => {
		if(map) map.invalidateSize();
	}
</script>

<svelte:window on:resize={resizeMap} />

<main>
	<header class="header">
		<h1 class="title">IP Address Tracker</h1>

		<form class="form" on:submit={handleSubmit}>
			<input class="form__input" bind:value={formIP} type="text" placeholder="Search for any IP address or domain">
			<button class="form__button" type="submit">
				<img src="./assets/img/icon-arrow.svg" alt="">
			</button>
		</form>

		<div class="information">
			<div class="information__section">
				<p class="information__title">IP Address</p>
				<p class="information__content">
					{#if ip !== ''}
						{ip}
					{:else}
						...
					{/if}
				</p>
			</div>
			<div class="information__section">
				<p class="information__title">Location</p>
				<p class="information__content">
					{#if location !== ''}
						{location}
					{:else}
						...
					{/if}
				</p>
			</div>
			<div class="information__section">
				<p class="information__title">Timezone</p>
				<p class="information__content">
					{#if timezone !== ''}
						UTC {timezone}
					{:else}
						...
					{/if}
				</p>
			</div>
			<div class="information__section">
				<p class="information__title">ISP</p>
				<p class="information__content">
					{#if isp !== ''}
						{isp}
					{:else}
						...
					{/if}
				</p>
			</div>
		</div>
	</header>

	<section>
		<div class="map" use:useMap></div>
	</section>
</main>

<style>
	main {
		font-family: Arial, Helvetica, sans-serif;
		text-align: center;
	}

	main * {
		box-sizing: border-box;
	}

	.header {
		height: 280px;
		padding: 8px 24px 0 24px;
		background-image: url(../assets/img/pattern-bg.png);
		background-position: center;
		background-size: cover;
	}

	.title {
		color: white;
		letter-spacing: -0.2px;
		word-spacing: -3px;
	}

	.form {
		position: relative;
		max-width: 550px;
		margin: 28px auto 45px auto;
	}

	.form__input {
		width: 100%;
		border: none;
		border-radius: 15px;
		background-color: white;
		padding: 20px 78px 18px 22px;
		font-size: 18px;
		letter-spacing: 0.1px;
		word-spacing: 0.2px;
	}

	.form__button {
		cursor: pointer;
		position: absolute;
		top: 0;
		right: -1px;
		bottom: 0;
		-webkit-appearance: none;
		border: none;
		border-radius: 0 15px 15px 0;
		background-color: black;
		padding: 0 23px;
		color: white;
	}

	.information {
		position: relative;
		z-index: 1000;
		display: flex;
		background-color: white;
		border-radius: 15px;
		width: 100%;
		max-width: 1110px;
		margin: 0 auto;
		padding: 44px 0 40px 0;
		transition: all .2s ease;
	}

	.information__section {
		width: 100%;
		padding: 0 32px;
		text-align: left;
		border-right: 1px solid #d9d9d9
	}

	.information__section:last-child {
		border-right: none;
	}

	.information__title {
		margin: -8px 0 0 0;
		color: #949494;
		text-transform: uppercase;
		font-weight: bold;
		font-size: 14px;
	}

	.information__content {
		margin: 13px 0 0 0;
		color: #2d2d2d;
		font-weight: bolder;
		font-size: 25px;
	}

	.map {
		height: calc(100vh - 280px);
	}

	@media (max-width: 850px) {
		.header {
			height: 300px;
		}

		.title {
			font-size: 1.6rem;
		}

		.form {
			margin-bottom: 25px;
		}

		.information {
			flex-direction: column;
			padding: 25px 0 0 0;
		}

		.information__section {
			border: none;
			text-align: center;
			margin-bottom: 23px;
		}

		.information__title {
			margin: 0;
			font-size: 12px;
		}

		.information__content {
			font-size: 20px;
			margin-top: 7px;
		}

		.map {
			height: calc(100vh - 300px);
		}
	}
</style>