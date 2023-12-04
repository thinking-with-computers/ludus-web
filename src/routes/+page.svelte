<script>
 import { EditorState } from "@codemirror/state";
 import { basicSetup } from "codemirror";
 import { EditorView, keymap } from "@codemirror/view";
 import { indentWithTab } from "@codemirror/commands";
 import { onMount } from "svelte";
 import P5 from "p5-svelte";
 import { run } from "@ludus/ludus-js-pure";

 let editorState, view, ludusResponse = "";

 onMount(() => {
	 editorState = EditorState.create({
		 doc: "add(1, 2)",
		 extensions: [
			 basicSetup,
			 keymap.of([indentWithTab]),
		 ]
	 });

	 view = new EditorView({
		 state: editorState,
		 parent: document.getElementById("code-editor")
	 });
 })

 async function run_code () {
	 let raw_selection = view.state.selection.toJSON();
	 let selection = {start: raw_selection.ranges[0].anchor, end: raw_selection.ranges[0].head};
	 let code = (selection.end > selection.start) ? view.state.doc.toString().slice(selection.start, selection.end) : view.state.doc.toString();
	 let res = await run(code);
	 console.log("running: ", code);
	 console.log("result: ", res);
	 if (res.result) {
		 ludusResponse = "<p>" + res.result + "</p>" + ludusResponse;
	 }
 }

 const sketch = (p5) => {
	 let cnv;

	 p5.setup = () => {
		 let canvas_elt = document.getElementById("canv");
		 let h = canvas_elt.clientHeight;
		 let w = canvas_elt.clientWidth;
		 cnv = p5.createCanvas(w, h);
		 cnv.parent("canv");
		 console.log(cnv);
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
	<div id="console">
		{#if ludusResponse}
			{@html ludusResponse}
		{/if}
	</div>
</main>

<style>
 main {
	 display: grid;
	 width: 100%;
	 height: 100%;
	 grid-template-columns: 1fr 1fr;
	 grid-template-rows: 2em auto auto 12em;
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
 }

 #code-editor :global(.cm-editor) {
	 height: 100%;
 }

 #canv {
	 background: #000;
	 grid-area: canv;
	 padding: 0;
	 margin: 0;
	 overflow: hidden;
 }

 #console {
	 background: #000;
	 border-top: 2px solid white;
	 grid-area: console;
	 color: white;
	 padding: 2ch;
	 box-sizing: border-box;
	 overflow-y: scroll;
	 font-family: 'Courier New', Courier, monospace;
 }

 :global(.p5Canvas) {
	 width: 100%;
	 height: 100%;
 }
</style>
