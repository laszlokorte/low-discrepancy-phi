<script>
	import favicon from "/favicon.svg";
	import * as L from "partial.lenses";
	import * as R from "ramda";
	import { atom, view, read, failableView } from "./svatom.svelte.js";
	import { forcePlain } from "./contenteditable.js";
	import { lerp, clamp, PHI } from "./utils.js";

	const intercept = atom(0.0);
	const total = atom(50);
	const slope = atom(PHI);
	const svgSize = atom({ w: 0, h: 0 });
	const svgWidth = view("w", svgSize);
	const svgHeight = view("h", svgSize);

	const generator = R.unfold(({ last, count }) =>
		count > 0
			? [
					(((last + slope.value) % 1) - 0.5) * 2,
					{ last: (last + slope.value) % 1, count: count - 1 },
				]
			: false,
	);
	const samples = $derived(
		generator({
			last: intercept.value + ((total.value / slope.value) % 1),
			count: total.value * 2,
		}),
	);

	const padding = 30;

	const viewBox = $derived(
		`${-svgWidth.value / 2 + padding} ${-svgHeight.value / 2 + padding} ${svgWidth.value - 2 * padding} ${svgHeight.value - 2 * padding}`,
	);
</script>

<section>
	<header>
		<h1>
			<img src={favicon} class="icon" alt="Icon" />Simple sequence of well
			distributed numbers
		</h1>

		<details>
			<summary>Introduction</summary>

			<p>
				This is an showcase of how to generate a <a
					href="https://en.wikipedia.org/wiki/Low-discrepancy_sequence"
					>Low-discrepancy sequence</a
				> of numbers by simply sampling a line that has the golden ratio
				(ϕ=1.61803&hellip;) as slope:
			</p>

			<math>
				<mrow>
					<mn>r</mn><mo>=</mo><mtext>mod</mtext><mo>(</mo><mn>a</mn
					><mo>+</mo><mo><mn>b</mn><mo>&centerdot;</mo><mn>n</mn>,</mo
					><mn>1</mn><mo>)</mo>
					<mtext
						>&nbsp;with&nbsp;<mrow>
							<mn>b</mn>
							<mo>=</mo>
							<mn>ϕ</mn></mrow
						></mtext
					>
				</mrow>
			</math>

			<p>
				You can also try other slopes to observe how the perceived
				randomness vanishes. Some other numbers than ϕ do also work
				quite well. <br />π does not.
			</p>

			<p>
				Using such a sequence of numbers might be useful if you want to
				generate a new (pseudo) random value from a limited range that
				is as distinct as possible from all previously generated values.
			</p>
			<p>
				Say you want to plot multiple data points in distinct colors and
				new to select a new color for each point but do not know how
				many data points there will be in advanced.
			</p>
		</details>

		<label class="number-picker"
			>Intercept (a): <input
				type="range"
				bind:value={intercept.value}
				min="0"
				max="1"
				step="0.01"
			/><input
				type="number"
				size="7"
				bind:value={intercept.value}
				min="0"
				max="1"
				step="0.01"
			/></label
		>
		<label class="number-picker"
			>Slope (b): <input
				type="range"
				bind:value={slope.value}
				min="0"
				max="6"
				step="0.001"
			/>
			<input
				type="number"
				size="7"
				bind:value={slope.value}
				min="0"
				max="6"
				step="0.001"
			/>
		</label>

		<div>
			<button onclick={() => (slope.value = PHI)}>ϕ</button>
			<button onclick={() => (slope.value = 1 / PHI)}>1/ϕ</button>
			<button onclick={() => (slope.value = 2 * PHI)}>ϕ</button>
			<button onclick={() => (slope.value = Math.PI * PHI)}
				>π&centerdot;ϕ</button
			>
			<button onclick={() => (slope.value = PHI * PHI)}
				>ϕ&centerdot;ϕ</button
			>
			<button onclick={() => (slope.value = Math.PI)}>π</button>
		</div>

		<label class="number-picker"
			>Number of Samples: <input
				type="range"
				bind:value={total.value}
				min="10"
				max="200"
				step="1"
			/></label
		>
	</header>

	<svg
		class="svg"
		height="300"
		bind:clientWidth={svgWidth.value}
		bind:clientHeight={svgHeight.value}
		{viewBox}
	>
		{#each samples as s, i (i)}
			<circle
				r="8"
				stroke="#555"
				stroke-width="2"
				cx={lerp(
					-svgWidth.value / 2,
					svgWidth.value / 2,
					i / samples.length,
				)}
				fill="hsl({360 * (s / 2)},100%,50%)"
				cy={-(s / 2) * svgHeight.value}
			></circle>
		{/each}
	</svg>
	<svg
		class="svg"
		height="300"
		bind:clientWidth={svgWidth.value}
		bind:clientHeight={svgHeight.value}
		{viewBox}
	>
		{#each samples as s, i (i)}
			<circle
				r="4"
				stroke="#555"
				stroke-width="1"
				cx={lerp(
					-svgWidth.value / 2,
					svgWidth.value / 2,
					i / samples.length,
				)}
				fill="hsl({360 * (s / 2)},100%,50%)"
				cy={-(s / 2) * svgHeight.value * 0.6 + 0.2}
			></circle>
		{/each}
	</svg>

	<svg class="svg" height="300" {viewBox}>
		{#each samples as s, i (i)}
			<rect
				width={svgWidth.value / samples.length}
				height={svgHeight.value / 3}
				y={svgHeight.value * (-1 / 2 + 0 / 3)}
				x={lerp(
					-svgWidth.value / 2,
					svgWidth.value / 2,
					i / samples.length,
				)}
				fill="hsl({360 * (i / samples.length)},0%,{(0.5 + s / 2) * 80 +
					10}%)"
			></rect>
			<rect
				width={svgWidth.value / samples.length}
				height={svgHeight.value / 3}
				y={svgHeight.value * (-1 / 2 + 1 / 3)}
				x={lerp(
					-svgWidth.value / 2,
					svgWidth.value / 2,
					i / samples.length,
				)}
				fill="hsl({360 * (i / samples.length)},50%,{(0.5 + s / 2) * 80 +
					10}%)"
			></rect>
			<rect
				width={svgWidth.value / samples.length}
				height={svgHeight.value / 3}
				y={svgHeight.value * (-1 / 2 + 2 / 3)}
				x={lerp(
					-svgWidth.value / 2,
					svgWidth.value / 2,
					i / samples.length,
				)}
				fill="hsl({360 * (i / samples.length)},100%,{(0.5 + s / 2) *
					80 +
					10}%)"
			></rect>
		{/each}
	</svg>

	<svg class="svg" height="300" {viewBox}>
		{#each samples as s, i (i)}
			<rect
				width={svgWidth.value / samples.length}
				height={svgHeight.value / 3}
				y={svgHeight.value * (-1 / 2 + 0 / 3)}
				x={lerp(
					-svgWidth.value / 2,
					svgWidth.value / 2,
					i / samples.length,
				)}
				fill="hsl({360 * (0.5 + s / 2)},90%,50%)"
			></rect>
			<rect
				width={svgWidth.value / samples.length}
				height={svgHeight.value / 3}
				y={svgHeight.value * (-1 / 2 + 1 / 3)}
				x={lerp(
					-svgWidth.value / 2,
					svgWidth.value / 2,
					i / samples.length,
				)}
				fill="hsl({360 * (0.5 + s / 2)},{100 *
					(i / samples.length)}%,50%)"
			></rect>
			<rect
				width={svgWidth.value / samples.length}
				height={svgHeight.value / 3}
				y={svgHeight.value * (-1 / 2 + 2 / 3)}
				x={lerp(
					-svgWidth.value / 2,
					svgWidth.value / 2,
					i / samples.length,
				)}
				fill="hsl({360 * (0.5 + s / 2)},50%,{100 *
					(i / samples.length)}%)"
			></rect>
		{/each}
	</svg>

	<footer>
		<a href="https://tools.laszlokorte.de" title="More educational tools"
			>More educational tools</a
		>
	</footer>
