<script lang="ts">
	export let onlyRendered = true;
	export let onlyRelative = true;
</script>

<div class="tabs">
	<input
		id="rendered"
		type="checkbox"
		name="rendered_tab"
		checked={onlyRendered}
		on:click={() => (onlyRendered = !onlyRendered)}
	/>
	<label class="tab_item" for="rendered">Show rendered</label>
	<input
		id="relative"
		type="checkbox"
		name="relative_tab"
		checked={onlyRelative}
		on:click={() => (onlyRelative = !onlyRelative)}
	/>
	<label class="tab_item" for="relative">Show related</label>
</div>

<style lang="scss">
	.tabs {
		margin-top: 0.5rem;
		padding-bottom: 0.5rem;
		box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
		width: 100%;
		margin: 0 auto;

		.tab_item {
			width: 50%;
			background-color: #f5f5f5;
			border-color: #b5b5b5;
			line-height: 2.5rem;
			font-size: 16px;
			text-align: center;
			color: #565656;
			display: block;
			float: left;
			text-align: center;
			font-weight: bold;
			transition: all 0.2s ease;
			font-family: system-ui, -apple-system, system-ui, 'Helvetica Neue', Helvetica, Arial,
				sans-serif;
			font-size: 16px;
			font-weight: 600;
		}
		.tab_item:hover {
			opacity: 0.75;
		}

		input {
			display: none;
		}
	}

	.tabs input:checked + .tab_item {
		background-color: #485fc7;
		border-color: #485fc7;
		color: #fff;
	}
</style>
