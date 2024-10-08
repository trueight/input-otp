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
	type SelectionState = [
		number,
		number,
		SelectionDirection | undefined
	];
	interface Props {
		maxLength: number;
		value?: string;
		children: Snippet;
		class?: string;
		inputZIndex?: number;
		onComplete?: () => void;
		onChange?: () => void;
		onFocus?: () => void;
		onBlur?: () => void;
		autofocus?: boolean;
		disabled?: boolean;
	}

	let inputElement: HTMLInputElement;
	let {
		maxLength = 6,
		value = $bindable(''),
		children,
		class: className,
		inputZIndex = 30,
		onComplete,
		onChange,
		onFocus,
		onBlur,
		autofocus,
		disabled
	}: Props = $props();
	const [newValue, previousValue] = withPrevious(value);
	const [newSelectionStart, previousSelectionStart] = withPrevious(0);
	const [newSelectionEnd, previousSelectionEnd] = withPrevious(0);
	const [newSelectionDirection, previousSelectionDirection] =
		withPrevious('none');
	const previousSelectionState: SelectionState = [
		$previousSelectionStart ?? 0,
		$previousSelectionEnd ?? 0,
		($previousSelectionDirection as SelectionDirection) ?? undefined
	];

	// initial global state
	$globalSelectionStart = $newSelectionStart;
	$globalSelectionEnd = $newSelectionEnd;
	$globalInsertMode = true;

	const updateSelectionRange = (
		start: number,
		end: number,
		direction: SelectionDirection | undefined
	) => {
		if (start !== -1 && end !== -1 && start !== end) {
			inputElement.setSelectionRange(start, end, direction);
		}
	};

	const isSingleCaret = (
		newSelectionStart: number,
		newSelectionEnd: number
	): boolean => newSelectionStart === newSelectionEnd;

	const isInsertMode = (
		newSelectionStart: number,
		newValue: string
	): boolean =>
		newSelectionStart === newValue.length &&
		newValue.length < maxLength;

	//I'm not satisfied with having to use global states, previous states, local states, arrays with more states, and all that
	//I think it needs a refactoring, if you know a way to improve this, I'd be happy to hear

	const onSelectionChange = () => {
		let selectionStart: number = -1;
		let selectionEnd: number = -1;
		let selectionDirection: SelectionDirection | undefined =
			undefined;

		if ($newValue.length !== 0) {
			const caretIsSingle = isSingleCaret(
				$newSelectionStart,
				$newSelectionEnd
			);
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
					const wasPreviouslyInserting =
						previousStart === previousEnd &&
						previousStart < maxLength;

					selectionDirection =
						caretPosition < previousEnd ? 'backward' : 'forward';

					if (
						!wasPreviouslyInserting &&
						selectionDirection === 'backward'
					) {
						offset = -1;
					}

					selectionStart = caretPosition + offset;
					selectionEnd = caretPosition + offset + 1;
				}
			}
			updateSelectionRange(
				selectionStart,
				selectionEnd,
				selectionDirection
			);
		}
		const s =
			selectionStart !== -1 ? selectionStart : $newSelectionStart;
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

	const onFocusChange = {
		focus: () => {
			if (onFocus) {
				onFocus();
			}
		},
		blur: () => {
			if (onBlur) {
				onBlur();
			}
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

		if ($newValue.length === maxLength && onComplete) {
			onComplete();
		}

		if ($newValue && $previousValue !== $newValue && onChange) {
			onChange();
		}
	});
</script>

<div
	class={disabled
		? 'input__container__default ' + className + ' disabled'
		: 'input__container__default ' + className}
>
	<!-- svelte-ignore a11y_autofocus -->
	<input
		style={inputZIndex ? `z-index: ${inputZIndex};` : ''}
		class="input__default"
		{disabled}
		onselectionchange={handleSelectionChange}
		{autofocus}
		onfocus={onFocusChange.focus}
		onblur={onFocusChange.blur}
		bind:this={inputElement}
		bind:value={$newValue}
		maxlength={maxLength}
	/>

	{@render children()}
</div>

<style>
	.input__container__default {
		display: flex;
		position: relative;
	}

	.input__default {
		position: absolute;
		inset: 0;
		width: 100%;
		height: 100%;
		opacity: 0;
		z-index: 30;
	}

	.disabled {
		opacity: 0.5;
		pointer-events: none;
	}
</style>
