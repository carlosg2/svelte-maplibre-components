<script lang="ts">
	import type { Map } from 'maplibre-gl';
	import { onMount } from 'svelte';
	import { draggable } from '@neodrag/svelte';
	import type { DragOptions } from '@neodrag/svelte';
	import MapGenerator, { PageOrientation, Size, DPI, Format, Unit } from '$lib/utils/map-generator';
	import PrintableAreaManager from '$lib/utils/printable-area-manager';
	import CrosshairManager from '$lib/utils/crosshair-manager';
	import './style/fa/css/all.css';

	export let map: Map;

	let printableArea: PrintableAreaManager | undefined;
	let crosshairManager: CrosshairManager | undefined;

	let mapGenerator: MapGenerator;
	let printButton: HTMLButtonElement;

	export let showPrintableArea = true;
	export let showCrosshair = true;
	export let position: 'top-right' | 'top-left' | 'bottom-right' | 'bottom-left' = 'top-right';

	export let paperSize = Size.A4;
	export let dpi = DPI[96];
	export let format = Format.PNG;
	export let orientation = PageOrientation.Landscape;
	export let isShownSetting = true;

	$: paperSize, updatePrintableArea();
	$: orientation, updatePrintableArea();

	let isExportContainerShown = false;
	let dragOptions: DragOptions = {};

	// eslint-disable-next-line
	function MapExportControl() {}

	MapExportControl.prototype.onAdd = function (map: Map) {
		this.map = map;

		this.controlContainer = document.createElement('div');
		this.controlContainer.className = 'mapboxgl-ctrl mapboxgl-ctrl-group';
		printButton.addEventListener('click', () => {
			isExportContainerShown = !isExportContainerShown;
		});
		this.controlContainer.appendChild(printButton);
		return this.controlContainer;
	};

	MapExportControl.prototype.onRemove = function () {
		if (!this.controlContainer || !this.controlContainer.parentNode || !this.map || !this.button) {
			return;
		}
		this.controlContainer.parentNode.removeChild(this.controlContainer);
		this.map.off('click', this.onClick.bind(this));
		this.map = undefined;
	};

	/*global MapExportControl */
	/*eslint no-undef: "error"*/
	// eslint-disable-next-line @typescript-eslint/ban-ts-comment
	// @ts-ignore
	let mapExportControl: MapExportControl = null;

	$: {
		if (map) {
			if (mapExportControl !== null && map.hasControl(mapExportControl) === false) {
				map.addControl(mapExportControl, position);
			}
			dragOptions.bounds = map.getContainer();
		}
	}

	onMount(async () => {
		const { default: MapGenerator } = await import('./utils/map-generator');
		mapGenerator = new MapGenerator();

		// eslint-disable-next-line @typescript-eslint/ban-ts-comment
		// @ts-ignore
		mapExportControl = new MapExportControl();
	});

	const exportMap = () => {
		const actualPaperSize = getActualPaperSize();
		mapGenerator.generate(
			map,
			// eslint-disable-next-line @typescript-eslint/ban-ts-comment
			// @ts-ignore
			actualPaperSize,
			dpi,
			format,
			Unit.mm
		);
	};

	const getActualPaperSize = () => {
		let actualPaperSize = [paperSize[0], paperSize[1]];
		if (orientation !== PageOrientation.Landscape) {
			actualPaperSize = actualPaperSize.reverse();
		}
		return actualPaperSize;
	};

	$: isExportContainerShown, toggleButton();
	const toggleButton = () => {
		if (!printButton) return;
		if (isExportContainerShown) {
			printButton.classList.add('maplibre-ctrl-icon-active');
		} else {
			printButton.classList.remove('maplibre-ctrl-icon-active');
		}

		if (!showPrintableArea) {
			togglePrintableArea(false);
		} else {
			togglePrintableArea(isExportContainerShown);
		}
		if (!showCrosshair) {
			toggleCrosshair(false);
		} else {
			toggleCrosshair(isExportContainerShown);
		}
	};

	const togglePrintableArea = (state: boolean) => {
		if (state === false) {
			if (printableArea !== undefined) {
				printableArea.destroy();
				printableArea = undefined;
			}
		} else {
			printableArea = new PrintableAreaManager(map);
			updatePrintableArea();
		}
	};

	const updatePrintableArea = () => {
		if (printableArea === undefined) {
			return;
		}
		const actualPaperSize = getActualPaperSize();
		printableArea.updateArea(actualPaperSize[0], actualPaperSize[1]);
	};

	const toggleCrosshair = (state: boolean) => {
		if (!map) return;
		if (state === false) {
			if (crosshairManager !== undefined) {
				crosshairManager.destroy();
				crosshairManager = undefined;
			}
		} else {
			crosshairManager = new CrosshairManager(map);
			crosshairManager.create();
		}
	};
</script>

<button class="maplibre-ctrl-icon" bind:this={printButton} />

