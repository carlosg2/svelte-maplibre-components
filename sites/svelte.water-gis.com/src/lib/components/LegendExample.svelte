<script lang="ts">
	import { onMount } from 'svelte';
	import { Map, NavigationControl } from 'maplibre-gl';
	import { MenuControl } from '@watergis/svelte-maplibre-menu';
	import { LegendPanel, LegendHeader } from '@watergis/svelte-maplibre-legend';
	import type { StyleSpecification } from 'maplibre-gl';

	let mapContainer: HTMLDivElement;
	let map: Map;
	let innerHeight = 0;
	let innerWidth = 0;

	$: menuHeight = innerHeight * 0.6;
	$: menuWidth = innerWidth * 0.95;

	let isMenuShown = true;
	let style: StyleSpecification;
	$: {
		if (map) {
			map.on('load', () => {
				style = map.getStyle();
			});
		}
	}

	let onlyRendered = true;
	let onlyRelative = true;
	let relativeLayers: { [key: string]: string } = {
		pipeline: 'Pipeline',
		pipeline_annotation: 'Pipeline Label',
		meter: 'Water Meter',
		'flow meter': 'Flow Meter',
		valve: 'Valve',
		firehydrant: 'Fire Hydrant',
		washout: 'Washout',
		tank: 'Tank',
		tank_annotation: 'Tank Label',
		wtp: 'WTP',
		wtp_annotation: 'WTP Label',
		intake: 'Intake',
		intake_annotation: 'Intake Label',
		parcels: 'Parcels',
		parcels_annotation: 'Parcels Label',
		village: 'Village',
		village_annotation: 'Village Label',
		dma: 'DMA',
		'dma-annotation': 'DMA Label',
		'contour-line': 'Countour',
		'contour-label': 'Contour Label',
		hillshade: 'Hillshade',
		sewer_connection: 'Households (Sewer)',
		sewer_commercial: 'Commecial (Sewer)',
		sewer_institution: 'Institution (Sewer)',
		sewer_public_toilet: 'Public Toilet (Sewer)',
		manhole: 'Manhole',
		sewer_pipeline: 'Sewer pipeline',
		sewer_treatment_plant: 'Wastewater treatment plant'
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
		<div class="legend-header">
			<LegendHeader bind:onlyRendered bind:onlyRelative />
		</div>
		<div class="legend-content" style="height:{menuHeight - 56}px">
			<LegendPanel bind:map {style} bind:onlyRendered bind:onlyRelative {relativeLayers} />
		</div>
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
		display: flex;
		flex-direction: column;
		position: relative;

		.legend-header {
			padding-left: 0.5rem;
			padding-top: 0.5rem;
			padding-bottom: 0.5rem;
		}

		.legend-content {
			overflow-x: hidden;
			overflow-y: auto;
			width: 100%;
		}
	}
</style>
