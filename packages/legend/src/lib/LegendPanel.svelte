<script lang="ts">
	import type { StyleSpecification, LayerSpecification, Map } from 'maplibre-gl';
	import Layer from './Layer.svelte';
	import SpriteLoader from './sprite';
	import { invisibleLayerMap } from './stores';

	export let map: Map;
	export let style: StyleSpecification;
	export let onlyRendered = true;
	export let onlyRelative = true;
	let spriteLoader: SpriteLoader | undefined;

	$: allLayers = style ? style.layers : [];
	let visibleLayerMap: { [key: string]: LayerSpecification } = {};
	export let relativeLayers: { [key: string]: string } = {};

	$: {
		if (map) {
			map.on('moveend', updateLayers);
			map.on('styledata', updateLayers);
			map.on('load', () => {
				updateLayers();
			});
		}

		if (relativeLayers && Object.keys(relativeLayers).length === 0) {
			onlyRelative = false;
		}
	}

	$: onlyRendered, updateLayers();
	$: onlyRelative, updateLayers();
	$: style, handleStyleChanged();

	const handleStyleChanged = async () => {
		if (!map) return;
		if (!style) return;
		const styleUrl = style.sprite;
		if (!styleUrl) return;
		spriteLoader = new SpriteLoader(styleUrl);
		await spriteLoader.load();
		updateLayers();
	};

	const updateLayers = () => {
		if (!map) return;
		if (!style) return;
		setTimeout(() => {
			visibleLayerMap = {};
			if (onlyRendered === true) {
				Object.keys($invisibleLayerMap).forEach((layerId) => {
					visibleLayerMap[layerId] = $invisibleLayerMap[layerId];
				});
				const features = map.queryRenderedFeatures();
				allLayers.forEach((layer) => {
					const filtered = features.filter((f) => f.layer.id === layer.id);
					if (filtered.length > 0) {
						visibleLayerMap[layer.id] = layer;
					}
				});
			} else {
				allLayers.forEach((layer) => {
					visibleLayerMap[layer.id] = layer;
				});
			}
		}, 500);
	};

	const layerVisibilityChanged = (e: {
		detail: { layer: LayerSpecification; visibility: 'visible' | 'none' };
	}) => {
		const layer: LayerSpecification = e.detail.layer;
		const visibility = e.detail.visibility;
		if (visibility === 'none') {
			$invisibleLayerMap[layer.id] = layer;
		} else {
			if ($invisibleLayerMap[layer.id]) {
				delete $invisibleLayerMap[layer.id];
			}
		}
	};
</script>

<ul class="legend-panel">
	{#if spriteLoader}
		{#key style}
			{#each allLayers as layer}
				{#if onlyRendered === true}
					{#if visibleLayerMap[layer.id]}
						{#if onlyRelative === true}
							{#if relativeLayers[layer.id]}
								<li class="legend-panel-block">
									<Layer
										{map}
										{layer}
										{spriteLoader}
										{relativeLayers}
										on:visibilityChanged={layerVisibilityChanged}
									/>
								</li>
							{/if}
						{:else}
							<li class="legend-panel-block">
								<Layer
									{map}
									{layer}
									{spriteLoader}
									{relativeLayers}
									on:visibilityChanged={layerVisibilityChanged}
								/>
							</li>
						{/if}
					{/if}
				{:else if onlyRelative === true}
					{#if relativeLayers[layer.id]}
						<li class="legend-panel-block">
							<Layer
								{map}
								{layer}
								{spriteLoader}
								{relativeLayers}
								on:visibilityChanged={layerVisibilityChanged}
							/>
						</li>
					{/if}
				{:else}
					<li class="legend-panel-block">
						<Layer
							{map}
							{layer}
							{spriteLoader}
							{relativeLayers}
							on:visibilityChanged={layerVisibilityChanged}
						/>
					</li>
				{/if}
			{/each}
		{/key}
	{/if}
</ul>

<style lang="scss">
	.legend-panel {
		display: block;
		list-style-type: disc;
		margin-block-start: 0em;
		margin-block-end: 0em;
		margin-inline-start: 0px;
		margin-inline-end: 0px;
		padding-inline-start: 1rem;

		.legend-panel-block {
			display: flex;
			vertical-align: middle;
			justify-content: left;
			align-items: left;
			margin: 0;
			padding: 0.2rem;
			border-bottom: 0.1rem solid rgb(197, 197, 197);
		}

		.legend-panel-block li {
			display: block;
		}

		.legend-panel-block:hover {
			background-color: rgb(239, 239, 239);
		}
	}
</style>
