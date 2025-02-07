<script>import { fly } from "svelte/transition";
import { cubicInOut } from "svelte/easing";
import { toText } from "./date-utils.js";
import { parse, createFormat } from "./parse.js";
import DateTimePicker from "./DatePicker.svelte";
import { writable } from "svelte/store";
import { createEventDispatcher } from "svelte";
const dispatch = createEventDispatcher();
const defaultDate = /* @__PURE__ */ new Date();
const innerStore = writable(null);
const store = (() => {
  return {
    subscribe: innerStore.subscribe,
    set: (date) => {
      if (date === null || date === void 0) {
        innerStore.set(null);
        value = date;
      } else if (date.getTime() !== $innerStore?.getTime()) {
        innerStore.set(date);
        value = date;
      }
    }
  };
})();
export let value = null;
$:
  store.set(value);
export let min = new Date(defaultDate.getFullYear() - 20, 0, 1);
export let max = new Date(defaultDate.getFullYear(), 11, 31, 23, 59, 59, 999);
export let placeholder = "2020-12-31 23:00:00";
export let valid = true;
export let disabled = false;
let classes = "";
export { classes as class };
export let format = "yyyy-MM-dd HH:mm:ss";
let formatTokens = createFormat(format);
$:
  formatTokens = createFormat(format);
export let locale = {};
function valueUpdate(value2, formatTokens2) {
  text = toText(value2, formatTokens2);
}
$:
  valueUpdate($store, formatTokens);
export let text = toText($store, formatTokens);
function textUpdate(text2, formatTokens2) {
  if (text2.length) {
    const result = parse(text2, formatTokens2, $store);
    if (result.date !== null) {
      valid = true;
      store.set(result.date);
    } else {
      valid = false;
    }
  } else {
    valid = true;
    if (value) {
      value = null;
      store.set(null);
    }
  }
}
$:
  textUpdate(text, formatTokens);
export let visible = false;
export let closeOnSelection = false;
export let browseWithoutSelecting = false;
export let timePrecision = null;
function onFocusOut(e) {
  if (e?.currentTarget instanceof HTMLElement && e.relatedTarget && e.relatedTarget instanceof Node && e.currentTarget.contains(e.relatedTarget)) {
    return;
  } else {
    visible = false;
  }
}
function keydown(e) {
  if (e.key === "Escape" && visible) {
    visible = false;
    e.preventDefault();
    e.stopPropagation();
  } else if (e.key === "Enter") {
    visible = !visible;
    e.preventDefault();
  }
}
function onSelect(e) {
  dispatch("select", e.detail);
  if (closeOnSelection) {
    visible = false;
  }
}
export let dynamicPositioning = false;
let InputElement;
let pickerElement;
let showAbove = false;
let pickerLeftPosition = null;
function setDatePickerPosition() {
  showAbove = false;
  pickerLeftPosition = null;
  if (visible && pickerElement && dynamicPositioning) {
    const inputRect = InputElement.getBoundingClientRect();
    const horizontalOverflow = pickerElement.offsetWidth - inputRect.width;
    const bottomThreshold = inputRect.bottom + pickerElement.offsetHeight + 5;
    const rightThreshold = inputRect.left + pickerElement.offsetWidth + 5;
    if (bottomThreshold > window.innerHeight) {
      showAbove = true;
    }
    if (rightThreshold > window.innerWidth) {
      pickerLeftPosition = -horizontalOverflow;
      if (inputRect.left < horizontalOverflow + 5) {
        const windowCenterPos = window.innerWidth / 2;
        const newPos = windowCenterPos - pickerElement.offsetWidth / 2;
        pickerLeftPosition = newPos - inputRect.left;
      }
    }
  }
}
function flyAutoPosition(node) {
  setDatePickerPosition();
  return fly(node, {
    duration: 200,
    easing: cubicInOut,
    y: showAbove ? 5 : -5
  });
}
</script>

<!-- svelte-ignore a11y-no-static-element-interactions -->
<div class="date-time-field {classes}" on:focusout={onFocusOut} on:keydown={keydown}>
	<input
		bind:this={InputElement}
		class:invalid={!valid}
		type="text"
		value={text}
		{placeholder}
		{disabled}
		on:focus={() => (visible = true)}
		on:mousedown={() => (visible = true)}
		on:input={(e) => {
			if (
				e instanceof InputEvent &&
				e.inputType === 'insertText' &&
				typeof e.data === 'string' &&
				e.currentTarget.value === text + e.data
			) {
				// check for missing punctuation, and add if there is any
				let result = parse(text, formatTokens, $store)
				if (result.missingPunctuation !== '' && !result.missingPunctuation.startsWith(e.data)) {
					text = text + result.missingPunctuation + e.data
					return
				}
			}
			text = e.currentTarget.value
		}}
	/>
	{#if visible && !disabled}
		<div
			class="picker"
			class:visible
			class:above={showAbove}
			transition:flyAutoPosition
			bind:this={pickerElement}
			style:--picker-left-position="{pickerLeftPosition}px"
		>
			<DateTimePicker
				on:focusout={onFocusOut}
				on:select={onSelect}
				bind:value={$store}
				{min}
				{max}
				{locale}
				{browseWithoutSelecting}
				{timePrecision}
			/>
		</div>
	{/if}
</div>

<style>.date-time-field {
  position: relative;
}

input {
  color: var(--date-picker-foreground, #000000);
  background: var(--date-picker-background, #ffffff);
  min-width: 0px;
  box-sizing: border-box;
  padding: 4px 6px;
  margin: 0px;
  border: 1px solid rgba(103, 113, 137, 0.3);
  border-radius: 3px;
  width: var(--date-input-width, 150px);
  outline: none;
  transition: all 80ms cubic-bezier(0.4, 0, 0.2, 1);
}
input:focus {
  border-color: var(--date-picker-highlight-border, #0269f7);
  box-shadow: 0px 0px 0px 2px var(--date-picker-highlight-shadow, rgba(2, 105, 247, 0.4));
}
input:disabled {
  opacity: 0.5;
}

.invalid {
  border: 1px solid rgba(249, 47, 114, 0.5);
  background-color: rgba(249, 47, 114, 0.1);
}
.invalid:focus {
  border-color: #f92f72;
  box-shadow: 0px 0px 0px 2px rgba(249, 47, 114, 0.5);
}

.picker {
  display: none;
  position: absolute;
  padding: 1px;
  left: var(--picker-left-position);
  z-index: 10;
}
.picker.above {
  bottom: 100%;
}
.picker.visible {
  display: block;
}</style>
