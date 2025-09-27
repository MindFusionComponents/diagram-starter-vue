<style src="@mindfusion/common-ui/themes/common-ui.css"></style>
<style src="@mindfusion/common-ui/themes/business.css"></style>
<template>
  <div class="App">
    <div class="container">
      <div class="main">
        <div class="sidebar-left">
          <overview :diagramView="diagramViewRef" :style="{ width: '200px', height: '200px' }" />
          <palette id="palette" :padding="2" :theme="'business'" :height="'calc(100vh - 270px)'">
            <palette-category :items="paletteItems" title="Flowchart Shapes" :expanded="true"></palette-category>
            <palette-category :items="paletteItems1" title="Data Shapes"></palette-category>
            <palette-category :items="paletteItems2" title="BPMN Shapes"></palette-category>
          </palette>
        </div>
        <div class="main">
          <zoom-control
            :diagramView="diagramViewRef"
            :style="{
              position: 'absolute',
              right: '20px',
              top: '30px',
              zIndex: 2,
            }"
          />
          <ruler :style="{ width: '100%' }">
            <diagram-view
              ref="diagramViewRef"
              id="diagram1"
              :diagram="diagram"
              linkBackId = "mindfusionLink"
              v-on:link-creating="onLinkCreating"
              v-on:node-created="onNodeCreated"
              :style="{
                position: 'absolute',
                height: 'auto',
                width: 'auto',
                left: '0px',
                right: '0px',
                top: '0px',
                bottom: '0px',
              }"
            />
          </ruler>
        </div>
      </div>
      <div class="sidebar">
      <button v-on:click="onNewClick">New</button>
      <button v-on:click="onSaveClick">Save</button>
      <button v-on:click="onLoadClick">Load</button>
      </div>
    </div>
    <div class="footer">
    <a id="mindfusionLink" href="https://mindfusion.dev/javascript-diagram.html">h/t MindFusion</a>
  </div>
  </div>
</template>

<script lang="ts">
import { ref } from 'vue'
import * as Drawing from '@mindfusion/drawing'
import * as Diagramming from '@mindfusion/diagramming'
import {
  DiagramView,
  Palette,
  Overview,
  ZoomControl,
  Ruler,
  PaletteCategory,
  PaletteItem
} from '@mindfusion/diagramming-vue'

