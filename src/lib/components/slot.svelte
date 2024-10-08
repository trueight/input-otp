<script lang="ts">
	import type { Component } from 'svelte';
	import {
		globalNewValue,
		globalInsertMode,
		globalSelectionStart,
		globalSelectionEnd
	} from '$lib/stores/stores.js';
	interface Props {
		index: number[] | number;
		class?: string;
		focusClass?: string;
		Caret?: Component;
	}

	const {
		index,
		class: className,
		focusClass,
		Caret
	}: Props = $props();

	const getClass = (index: number) => {
		const single =
			$globalInsertMode &&
			$globalSelectionStart === index &&
			$globalSelectionEnd === index
				? true
				: false;

		const range =
			!$globalInsertMode &&
			index >= $globalSelectionStart &&
			index < $globalSelectionEnd
				? true
				: false;

		return single || range ? className + ' ' + focusClass : className;
	};

	const getCaret = (index: number) => {
		const showCaret =
			$globalInsertMode &&
			$globalSelectionStart === index &&
			$globalSelectionEnd === index &&
			Caret &&
			!$globalNewValue[index]
				? true
				: false;

		return showCaret;
	};
</script>

{#if typeof index === 'number'}
	<div class={getClass(index)}>
		{#if getCaret(index)}
			<Caret />
		{:else}
			{$globalNewValue[index]}
		{/if}
	</div>
{:else}
	{#each index as index}
		<div class={getClass(index)}>
			{#if getCaret(index)}
				<Caret />
			{:else}
				{$globalNewValue[index]}
			{/if}
		</div>
	{/each}
{/if}
