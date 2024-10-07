<script lang="ts">
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
	}

	const { index, class: className, focusClass: focusClassName }: Props = $props();
</script>

{#if typeof index === 'number'}
	<div
		class={(globalInsertMode && $globalSelectionStart === index && $globalSelectionEnd === index) ||
		(!$globalInsertMode && index >= $globalSelectionStart && index < $globalSelectionEnd)
			? className + ' ' + focusClassName
			: className}
	>
		{$globalNewValue[index]}
	</div>
{:else}
	{#each index as index}
		<div
			class={
				(globalInsertMode && $globalSelectionStart === index && $globalSelectionEnd === index) ||
				(!$globalInsertMode && index >= $globalSelectionStart && index < $globalSelectionEnd)
					? className + ' ' + focusClassName
					: className
			}
		>
			{$globalNewValue[index]}
		</div>
	{/each}
{/if}
