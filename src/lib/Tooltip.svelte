<script lang="ts">
	export let text: string;
</script>

<div class="tooltip-wrapper">
	<slot />
	<span class="tooltip-text">{text}</span>
</div>

<style>
	.tooltip-wrapper {
		position: relative;
		display: inline-block;
		width: 100%;
	}

	.tooltip-text {
		visibility: hidden;
		position: absolute;
		z-index: 100;
		bottom: calc(100% + 10px);
		left: 0;
		background-color: #333;
		color: white;
		text-align: left;
		padding: 8px 12px;
		border-radius: 6px;
		font-size: 14px;
		line-height: 1.4;
		width: max-content;
		max-width: 300px;
		opacity: 0;
		transition: opacity 0.2s;
		box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
		pointer-events: none;
	}

	/* Ensure tooltip stays within viewport */
	@media (max-width: 768px) {
		.tooltip-text {
			max-width: 250px;
			left: 50%;
			transform: translateX(-50%);
		}
	}

	.tooltip-text::after {
		content: '';
		position: absolute;
		bottom: -10px;
		left: 20px;
		border-width: 5px;
		border-style: solid;
		border-color: #333 transparent transparent transparent;
	}

	.tooltip-wrapper:hover .tooltip-text {
		visibility: visible;
		opacity: 1;
	}
</style>
