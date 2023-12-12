<script>
 import { EditorState } from "@codemirror/state";
 import { basicSetup } from "codemirror";
 import { EditorView, keymap } from "@codemirror/view";
 import { indentWithTab } from "@codemirror/commands";
 import { onMount } from "svelte";
 import P5 from "p5-svelte";
 import { run } from "@ludus/ludus-js-pure";

 /*
		TODO:
	* [ ] change editor font to Victor Mono (we want ligatures) -> Matt
		- see: https://codemirror.net/examples/styling/
	* [ ] get Victor Mono cursive italics wired up -> Matt
	* [ ] keep editor at fixed width, add line wrapping -> Matt
	* [ ] add syntax highlighting (Lezer parser is almost ready to go) -> Scott
	* [ ] figure out a documentation scheme -> Matt
	* [ ] figure out how to redraw things on resize -> Matt
	* [ ] write a "tour" of ludus with different code -> ?
	* [ ] wire up a system whereby the code is loaded from a file, not hardcoded into js/svelte -> Matt
	* [ ] improve console formatting -> Matt
	* [ ] run only selected code -> Matt

	*/

 let editorState, view, ludusResponse = "";

 onMount(() => {
	 editorState = EditorState.create({
		 doc: `& Welcome to Ludus!
& to run the code in this window, hit the RUN button
& Ludus will evaluate this script and return its value
& lines that begin with an ampersand--&--are comments
& comments don't have any effect

& here's your usual first program:
print! ("Hello, world!")

& the message and return value appear in the bottom right pane

& ... here are a few other things you can do

& send the turtle forward 100 paces:
& forward! (100)

& turn the turtle right a quarter turn:
& right! (0.25)

& and, if you're already thinking about Logo:
& repeat 4 {
&	forward! (100)
&	right! (0.25)
& }
		 `,
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

 let p;

 function draw_turtle (draw) {
 	 p.push();
 	 p.translate(p.width / 2, p.height / 2);
 	 p.rotate(p.PI);
 	 for (const [method, ...args] of draw) {
 		 p[method](...args);
 	 }
 	 p.pop();
 }

 function print_console (msgs) {
 	 for (const raw_msg of msgs) {
 		 for (const msg of raw_msg.split("\n").reverse()) {
	 		 console.log(msg);
	 		 ludusResponse = msg + "<br/>" + ludusResponse;
	 	 }
 	 }
 }

 function run_code () {
	 let raw_selection = view.state.selection.toJSON();
	 let selection = {start: raw_selection.ranges[0].anchor, end: raw_selection.ranges[0].head};
	 let code = (selection.end > selection.start) ? view.state.doc.toString().slice(selection.start, selection.end) : view.state.doc.toString();
	 let ludus_response = run(code);
	 let log = ludus_response.console ?? [];
	 let {result, draw, errors = []} = ludus_response;
	 if (draw) { draw_turtle(draw); } else { ludus_init(); }
		 print_console([...errors, ...log]);
	 if (result) ludusResponse = "<i>:ludus=> </i>" + result + "<br/>" + ludusResponse;
 }

 function ludus_init () {
	 let {draw} = run(":nothing"); // get a response running nothing
	 draw_turtle(draw); // draw the base state
 }

 const sketch = (p5) => {
	 let cnv;
	 p = p5;

	 p5.setup = () => {
		 let canvas_elt = document.getElementById("canv");
		 let h = canvas_elt.clientHeight;
		 let w = canvas_elt.clientWidth;
		 cnv = p5.createCanvas(w, h);
		 cnv.parent("canv");
		 console.log(cnv);
		 p5.noLoop();
		 p5.background(0);
		 ludus_init();
	 }
 }

</script>

<main>
	<header>
		<h1>Ludus</h1>
		<a href="/" on:click|preventDefault={run_code}>RUN</a>
		<link rel="stylesheet"
  				href="https://fonts.googleapis.com/css?family=Victor+Mono">
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
	 grid-template-rows: 3em auto auto 12em;
	 grid-template-areas:
		 "header header"
		 "editor canv"
		 "editor canv"
		 "editor console";
 }

 header {
	 grid-area: header;
	 display: flex;
	 align-items: center;
	 justify-content: space-around;
	 background: #245688;
	 padding-top: 0.25em;
 }

 header h1 {
	 font-size: 1em;
	 font-family: 'Victor Mono';
	 font-style: italic;
	 font-weight: 200;
	 color: #FFFFFF;
 }

 header a {
	 display: block;
	 font-size: 1em;
	 font-family: 'Victor Mono';
	 font-style: bold;
	 font-weight: 800;
	 color: #FFFFFF;
	 background: #55555555;
	 padding-top: 2px;
	 padding-bottom: 2px;
	 padding-left: 6px;
	 padding-right: 6px;
	 border-style: solid;
	 border-color: #AAAAAA55;
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
	 font-family: 'Victor Mono', monospace;
	 font-weight: 200;
	 font-size: 12px;
 }

 :global(.p5Canvas) {
	 width: 100%;
	 height: 100%;
 }
</style>