</section>

<style>
	.svg {
		width: 100%;
		background: #eee;
	}

	summary {
		cursor: pointer;
	}

	header {
		position: sticky;
		top: 0em;
		background: #fffe;
		padding: 2em 1em 1em;
		margin-bottom: 1em;
	}

	details {
		padding: 0.5em 1em;
		transition: background 0.08s ease;
	}

	details[open] {
		background: #fefaf0;
	}

	summary {
		padding: 0.4em 0;
		cursor: pointer;
		user-select: none;
	}

	summary > span {
		text-decoration: underline;
	}

	hr {
		background: none;
		border: none;
		border-top: 1px solid #333;
	}

	footer {
		text-align: center;
		padding: 2em;
		border-top: 1px solid #aaa;
		margin-top: 2em;
	}

	.icon {
		height: 2em;
		width: 2em;
		display: inline-block;
		vertical-align: middle;
		margin: 0 0.5em 0 0;
	}

	h1 {
		margin: 0;
	}

	section {
		margin: 0 auto 3em;
		max-width: 60em;
	}

	.code-snippet {
		display: block;
		background: #333;
		color: #fff;
		padding: 1em;
		font-family: monospace;
		font-size: 1.3em;
		line-height: 1.4;
	}

	a {
		color: #15218d;
	}

	.code-template {
		background: #15218d;
		color: #fff;
		padding: 1em;
		font-size: 2em;
		font-family: monospace;
		display: grid;
		grid-template-columns: auto auto auto;
		justify-content: start;
		align-items: baseline;
	}

	.code-template-start {
		grid-column: 1;
	}

	.code-template-content {
		grid-column: 2;
	}

	.code-template-end {
		grid-column: 3;
	}

	.code-template-prompt {
		grid-column: 2;
		grid-row: 2;
		text-align: center;
		align-items: center;
		font-size: 0.5em;
		color: #cceeff;
		user-select: none;
	}

	.code-placeholder {
		min-width: 1em;
		display: inline-block;
		background: #cceeff;
		color: #000;
		padding: 0.2em 0.4em;
		margin: 0.2em;
	}

	.syntax-error {
		color: #aa0000;
		outline: #aa0000 3px solid;
		background: #ffdddd;
	}

	.error-message {
		padding: 1em;
		color: #aa0000;
		background: #ffdddd;
	}

	ul {
		list-style: none;
		padding: 0;
		margin: 0;
		display: flex;
		gap: 0.5em;
		flex-direction: column;
	}

	pre {
		white-space: pre-wrap;
		background: #333;
		color: #fff;
		overflow: auto;
		resize: vertical;
		padding: 1em;
		box-sizing: border-box;
	}

	textarea {
		white-space: pre-wrap;
		background: #ffffee;
		color: #000;
		width: 100%;
		min-height: 10em;
		border: 0;
		resize: vertical;
		padding: 1em;
		box-sizing: border-box;
	}

	.number-picker {
		display: flex;
		align-items: center;
		gap: 1em;
	}

	input[type="range"] {
		padding: 1em;
		margin: 0;
	}

	input[type="text"] {
		margin: 0;
	}

	.phantom {
		visibility: hidden;
	}

	.controls {
		display: flex;
		margin: 1em 0;
	}

	button {
		border: none;
		background: #2541ad;
		color: #fff;
		padding: 0.3em 0.5em;
		display: inline-block;
		font: inherit;
		cursor: pointer;
	}

	button:hover {
		background: #2562ad;
	}

	button:active {
		background: #252aad;
	}

	button:focus-visible {
		outline: 3px solid #4daace;
	}

	.button-bar {
		display: flex;
		gap: 0.2em;
		margin: 4px 0;
		align-items: baseline;
	}

	.button-bar-intro {
		padding-right: 0.5em;
	}
</style>
