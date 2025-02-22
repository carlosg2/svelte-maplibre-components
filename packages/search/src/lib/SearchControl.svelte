<script lang="ts">
	import { onMount } from 'svelte';
	import AutoComplete from 'simple-svelte-autocomplete';
	import { GeoJSONFeature, Map, Marker } from 'maplibre-gl';
	import { createEventDispatcher } from 'svelte';
	import type { SearchOption } from './types';

	export let map: Map;
	export let searchOption: SearchOption;
	export let position: 'top-left' | 'top-right' | 'bottom-left' | 'bottom-right' = 'top-left';

	const dispatch = createEventDispatcher();
	let searchItems: any[] = [];
	let searchObject: { [key: string]: GeoJSONFeature } = {};
	let selectedObject: { [key: string]: string } = {};
	let markerSearch: Marker | undefined;
	$: selectedObject, zoomToSearchedObject();

	let searchContainer: HTMLDivElement;
	// eslint-disable-next-line
	function SearchControl() {}

	SearchControl.prototype.onAdd = function (map: Map) {
		this.map = map;

		this.controlContainer = document.createElement('div');
		this.controlContainer.className = 'mapboxgl-ctrl';
		this.controlContainer.appendChild(searchContainer);
		return this.controlContainer;
	};

	SearchControl.prototype.onRemove = function () {
		if (!this.controlContainer || !this.controlContainer.parentNode || !this.map) {
			return;
		}
		this.controlContainer.parentNode.removeChild(this.controlContainer);
		this.map = undefined;
	};

	const zoomToSearchedObject = () => {
		if (!(searchObject && Object.keys(searchObject).length > 0)) return;
		if (!(selectedObject && Object.keys(selectedObject).length > 0 && selectedObject['key']))
			return;
		const f = searchObject[selectedObject['key']];
		if (!f) return;
		// eslint-disable-next-line @typescript-eslint/ban-ts-comment
		// @ts-ignore
		const coordinates = f.geometry.coordinates;
		map.flyTo({ center: coordinates, zoom: searchOption.zoom, curve: 1 });

		if (markerSearch) {
			markerSearch.remove();
			markerSearch = undefined;
		}
		markerSearch = new Marker({
			draggable: true
		})
			.setLngLat(coordinates)
			.addTo(map);

		dispatch('zoomed');
	};

	let onChange = () => {
		if (!selectedObject) {
			if (markerSearch) {
				markerSearch.remove();
				markerSearch = undefined;
			}
		}
	};

	/*global SearchControl */
	/*eslint no-undef: "error"*/
	// eslint-disable-next-line @typescript-eslint/ban-ts-comment
	// @ts-ignore
	let searchControl: SearchControl = null;

	onMount(async () => {
		if (!map) return;
		// eslint-disable-next-line @typescript-eslint/ban-ts-comment
		// @ts-ignore
		searchControl = new SearchControl();

		if (map.hasControl(searchControl) === false) {
			map.addControl(searchControl, position);
		}

		const res = await fetch(searchOption.url);
		const data = await res.json();
		data.features.forEach((feature: GeoJSONFeature) => {
			// eslint-disable-next-line @typescript-eslint/ban-ts-comment
			// @ts-ignore
			const key = feature.geometry.coordinates.join(',');
			const temp = JSON.parse(JSON.stringify(feature));
			temp.properties = {};
			searchOption.target.forEach((t) => {
				if (feature.properties[t]) {
					temp.properties[t] = feature.properties[t];
				}
			});
			if (Object.keys(temp.properties).length === 0) return;
			temp.properties.key = key;
			searchItems.push(temp.properties);
			searchObject[key] = JSON.parse(JSON.stringify(feature));
		});
	});
</script>

<div class="data-container" bind:this={searchContainer}>
	<AutoComplete
		bind:this={searchContainer}
		items={searchItems}
		bind:selectedItem={selectedObject}
		bind:onChange
		placeholder={searchOption.placeholder}
		hideArrow={false}
		showClear={true}
		showLoadingIndicator={false}
		labelFunction={(properties) => (properties ? searchOption.format(properties) : '')}
		maxItemsToShowInList="10"
	/>
</div>

<style lang="scss">
	:global(.autocomplete) {
		width: 100%;
	}

	:global(.autocomplete-input) {
		background-color: #fff !important;
		border-radius: 10px !important;
		border: 1px solid rgb(147, 147, 147) !important;
		box-shadow: 3px 3px 3px rgba(0, 0, 0, 0.1) !important;
		color: #4a4a4a !important;
		font-family: ProximaNova, sans-serif !important;
		font-size: 14px !important;
		height: 40px !important;
	}

	:global(.autocomplete-list) {
		top: 5px !important;
		background-color: #fff;
		border-radius: 10px;
		border: 1px solid rgb(147, 147, 147);
		box-shadow: 3px 3px 3px rgba(0, 0, 0, 0.1);
		color: #4a4a4a !important;
		font-family: ProximaNova, sans-serif !important;
		font-size: 14px !important;
	}
</style>
