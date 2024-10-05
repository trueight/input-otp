<script lang="ts">
	import { withPrevious } from '$lib/stores/previous.svelte';
	import Monitor from './monitor.svelte';

	const { maxLength = 6, defaultValue = '' } = $props();
	const [newValue, previousValue] = withPrevious(defaultValue);

	let inputElement: HTMLInputElement;
	const [newSelectionStart, previousSelectionStart] = withPrevious(0);
	const [newSelectionEnd, previousSelectionEnd] = withPrevious(0);
	const [newDirection, previousDirection] = withPrevious('none');

	const handleSelectionChange = () => {
		if (inputElement) {
			$newSelectionStart = inputElement.selectionStart ?? 0;
			$newSelectionEnd = inputElement.selectionEnd ?? 0;
		}
	};

	$effect(() => {
		if (
			inputElement &&
			typeof $previousValue === 'string' &&
			$newValue.length < $previousValue.length
		) {
			inputElement.dispatchEvent(new Event('selectionchange'));
		}
	});
</script>

<input
	onselectionchange={handleSelectionChange}
	bind:this={inputElement}
	bind:value={$newValue}
	maxlength={maxLength}
/>

<Monitor
	{maxLength}
	newValue={$newValue}
	newSelectionStart={$newSelectionStart}
	newSelectionEnd={$newSelectionEnd}
/>
