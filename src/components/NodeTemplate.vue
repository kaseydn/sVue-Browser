<script setup>
  import { Handle, Position } from '@vue-flow/core'
  // import { customNode } from './elements'

  const props = defineProps({
    data: {
      type: Object,
    },
  })

  let oneway, twoway;
  // if (customNode.data.oneway) {oneway = customNode.data.oneway;}
  // if (customNode.data.twoway) {twoway = customNode.data.twoway;}

  if (props.data.data.oneway) {oneway = props.data.data.oneway;}
  if (props.data.data.twoway) {twoway = props.data.data.twoway;}

  let label = props.data.label;
  // let label = customNode.label;

  // const c = JSON.stringify(customNode);
  // console.log('NodeTemplate: Custom Node:', c);


</script>

<template id="node">
  <div style="padding-top: .5rem;">{{ label }}</div>

  <div
    style="display: flex; flex-direction: row; align-self: center; justify-content: center; max-width: 90%; margin: auto; gap: 3px"
  >

  <template v-for="vars of oneway" >
      <div class="oneway">
        <p type="text">{{ vars }}</p>
      </div>
    </template>

    <template v-for="vars of twoway" >
      <div class="twoway">
        <p type="text">{{ vars }}</p>
      </div>
    </template>
  </div>

  <Handle id="a" type="target" :position="Position.Top" />

  <!-- <Handle id="b" type="source" :position="Position.Bottom" /> -->
</template>

<style>
#node {
  width: 12rem;
}
.oneway {
  font-size: .7rem;
  color: rgb(152, 0, 0);
}
.twoway {
  font-size: .7rem;
  color: rgb(3, 0, 152);
}
</style>