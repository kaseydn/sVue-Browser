<script setup>
import {
  Panel,
  PanelPosition,
  VueFlow,
  isNode,
  useVueFlow,
} from "@vue-flow/core";
import { Background } from "@vue-flow/background";
import { Controls } from "@vue-flow/controls";
import { MiniMap } from "@vue-flow/minimap";
import { ref } from "vue";
// import { initialElements } from "./components/initial-elements";
// import { initialElements } from "./components/elements";
import { tree } from './components/elements';

import Node from "./components/NodeTemplate.vue";

const initialElements = [];

const customNode = new NewNode('7', 'custom', 'G-G-Baby One', 
{x: 0, y: 600}, '', {oneway: ['v1-A', 'v1-B', 'v1-C'], twoway: ['v2-A']});
initialElements.push(customNode);

const customEdge = new NewEdge('e6-7', '6', '7');
initialElements.push(customEdge);
console.log('initialElements:',initialElements);

// create a node constructor func:
function NewNode(id, type, label, position, parentId, data) {
  this.id = id;
  this.type = type;
  this.label = label;
  this.position = position;
  this.parentId = parentId;
  this.data = data;
  this.class = "light";
}
// create an edge constructor func:
function NewEdge(id, source, target) {
  this.id = id;
  // this.type = 'smoothstep';
  this.label;
  this.source = source;
  this.target = target;
  // this.color = '#E83015'; // maybe style this instead??? Or research how to change edge color????
}

// Create instances of nodes and edges and push to 'initialElements':
function createNodesAndEdges(tree) {
  // console.log(tree);
  const q = [tree];
  let nodeId = 1;
  let x = [0]; // []
  let y = [0]; // []

  while (q.length) {
    // initialize variables and save to respective values:
    const node = q.shift();
    let type = "";
    // let type = "custom";
    let newPos = { x: x.shift(), y: y.shift() }; // {x: 250, y: 0}
    let parentId = "";

    // Add oneway and twoway props from AST to 'data' to be passed into a new instance of NewNode:
    const oneway = node.oneWayProps;
    const twoway = node.twoWayProps;
    const data = {oneway, twoway};
    // data.oneway.push('');
    // data.twoway.push('');
    const d = JSON.stringify(data);
    console.log('Data: ',d);

    const data1 = {};
    

    // change 'type' property based on if the node is a root node or leaf node:
    if (!initialElements.length) {
      type = "input";
    }
    // if (!node.children.length) {
    //   type = "output";
    // }

    // if the current node in the AST has a parentId property, assign it to 'parentId'
    if (node.parentId) {
      parentId = node.parentId;
    }
    

    // instantiate a new node and push to 'initialElements' array:
    const newNode = new NewNode(nodeId, type, node.fileName, newPos, parentId, data1);
    initialElements.push(newNode);
    nodeId += 1;
    // console.log(initialElements);

    // If the current node has children, push its children to the 'q' and create x and y postions for each child and push them to 'xQ' and 'yQ' respectively
    if (node.children.length) {
      // push all child nodes to 'q'
      q.push(...node.children);

      // intialize variable to calculate x and y positions:
      let horizontalSpace = 200; // horizontal space between sibling nodes
      let verticalSpace = 150; // vertical space between parent/child nodes
      let newXPos = -((node.children.length * horizontalSpace) / 2 - horizontalSpace / 2);
      // console.log(newXPos);

      // create new x and y positions for each child node and push to respective arrays:
      for (let i = 0; i < node.children.length; i++) {
        x.push(newNode.position.x + newXPos);
        newXPos += horizontalSpace;
        y.push(newNode.position.y + verticalSpace);

        // add a 'parentId' to all children of the current node in the AST:
        node.children[i].parentId = newNode.id;
      }
    }

    // Instantiate a new edge object and push to 'initialElements' array
    if (newNode.parentId) {
      const id = `e${newNode.parentId}-${newNode.id}`;
      const newEdge = new NewEdge(id, `${newNode.parentId}`, `${newNode.id}`);

      initialElements.push(newEdge);
    }
    // console.log("Initial Elements:", initialElements);
    // nodeQ = [rootNode, child1, child2a, child2b];
    // xQ = [250, 250, 150, 350];
    // yQ = [0, 100, 200, 200];
  }
}
 
// Need to call 'createNodesAndEdges' inside of an event listener for a post message carrying AST data from panel.ts
createNodesAndEdges(tree);

/**
 * useVueFlow provides all event handlers and store properties
 * You can pass the composable an object that has the same properties as the VueFlow component props
 */
const {
  onPaneReady,
  onNodeDragStop,
  onConnect,
  addEdges,
  setTransform,
  toObject,
} = useVueFlow();

