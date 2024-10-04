<script lang="ts">
	import { withPrevious } from '$lib/stores/previous.svelte';

	const { maxLength = 6, defaultValue = '' } = $props();
	const [newValue, previousValue] = withPrevious(defaultValue);

	let inputElement: HTMLInputElement;
	let selectionStart: number = $state(0);
	let selectionEnd: number = $state(0);

	const handleSelectionChange = () => {
		if (inputElement) {
			selectionStart = inputElement.selectionStart ?? 0;
			selectionEnd = inputElement.selectionEnd ?? 0;
		}
	}

	$effect(() => {
		if (inputElement &&typeof $previousValue === 'string' && $newValue.length < $previousValue.length) {
			inputElement.dispatchEvent(new Event('selectionchange'));
		}
	});
</script>

current: {$newValue} - previous: {$previousValue}

start: {selectionStart} - end: {selectionEnd}

<input
	onselectionchange={handleSelectionChange}
	bind:this={inputElement}
	bind:value={$newValue}
	maxlength={maxLength}
/>
