<script setup lang="ts">
import { GraphMaker, GraphMakerProps } from "@milaboratories/graph-maker";
import '@milaboratories/graph-maker/styles';
import { computed, watch } from 'vue';
import { useApp } from "../app";
import { CalculateTableDataRequest, FindColumnsRequest, PObjectId } from '@platforma-sdk/model';

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
  const overlapId = response1?.hits[0].columnId;

  const request2 = {
    columnFilter: {
      name: ['pl7.app/label'],
      type: ['String'],
      annotationValue: {"pl7.app/isLabel": "true", "pl7.app/label": "Sample"}
    },
    compatibleWith: [],
    strictlyCompatible: false
  } as FindColumnsRequest;
  const response2 = await platforma?.pFrameDriver.findColumns(handle!, request2);
  const labelsId = response2?.hits[0].columnId;

  console.log(overlapId, labelsId)

  const request = {
    "src": {
      "type": "outer",
      "primary": {
        "type": "inner",
        "entries": [
          {
            "type": "column",
            "column": overlapId
          }
        ]
      },
      "secondary": [
        {
          "type": "column",
          "column": labelsId
        }
      ]
    },
    "filters": [
      {
        "type": "bySingleColumnV2",
        "column": {
          "type": "axis",
          "id": {
            "type": "String",
            "name": "pl7.app/vdj/overlapMetric",
            "domain": {}
          }
        },
        "predicate": {
          "operator": "Equal",
          "reference": "F1Index"
        }
      },
      {
        "type": "bySingleColumnV2",
        "column": {
          "type": "axis",
          "id": {
            "type": "String",
            "name": "pl7.app/vdj/chain",
            "domain": {}
          }
        },
        "predicate": {
          "operator": "Equal",
          "reference": "IGH"
        }
      }
    ],
    "sorting": []
  } as unknown as CalculateTableDataRequest<PObjectId>;
  const table = await platforma?.pFrameDriver.calculateTableData(handle!, request);
  console.log('table:', table);

  const labelsColumnRequest = {
    src: {
      type: 'column',
      column: labelsId,
    },
    filters: [],
    sorting: [],
  } as unknown as CalculateTableDataRequest<PObjectId>;
  const labelsColumn = await platforma?.pFrameDriver.calculateTableData(handle!, labelsColumnRequest)
  console.log('labels:', labelsColumn);
}, {immediate: true})
</script>

<template>
    <GraphMaker chart-type="heatmap" :p-frame="app.model.outputs.pf" v-model="app.model.ui.graphState"
        :default-options="defaultOptions" />
</template>