/**
 * Our elements
 */
const elements = ref(initialElements); //  imports array of nodes and edges and stores to state (at 'elements.value')

const c = JSON.stringify(elements.data);
console.log('App.vue: elements.data:', c);

/**
 * This is a Vue Flow event-hook which can be listened to from anywhere you call the composable, instead of only on the main component
 *
 * onPaneReady is called when viewpane & nodes have visible dimensions
 */

// What does this do????
onPaneReady(({ fitView }) => {
  fitView();
});

// What does this do????
onNodeDragStop((e) => console.log("drag stop", e));

/**
 * onConnect is called when a new connection is created.
 * You can add additional properties to your new edge (like a type or label) or block the creation altogether
 */

// What is this doing exactly????
onConnect((params) => addEdges(params));

// initialize 'dark' in state and set to false:
const dark = ref(false);

/**
 * To update node properties you can simply use your elements v-model and mutate the elements directly
 * Changes should always be reflected on the graph reactively, without the need to overwrite the elements
 */

// Currently randomizes node positions, called on click of Panel button.  Change this to switch from vertical tree to horizontal tree:
function updatePos() {
  return elements.value.forEach((el) => {
    if (isNode(el)) {
      el.position = {
        // What is this doing exactly???
        x: Math.random() * 400,
        y: Math.random() * 400,
      };
    }
  });
}

/**
 * toObject transforms your current graph data to an easily persist-able object
 */
// Invoked on click of Panel button:
function logToObject() {
  return console.log(toObject());
}

/**
 * Resets the current viewpane transformation (zoom & pan)
 */
// Change this to re-render the file in Vue Flow:
function resetTransform() {
  return setTransform({ x: 0, y: 0, zoom: 1 });
}

//
function toggleClass() {
  return (dark.value = !dark.value);
}


</script>

<template>
  <VueFlow
    v-model="elements"
    :class="{ dark }"
    class="basicflow"
    :default-viewport="{ zoom: 1.5 }"
    :min-zoom="0.2"
    :max-zoom="4"
  >
      
    <template #node-custom="{ data }">
      <Node :data="customNode" />
    </template>

    <Background :pattern-color="dark ? '#FFFFFB' : '#134c84'" :variant="true" />

    <MiniMap />

    <Controls />

    <Panel :position="PanelPosition.TopRight" class="controls">
      <button
        style="background-color: #113285; color: white"
        title="Reset Transform"
        @click="resetTransform"
      >
        <svg width="16" height="16" viewBox="0 0 32 32">
          <path
            fill="#FFFFFB"
            d="M18 28A12 12 0 1 0 6 16v6.2l-3.6-3.6L1 20l6 6l6-6l-1.4-1.4L8 22.2V16a10 10 0 1 1 10 10Z"
          />
        </svg>
      </button>

      <button
        style="background-color: #6f3381"
        title="Shuffle Node Positions"
        @click="updatePos"
      >
        <svg width="16" height="16" viewBox="0 0 24 24">
          <path
            fill="#FFFFFB"
            d="M14 20v-2h2.6l-3.2-3.2l1.425-1.425L18 16.55V14h2v6Zm-8.6 0L4 18.6L16.6 6H14V4h6v6h-2V7.4Zm3.775-9.425L4 5.4L5.4 4l5.175 5.175Z"
          />
        </svg>
      </button>

      <button
        :style="{
          backgroundColor: dark ? '#FFFFFB' : '#292524',
          color: dark ? '#292524' : '#FFFFFB',
        }"
        @click="toggleClass"
      >
        <template v-if="dark">
          <svg width="16" height="16" viewBox="0 0 24 24">
            <path
              fill="#292524"
              d="M12 17q-2.075 0-3.537-1.463Q7 14.075 7 12t1.463-3.538Q9.925 7 12 7t3.538 1.462Q17 9.925 17 12q0 2.075-1.462 3.537Q14.075 17 12 17ZM2 13q-.425 0-.712-.288Q1 12.425 1 12t.288-.713Q1.575 11 2 11h2q.425 0 .713.287Q5 11.575 5 12t-.287.712Q4.425 13 4 13Zm18 0q-.425 0-.712-.288Q19 12.425 19 12t.288-.713Q19.575 11 20 11h2q.425 0 .712.287q.288.288.288.713t-.288.712Q22.425 13 22 13Zm-8-8q-.425 0-.712-.288Q11 4.425 11 4V2q0-.425.288-.713Q11.575 1 12 1t.713.287Q13 1.575 13 2v2q0 .425-.287.712Q12.425 5 12 5Zm0 18q-.425 0-.712-.288Q11 22.425 11 22v-2q0-.425.288-.712Q11.575 19 12 19t.713.288Q13 19.575 13 20v2q0 .425-.287.712Q12.425 23 12 23ZM5.65 7.05L4.575 6q-.3-.275-.288-.7q.013-.425.288-.725q.3-.3.725-.3t.7.3L7.05 5.65q.275.3.275.7q0 .4-.275.7q-.275.3-.687.287q-.413-.012-.713-.287ZM18 19.425l-1.05-1.075q-.275-.3-.275-.712q0-.413.275-.688q.275-.3.688-.287q.412.012.712.287L19.425 18q.3.275.288.7q-.013.425-.288.725q-.3.3-.725.3t-.7-.3ZM16.95 7.05q-.3-.275-.287-.688q.012-.412.287-.712L18 4.575q.275-.3.7-.288q.425.013.725.288q.3.3.3.725t-.3.7L18.35 7.05q-.3.275-.7.275q-.4 0-.7-.275ZM4.575 19.425q-.3-.3-.3-.725t.3-.7l1.075-1.05q.3-.275.713-.275q.412 0 .687.275q.3.275.288.688q-.013.412-.288.712L6 19.425q-.275.3-.7.287q-.425-.012-.725-.287Z"
            />
          </svg>
        </template>

        <template v-else>
          <svg width="16" height="16" viewBox="0 0 24 24">
            <path
              fill="#FFFFFB"
              d="M12 21q-3.75 0-6.375-2.625T3 12q0-3.75 2.625-6.375T12 3q.35 0 .688.025q.337.025.662.075q-1.025.725-1.637 1.887Q11.1 6.15 11.1 7.5q0 2.25 1.575 3.825Q14.25 12.9 16.5 12.9q1.375 0 2.525-.613q1.15-.612 1.875-1.637q.05.325.075.662Q21 11.65 21 12q0 3.75-2.625 6.375T12 21Z"
            />
          </svg>
        </template>
      </button>

      <button title="Log `toObject`" @click="logToObject">
        <svg width="16" height="16" viewBox="0 0 24 24">
          <path
            fill="#292524"
            d="M20 19V7H4v12h16m0-16a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h16m-7 14v-2h5v2h-5m-3.42-4L5.57 9H8.4l3.3 3.3c.39.39.39 1.03 0 1.42L8.42 17H5.59l3.99-4Z"
          />
        </svg>
      </button>
    </Panel>
  </VueFlow>
