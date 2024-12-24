<script setup lang="ts">
import { GraphMaker, GraphMakerProps } from "@milaboratories/graph-maker";
import '@milaboratories/graph-maker/styles';
import { computed, watch } from 'vue';
import { useApp } from "../app";
import { FindColumnsRequest } from '@platforma-sdk/model';

const app = useApp();

const defaultOptions = computed((): GraphMakerProps['defaultOptions'] => {
    const overlapSpec = app.model.outputs.overlapSpec
    
    if (!overlapSpec) {
        return undefined
    }

    const defaults: GraphMakerProps['defaultOptions'] = [
        {
            inputName: 'value',
            selectedSource: overlapSpec
        },
        {
            inputName: 'x',
            selectedSource: overlapSpec.axesSpec[2]
        },
        {
            inputName: 'y',
            selectedSource: overlapSpec.axesSpec[3]
        },
        {
            inputName: 'facetBy',
            selectedSource: overlapSpec.axesSpec[1]
        },
        {
            inputName: 'tabBy',
            selectedSource: overlapSpec.axesSpec[0]
        }
    ];

    return defaults;
})

watch(() => app.model.outputs.pf, async (handle) => {
  const request1 = {
    columnFilter: {
      name: ['pl7.app/vdj/overlap'],
      type: ['Float']
    },
    compatibleWith: [],
    strictlyCompatible: false
  } as FindColumnsRequest;
  const response1 = await platforma?.pFrameDriver.findColumns(handle!, request1);
  const overlapColumn = response1?.hits[0];

  console.log(overlapColumn?.spec)

  const request = {
    "columnFilter": {
      "type": ["String"],
      "annotationValue": {}
    },
    "compatibleWith": overlapColumn?.spec.axesSpec.map(spec => ({
      type: spec.type,
      name: spec.name,
      domain: spec.domain
    })),
    "strictlyCompatible": true
  } as FindColumnsRequest;
  const columns = await platforma?.pFrameDriver.findColumns(handle!, request);
  console.log('columns:', columns);

}, {immediate: true})
</script>

<template>
    <GraphMaker chart-type="heatmap" :p-frame="app.model.outputs.pf" v-model="app.model.ui.graphState"
        :default-options="defaultOptions" />
</template>