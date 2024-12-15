<script setup lang="ts">
import { GraphMaker, GraphMakerProps } from "@milaboratories/graph-maker";
import '@milaboratories/graph-maker/styles';
import { computed } from "vue";
import { useApp } from "../app";

const app = useApp();

const defaultOptions = computed((): GraphMakerProps['defaultOptions'] => {
    const overlapSpec = app.model.outputs.overlapSpec
    console.log("Privet");

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
</script>

<template>
    <GraphMaker chart-type="heatmap" :p-frame="app.model.outputs.pf" v-model="app.model.ui.graphState"
        :default-options="defaultOptions" />
</template>