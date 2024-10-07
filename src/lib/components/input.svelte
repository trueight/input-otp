<script lang="ts">
	import { withPrevious } from '$lib/stores/stores.js';
	import {
		globalNewValue,
		globalInsertMode,
		globalSelectionStart,
		globalSelectionEnd
	} from '$lib/stores/stores.js';
	import type { Snippet } from 'svelte';
	type SelectionDirection = 'none' | 'forward' | 'backward';
	type SelectionState = [number, number, SelectionDirection | undefined];
	interface Props {
		maxLength: number;
		value?: string;
		children: Snippet;
		class?: string;
		inputZIndex?: number;
	}

	let inputElement: HTMLInputElement;
	let { maxLength = 6, value = $bindable(''), children, class: className, inputZIndex = 30 }: Props = $props();
	const [newValue, previousValue] = withPrevious(value);
	const [newSelectionStart, previousSelectionStart] = withPrevious(0);
	const [newSelectionEnd, previousSelectionEnd] = withPrevious(0);
	const [newSelectionDirection, previousSelectionDirection] = withPrevious('none');
	const previousSelectionState: SelectionState = [
		$previousSelectionStart ?? 0,
		$previousSelectionEnd ?? 0,
		($previousSelectionDirection as SelectionDirection) ?? undefined
	];

	const updateSelectionRange = (
		start: number,
		end: number,
		direction: SelectionDirection | undefined
	) => {
		if (start !== -1 && end !== -1 && start !== end) {
			inputElement.setSelectionRange(start, end, direction);
		}
	};

	const isSingleCaret = (newSelectionStart: number, newSelectionEnd: number): boolean =>
		newSelectionStart === newSelectionEnd;

	const isInsertMode = (newSelectionStart: number, newValue: string): boolean =>
		newSelectionStart === newValue.length && newValue.length < maxLength;

	const onSelectionChange = () => {
		let selectionStart: number = -1;
		let selectionEnd: number = -1;
		let selectionDirection: SelectionDirection | undefined = undefined;

		if ($newValue.length !== 0) {
			const caretIsSingle = isSingleCaret($newSelectionStart, $newSelectionEnd);
			const insertMode = isInsertMode($newSelectionStart, $newValue);
			$globalInsertMode = insertMode;

			if (caretIsSingle && !insertMode) {
				const caretPosition = $newSelectionStart;
				if (caretPosition === 0) {
					selectionStart = 0;
					selectionEnd = 1;
					selectionDirection = 'forward';
				} else if (caretPosition === maxLength) {
					selectionStart = caretPosition - 1;
					selectionEnd = caretPosition;
					selectionDirection = 'backward';
				} else if (maxLength > 1 && $newValue.length > 1) {
					let offset = 0;
					const [previousStart, previousEnd] = previousSelectionState;
					const wasPreviouslyInserting = previousStart === previousEnd && previousStart < maxLength;

					selectionDirection = caretPosition < previousEnd ? 'backward' : 'forward';

					if (!wasPreviouslyInserting && selectionDirection === 'backward') {
						offset = -1;
					}

					selectionStart = caretPosition + offset;
					selectionEnd = caretPosition + offset + 1;
				}
			}
			updateSelectionRange(selectionStart, selectionEnd, selectionDirection);
		}
		const s = selectionStart !== -1 ? selectionStart : $newSelectionStart;
		const e = selectionEnd !== -1 ? selectionEnd : $newSelectionEnd;
		const d = selectionDirection ?? $newSelectionDirection;

		previousSelectionState[0] = s;
		previousSelectionState[1] = e;
		previousSelectionState[2] = d as SelectionDirection | undefined;
		$globalSelectionStart = s;
		$globalSelectionEnd = e;
	};

	const handleSelectionChange = () => {
		if (inputElement) {
			$newSelectionStart = inputElement.selectionStart ?? 0;
			$newSelectionEnd = inputElement.selectionEnd ?? 0;
		}

		if (inputElement === document.activeElement) {
			onSelectionChange();
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

		if ($newValue !== $previousValue) {
			value = $newValue;
			$globalNewValue = $newValue;
		}
	});
</script>

<div class={className} style="display: flex; position: relative; ">
	<input
		style="position: absolute; inset: 0; width: 100%; height: 100%; opacity: 0; z-index: {inputZIndex};"
		onselectionchange={handleSelectionChange}
		bind:this={inputElement}
		bind:value={$newValue}
		maxlength={maxLength}
	/>

	{@render children()}
</div>
