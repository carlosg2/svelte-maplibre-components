<script lang="ts">
	import { onMount } from 'svelte';
	import { Map, NavigationControl } from 'maplibre-gl';
	import { MenuControl } from '@watergis/svelte-maplibre-menu';
	import { MeasurePanel, type MeasureOption } from '@watergis/svelte-maplibre-measure';

	let isMenuShown = true;

	let mapContainer: HTMLDivElement;
	let map: Map;
	let innerHeight = 0;
	let innerWidth = 0;

	$: menuHeight = innerHeight * 0.6;
	$: menuWidth = innerWidth * 0.95;

	let terrainRgbUrl = 'https://narwassco.github.io/narok-terrain/tiles/{z}/{x}/{y}.png';
	let measureOption: MeasureOption = {
		tileSize: 512,
		font: ['Roboto Medium'],
		fontSize: 12,
		fontHalo: 1,
		mainColor: '#263238',
		haloColor: '#fff'
	};

	onMount(async () => {
		map = new Map({
			container: mapContainer,
			style: 'https://narwassco.github.io/mapbox-stylefiles/unvt/style.json'
		});
		map.addControl(new NavigationControl({}));
		map.scrollZoom.disable();
		map.touchPitch.enable();
	});
</script>

<svelte:window bind:innerWidth bind:innerHeight />

<MenuControl bind:map position={'top-right'} bind:isMenuShown width={menuWidth} height={menuHeight}>
	<div slot="primary" class="primary-container">
		<h4>Measure tool with elevation enquiry</h4>
		<MeasurePanel bind:map bind:measureOption bind:terrainRgbUrl />
	</div>
	<div slot="secondary">
		<div class="map" bind:this={mapContainer} />
	</div>
</MenuControl>

<style lang="scss">
	@import 'maplibre-gl/dist/maplibre-gl.css';

	.map {
		position: absolute;
		top: 0;
		bottom: 0;
		width: 100%;
		height: 100%;
		z-index: 1;
	}

	.primary-container {
		margin: 0.5rem;
	}
</style>