{#if isExportContainerShown}
	<nav class="panel is-success export-container" use:draggable={dragOptions}>
		<div class="panel-heading heading-control">
			<span class="icon drag-icon">
				<i class="fas fa-up-down-left-right" />
			</span>
			Export tool
			<!-- svelte-ignore a11y-click-events-have-key-events -->
			<span
				class="icon close-icon"
				on:click={() => {
					isExportContainerShown = !isExportContainerShown;
				}}
			>
				<i class="fas fa-xmark" />
			</span>
		</div>
		{#if isShownSetting}
			<div class="field">
				<!-- svelte-ignore a11y-label-has-associated-control -->
				<label class="label is-small">Paper Size</label>
				<div class="control has-icons-left">
					<div class="select is-small is-fullwidth">
						<select bind:value={paperSize}>
							{#each Object.keys(Size) as key}
								<option value={Size[key]}>{key}</option>
							{/each}
						</select>
					</div>
					<div class="icon is-small is-left">
						<i class="fas fa-file" />
					</div>
				</div>
			</div>
			<div class="field">
				<!-- svelte-ignore a11y-label-has-associated-control -->
				<label class="label is-small">Page Orientation</label>
				<div class="control">
					{#each Object.keys(PageOrientation) as key}
						<label class="radio" style="color:black">
							<input
								type="radio"
								name="orientation"
								on:click={() => {
									orientation = PageOrientation[key];
								}}
								checked={orientation === PageOrientation[key]}
							/>
							<div class="icon is-small is-left">
								<i
									class="fas {`${
										PageOrientation[key] === PageOrientation.Landscape
											? 'fa-left-right'
											: 'fa-up-down'
									}`}"
								/>
							</div>
							{key}
						</label>
					{/each}
				</div>
			</div>
			<div class="field">
				<!-- svelte-ignore a11y-label-has-associated-control -->
				<label class="label is-small">Format</label>
				<div class="control has-icons-left">
					<div class="select is-small is-fullwidth">
						<select bind:value={format}>
							{#each Object.keys(Format) as key}
								<option value={Format[key]}>{key}</option>
							{/each}
						</select>
					</div>
					<div class="icon is-small is-left">
						<i class="fas fa-file-pdf" />
					</div>
				</div>
			</div>
			<div class="field">
				<!-- svelte-ignore a11y-label-has-associated-control -->
				<label class="label is-small">DPI</label>
				<div class="control has-icons-left">
					<div class="select is-small is-fullwidth">
						<select bind:value={dpi}>
							{#each Object.keys(DPI) as key}
								<option value={DPI[key]}>{key}</option>
							{/each}
						</select>
					</div>
					<div class="icon is-small is-left">
						<i class="fas fa-braille" />
					</div>
				</div>
			</div>
		{/if}
		<div class="columns m-1 is-mobile">
			<div class="column is-10 p-0 m-0">
				<button class="button is-fullwidth is-small is-success p-0 m-0" on:click={exportMap}>
					<span class="icon">
						<i class="fas fa-download" />
					</span>
					<span>Export</span>
				</button>
			</div>
			<div class="column is-2 p-0 m-0">
				<button
					class="button is-fullwidth is-small is-light p-0 m-0"
					on:click={() => (isShownSetting = !isShownSetting)}
				>
					<span class="icon">
						<i class="fas fa-gear" />
					</span>
				</button>
			</div>
		</div>
	</nav>
{/if}

<style lang="scss">
	@import 'bulma/bulma.sass';

	.maplibre-ctrl-icon {
		background: url('data:image/svg+xml;charset=UTF-8,<svg id="Capa_1" enable-background="new 0 0 512 512" height="512" viewBox="0 0 512 512" width="512" xmlns="http://www.w3.org/2000/svg"><g><path d="m422.5 99v-24c0-41.355-33.645-75-75-75h-184c-41.355 0-75 33.645-75 75v24z"/><path d="m118.5 319v122 26 15c0 16.568 13.431 30 30 30h214c16.569 0 30-13.432 30-30v-15-26-122zm177 128h-80c-8.284 0-15-6.716-15-15s6.716-15 15-15h80c8.284 0 15 6.716 15 15s-6.716 15-15 15zm0-64h-80c-8.284 0-15-6.716-15-15s6.716-15 15-15h80c8.284 0 15 6.716 15 15s-6.716 15-15 15z"/><path d="m436.5 129h-361c-41.355 0-75 33.645-75 75v120c0 41.355 33.645 75 75 75h13v-80h-9c-8.284 0-15-6.716-15-15s6.716-15 15-15h24 304 24c8.284 0 15 6.716 15 15s-6.716 15-15 15h-9v80h14c41.355 0 75-33.645 75-75v-120c0-41.355-33.645-75-75-75zm-309 94h-48c-8.284 0-15-6.716-15-15s6.716-15 15-15h48c8.284 0 15 6.716 15 15s-6.716 15-15 15z"/></g></svg>');
		background-position: center;
		background-repeat: no-repeat;
		background-size: 70%;
	}

	.maplibre-ctrl-icon-active {
		background: url('data:image/svg+xml;charset=UTF-8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 320 512"><!--! Font Awesome Pro 6.2.0 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license (Commercial License) Copyright 2022 Fonticons, Inc. --><path d="M310.6 150.6c12.5-12.5 12.5-32.8 0-45.3s-32.8-12.5-45.3 0L160 210.7 54.6 105.4c-12.5-12.5-32.8-12.5-45.3 0s-12.5 32.8 0 45.3L114.7 256 9.4 361.4c-12.5 12.5-12.5 32.8 0 45.3s32.8 12.5 45.3 0L160 301.3 265.4 406.6c12.5 12.5 32.8 12.5 45.3 0s12.5-32.8 0-45.3L205.3 256 310.6 150.6z"/></svg>');
		background-position: center;
		background-repeat: no-repeat;
		background-size: 40%;
	}

	.export-container {
		position: absolute;
		background-color: white;
		padding: 10px;
		bottom: 10px;
		right: 10px;
		z-index: 10;
		cursor: grab;
		width: 260px;
	}

	.heading-control {
		position: relative;

		.close-icon {
			position: absolute;
			top: 15px;
			right: 10px;
			cursor: pointer;
		}
	}
</style>
