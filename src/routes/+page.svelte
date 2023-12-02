<script>
 import { EditorState } from "@codemirror/state";
 import { basicSetup } from "codemirror";
 import { EditorView, keymap } from "@codemirror/view";
 import { indentWithTab } from "@codemirror/commands";
 import { onMount } from "svelte";
 import P5 from "p5-svelte";
 import { run } from "@ludus/ludus-js-pure";

 onMount(() => {
	 let startState = EditorState.create({
		 doc: "Hello World",
		 extensions: [
			 basicSetup,
			 keymap.of([indentWithTab]),
		 ]
	 });

	 let view = new EditorView({
		 state: startState,
		 parent: document.getElementById("code-editor")
	 });
 })

 function run_code () {
	 console.log("RUNNING AS FAST AS I CAN!");
	 console.log(run("add(1, 2)"));
 }

 const sketch = (p5) => {
	 p5.setup = () => {

	 }

	 p5.draw = () => {

	 }
 }

</script>

<main>
	<header>
		<h1>Ludus</h1>
		<a href="/" on:click|preventDefault={run_code}>RUN</a>
	</header>
	<div id="code-editor"></div>
	<div id="canv">
		<P5 {sketch} />
	</div>
	<div id="console"></div>
</main>

<style>
 main {
	 display: grid;
	 width: 100%;
	 height: 100%;
	 grid-template-columns: 1fr 1fr;
	 grid-template-rows: 2em auto;
	 grid-template-areas:
		 "header header"
		 "editor canv"
		 "editor canv"
		 "editor console";
 }

 header {
	 grid-area: header;
	 height: 2em;
	 display: flex;
	 align-items: baseline;
	 justify-content: space-around;
 }

 header h1 {
	 font-size: 1em;
 }

 header a {
	 display: block;
 }

 #code-editor {
	 min-height: 4em;
	 grid-area: editor;
	 height: 100%;
 }

 #code-editor :global(.cm-editor) {
	 height: 100%;
 }

 #canv {
	 height: 100%;
	 background: #000;
	 grid-area: canv;
 }

 #console {
	 background: #000;
	 border-top: 2px solid white;
	 height: 100%;
	 grid-area: console;
 }

 :global(.p5Canvas) {
	 width: 100%;
	 height: 100%;
 }
</style>
