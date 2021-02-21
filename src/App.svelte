<script>
	import CopyClipBoard from "./CopyClipBoard.svelte";

	let input;
	$: output = convert(input);

	const convert = (s) => {
		if (!s) return "";
		return parse(s)
			.map((s) => [s.question, s.choices].flat().join("\t"))
			.join("\n");
	};

	const sanitize = (s) =>
		s
			.replace("GO ON TO THE NEXT PAGE", "")
			.replace(/\d\s?\d\s?\d\s?\./, "")
			.replace(/\u00AD/, "") // remove unicode space
			.replace("■", "")
			.replace(/\s-+[\s,]/, " ANSWER ")
			.trim();

	const parse = (s) => {
		const arr = s
			.split(/\r?\n/)
			.map(sanitize)
			.filter((s) => s !== "");
		const problems = [];

		let p = "";

		for (const a of arr) {
			p += a;

			if (a.indexOf("(D)") === -1) continue;

			p = p.replace(/^[^a-zA-Z\-]*/, "");

			const pos = p.indexOf("(A)");
			problems.push({
				question: p.substring(0, pos),
				choices: p
					.substring(pos)
					.split(/\([AB8CD]\)/)
					.map((s) => s.trim())
					.filter((s) => s !== ""),
			});
			p = "";
		}

		return problems;
	};

	const copy = () => {
		const app = new CopyClipBoard({
			target: document.getElementById("clipboard"),
			props: { name: output },
		});
		app.$destroy();
	};
</script>

<main>
	<div>
		<h1>Input</h1>
		<textarea bind:value={input} />
	</div>

	<div>
		<h1>Output</h1>
		<button on:click={copy}>Copy!</button>
		<textarea bind:value={output} disabled />
	</div>
	<div id="clipboard" />
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	textarea {
		width: 100%;
		height: 200px;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
