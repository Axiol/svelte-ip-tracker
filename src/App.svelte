<svelte:head>
	<link rel="stylesheet" href="./assets/css/normalize.css">
	<link rel="stylesheet" href="./assets/css/leaflet.css">
</svelte:head>

<script>
	import L from 'leaflet';

	let map;
	let formIP = '';

	const handleSubmit = (e) => {
		e.preventDefault();
		console.log('foo');
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

	const useMap = (container) => {
		console.log(container);
		map = createMap(container);

		return {
			destroy: () => {
				map.remove();
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
			<input class="form__input" value={formIP} type="text" placeholder="Search for any IP address or domain">
			<button class="form__button" type="submit">
				<img src="./assets/img/icon-arrow.svg" alt="">
			</button>
		</form>

		<div class="information">
			<div class="information__section">
				<p class="information__title">IP Address</p>
				<p class="information__content">192.212.174.101</p>
			</div>
			<div class="information__section">
				<p class="information__title">Location</p>
				<p class="information__content">Brooklyn, NY 10001</p>
			</div>
			<div class="information__section">
				<p class="information__title">Timezone</p>
				<p class="information__content">UTC -05:00</p>
			</div>
			<div class="information__section">
				<p class="information__title">ISP</p>
				<p class="information__content">SpaceX Starlink</p>
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
		padding: 8px 20px 0 20px;
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
		padding: 19px 78px 20px 22px;
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
		/* display: none; */
		height: calc(100vh - 280px);
	}
</style>