</template>

<style>
@import "https://cdn.jsdelivr.net/npm/@vue-flow/core@1.20.2/dist/style.css";
@import "https://cdn.jsdelivr.net/npm/@vue-flow/core@1.20.2/dist/theme-default.css";
@import "https://cdn.jsdelivr.net/npm/@vue-flow/controls@latest/dist/style.css";
@import "https://cdn.jsdelivr.net/npm/@vue-flow/minimap@latest/dist/style.css";
@import "https://cdn.jsdelivr.net/npm/@vue-flow/node-resizer@latest/dist/style.css";

html,
body,
#app {
  margin: 0;
  height: 100%;
  background: #272727;
}



#app {
  text-transform: uppercase;
  font-family: "JetBrains Mono", monospace;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: rgba(255, 255, 255, 0.2);
}

.vue-flow__minimap {
  transform: scale(75%);
  transform-origin: bottom right;
}

.vue-flow__node-custom {
  width: 9.5rem;
  border-radius: .2rem;
}

.basicflow .vue-flow__node {
  background: #c1c1c1;
  color: #000000;
  border: 1px solid rgb(191, 99, 252);
  /* box-shadow: 0 .15rem .2rem .4rem rgba(252, 99, 232, 0.2) */
  font-weight: strong;
}

.basicflow.dark {
  background: #272727;
  color: rgba(0, 0, 0, 0.2);
}

.basicflow.dark .vue-flow__node {
  background: #494949;
  color: #fffffb;
  border: 2px solid rgb(149, 0, 255);
}
.basicflow.dark .vue-flow__controls .vue-flow__controls-button {
  background: #292524;
  fill: #fffffb;
  border-color: #fffffb;
}
.basicflow.dark .vue-flow__edge-textbg {
  fill: #292524;
}
.basicflow.dark .vue-flow__edge-text {
  fill: #fffffb;
}
.basicflow .controls {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 8px;
}
.basicflow .controls button {
  padding: 4px;
  border-radius: 5px;
  font-weight: 600;
  -webkit-box-shadow: 0px 5px 10px 0px rgba(0, 0, 0, 0.3);
  box-shadow: 0 5px 10px #0000004d;
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;
}
.basicflow .controls button:hover {
  transform: scale(102%);
  transition: 0.25s all ease;
}
</style>