export const diagramViewRef = ref(null)
export default {
  name: 'ControlsSample',
  setup() {
    // create the diagram
    const diagram = new Diagramming.Diagram()

    const theme = new Diagramming.Theme()
    const shapeNodeStyle = new Diagramming.Style()
    shapeNodeStyle.brush = { type: 'SolidBrush', color: '#e0e9e9' }
    shapeNodeStyle.stroke = '#7F7F7F'
    shapeNodeStyle.fontName = 'Verdana'
    shapeNodeStyle.fontSize = 4
    shapeNodeStyle.nodeEffects = [new Diagramming.GlassEffect()]
    theme.styles.set('std:ShapeNode', shapeNodeStyle)
    diagram.theme = theme

    diagram.bounds = new Drawing.Rect(0, 0, 1000, 1000);

    // you can create diagram items from code;
    // alternative syntax is diagram.addItem(new ShapeNode());
    var node1 = diagram.factory.createShapeNode(10, 10, 30, 30);
    node1.text = "Hello";

    var node2 = diagram.factory.createShapeNode(60, 25, 30, 30);
    node2.text = "World";

    diagram.factory.createDiagramLink(node1, node2);

	// automatically route links drawn by user
	diagram.routeLinks = true;

    // stock shape geometries are listed here:
    // https://www.mindfusion.eu/onlinehelp/jsdiagram/CC_refTable_of_Predefined_Shapes_4.htm

    // use the shape designer tool to draw custom shape geometries:
    // https://mindfusion.eu/tools/shape-designer.html

    // apart from ShapeNode, you could also add TableNode or ContainerNode objects
    let paletteItems = [];
    var shapes = ["Start", "Input", "Process", "Decision"]
    for (let i = 0; i < shapes.length; ++i) {
      let node = new Diagramming.ShapeNode();
      node.shape = shapes[i];
      node.style = shapeNodeStyle;
      paletteItems.push(new PaletteItem(node, shapes[i]));
    }

    let paletteItems1 = [];
    shapes = ["Database", "Input", "Delay", "Document", "ManualOperation"];
    for (let i = 0; i < shapes.length; ++i) {
      let node = new Diagramming.ShapeNode();
      node.shape = shapes[i];
      node.style = shapeNodeStyle;
      paletteItems1.push(new PaletteItem(node, shapes[i]));
    }


    let paletteItems2 = [];
    shapes = ["BpmnStartLink", "BpmnIntermediateLink", "BpmnEndLink",
      "BpmnStartMessage", "BpmnIntermediateMessage", "BpmnEndMessage"];
    for (let i = 0; i < shapes.length; ++i) {
      let node = new Diagramming.ShapeNode();
      node.shape = shapes[i];
      node.style = shapeNodeStyle;
      paletteItems2.push(new PaletteItem(node, shapes[i]));
    }

    return { diagram, diagramViewRef, paletteItems, paletteItems1, paletteItems2 }
  },

  methods: {


  // detect user's actions by handling diagram events, such as nodeCreated
  onNodeCreated(sender : Diagramming.Diagram, args: Diagramming.NodeEventArgs) {
    console.log("user has created a node");
    args.node.brush = "lightblue";
  },

  // validation events let us prevent users' actions; for example,
  // onLinkCreating handler below prevents users from drawing a cycle
  onLinkCreating(sender : Diagramming.Diagram, args: Diagramming.LinkEventArgs) {
    if (args.destination == null) {
      // not pointing to a node yet
      return;
    }

    var pathFinder = new Diagramming.PathFinder(this.diagram);
    var path = pathFinder.findShortestPath(
      args.destination, args.origin);

    if (path != null) {
      // adding this new link would create a cycle
      // [origin]--[dest]--[path internal nodes]--[origin]

      args.cancel = true;
    }
  },

  onNewClick() {
    this.diagram.clearAll();
  },

  insecureContextMessage() { return 'The File System API is not available in this context. Please run the page from a web server (npm start).'; },

  async onSaveClick() {
    try {
      // in this example we store diagram JSON files on local file system;
      // alternatively you could send JSON to server-side using fetch API, e.g.
      // fetch('api_url', { method: 'POST', ... }
      const json = this.diagram.toJson();

      // file system API is not fully supported in some browsers yet
      if (window.showSaveFilePicker) {
        if (!window.isSecureContext) {
          alert(this.insecureContextMessage());
          return;
        }

        const handle = await window.showSaveFilePicker(
          {
            startIn: 'documents',
            suggestedName: 'diagram.json',
            types: [{
              description: 'JSON Files',
              accept: {
                'application/json': ['.json'],
              },
            }],
          });
        const writable = await handle.createWritable();
        await writable.write(json);
        await writable.close();
      }
      else {
        // work-around for browsers that do not support file system
        const blob = new Blob([json], { type: 'application/json' });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = 'diagram.json';
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
        URL.revokeObjectURL(url);
      }
    }
    catch (err) {
      if (typeof err === "string") {
        console.error(err);
      } else if (err instanceof Error) {
        console.error(err.name, err.message);
      }
    }
  },

  async onLoadClick() {
    try {
      // in this example we store diagram JSON files on local file system;
      // alternatively you could load JSON from server-side using fetch API, e.g.
      // fetch('api_url', { method: 'GET', ... }

      // file system API is not fully supported in some browsers yet
      if (window.showOpenFilePicker) {
        if (!window.isSecureContext) {
          alert(this.insecureContextMessage());
          return;
        }

        const [handle] = await window.showOpenFilePicker(
          {
            startIn: 'documents',
            types: [{
              description: 'JSON Files',
              accept: {
                'application/json': ['.json'],
              },
            }],
          });
        const file = await handle.getFile();
        const content = await file.text();
        this.diagram.fromJson(content);
      }
      else {
        // work-around for browsers that do not support file system
        const input = document.createElement('input');
        input.type = 'file';
        input.accept = '.json,application/json';
        input.onchange = async (e) => {
          if (e.target != null) {
            var t = (<HTMLInputElement>e.target);
            const file = t.files![0];
            const content = await file.text();
            this.diagram.fromJson(content);
          }
        };
        input.click();
      }
    }
    catch (err) {
      if (typeof err === "string") {
        console.error(err);
      } else if (err instanceof Error) {
        console.error(err.name, err.message);
      }
    }
  }


  },
  data() {
    return {
      diagramView: DiagramView,
    }
  },
  components: {
    DiagramView,
    Overview,
    ZoomControl,
    Ruler,
    Palette,
    PaletteCategory
  },
}
</script>

<style>
body {
  margin: 0;
  padding: 0;
  font-family: sans-serif;
}
.container {
  position: absolute;
  display: flex;
  top: 0;
  left: 0;
  bottom: 60px;
  right: 0;
  text-align: left;
}
.main {
  flex: 1 1 100%;
  overflow: hidden;
  display: flex;
  position: relative;
}
.sidebar {
  flex: 0 0 200px;
  order: 1;
  padding-left: 15px;
  overflow-y: auto;
}
.sidebar-left{
  flex: 0 0 220px;
  overflow-y: auto;
}
.footer {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 40px;
  background-color: #fafafa;
  border-top: 1px solid #e2e4e7;
  padding: 5px;
}
.sidebar button {
	display: block;
	width: 80%;
	margin: 10px auto;
	padding: 10px;
	font-size: 16px;
}

</style>

