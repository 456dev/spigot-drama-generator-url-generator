<script setup lang="ts">
import { useFetch } from '@vueuse/core'
import { computed, type Ref, ref, watchEffect } from 'vue'
import FilteredSelector from '@/components/FilteredSelector.vue'

const dataUrl = 'https://raw.githubusercontent.com/mdcfe/spigot-drama-generator/master/src/data.json'

const { isFetching, error, data } = await useFetch(dataUrl)

const replacementRegex = new RegExp(/\[([A-z]+)]/g)


type SourceData = {
  combinations: Record<string, string[]>
  sentences: string[]
  social: string[] // unused, but part of data json
}


type ResultData = {
  [key: string]: number | number[] | undefined // only base number if key == "sentence"
}

const stringData = computed(() => typeof data.value === 'string' ? data.value : '')
const typedData = computed(() => JSON.parse(stringData.value) as SourceData)

function entrify(data: string[]) {
  return data.map((value, index) => ({
    label: value,
    value: index.toString()
  }))
}

const selectedSentenceEntry = ref<string | null>(null)

watchEffect(() => {
  if (selectedSentenceEntry.value !== null) {
    selectedVariables.forEach(variable => variable.value = null)
  }
})

const selectedSentence = computed(() => {
  if (selectedSentenceEntry.value === null) {
    return ''
  }
  return typedData.value.sentences[parseInt(selectedSentenceEntry.value)]
})

const variables = computed(() => selectedSentence.value.matchAll(replacementRegex))
const variableArray = computed(() => Array.from(variables.value).map(match => match[1]))
const selectedVariables: Ref<string | null>[] = []
watchEffect(() => {
  selectedVariables.length = 0
  for (const variable of variableArray.value) {
    selectedVariables.push(ref<string | null>(null))
  }
})

const selectedVariableIndexList = computed(() => selectedVariables.map(variable => variable.value === null ? null : parseInt(variable.value)))
const canShowResult = computed(() => selectedSentenceEntry.value !== null && selectedVariableIndexList.value.every(index => index !== null))
const canShowPartialResult = computed(() => selectedSentenceEntry.value !== null && selectedVariableIndexList.value.some(index => index !== null))
const selectedVariableValueList = computed(() => selectedVariableIndexList.value.map((index, i) => index === null ? '' : typedData.value.combinations[variableArray.value[i]][index]) ?? '')

const resultValue = computed(() => {
  let message = selectedSentence.value
  for (const [index, variable] of variableArray.value.entries()) {
    message = message.replace(`[${variable}]`, selectedVariableValueList.value[index] === '' ? `%${variable}%`:selectedVariableValueList.value[index])
  }
  return message
})

const resultData = computed(() => {
  if (!canShowResult.value) {
    return null
  }
  const result: ResultData = {
    sentence: parseInt(selectedSentenceEntry.value ?? '')
  }
  for (const [index, variable] of variableArray.value.entries()) {
    if ((result[variable] as number[] | undefined)?.push(selectedVariableIndexList.value[index] ?? -1) === undefined) {
      result[variable] = [selectedVariableIndexList.value[index] ?? -1]
    }
  }
  return result
})

const resultJson = computed(() => resultData.value !== null ? JSON.stringify(resultData.value): "")
const resultBase64 = computed(() => resultJson.value !== '' ? btoa(resultJson.value) : "")
// watchEffect(() => { // TODO: re-add once current state is parsed from hash
//   window.location.hash = resultBase64.value
// })

const dramaUrl = computed(() => `https://drama.mdcfe.dev/${resultBase64.value}`)

</script>

<template>
  <div>
    <p v-if="isFetching">Fetching data...</p>
    <p v-if="error">Error: {{ error }} </p>
    <div v-if="data">
      <!--      <p>{{typedData}}</p>-->
      <h1><a href="https://drama.mdcfe.dev" rel="noopener" target="_blank">Spigot Drama</a> Builder</h1>
      <h2>Select a Template</h2>

      <FilteredSelector id="sentence" :entries="entrify(typedData.sentences)" v-model="selectedSentenceEntry" />
      <p v-if="selectedSentence">Full Template:<br/>{{selectedSentence}}</p>
      <div class="varSelector">
        <div class="varSelectorItem" v-for="(variable, index) in variableArray" v-bind:key="index">
          <h2>Var {{ index + 1 }}: {{ variable }}</h2>
          <FilteredSelector :id="'var-'+index" :entries="entrify(typedData.combinations[variable])"
                            v-model="selectedVariables[index].value" />
        </div>
      </div>
      <div class="resultContainer" v-if="canShowResult || canShowPartialResult">
        <h1>Result</h1>
        <p class="result">{{ resultValue }}</p>
        <a v-if="canShowResult" :href="dramaUrl" rel="noopener" target="_blank">Permalink: <code>{{ dramaUrl }}</code></a>
      </div>
    </div>

    <p class="credits">Made by <a href="https://github.com/456dev/spigot-drama-generator-url-generator" rel="noopener" target="_blank">456dev</a>, with credit to the original <a href="https://drama.mdcfe.dev" rel="noopener" target="_blank">site</a> and data by <a href="https://github.com/mdcfe/" rel="noopener" target="_blank">mdcfe</a> and contributors</p>

  </div>
</template>

<style scoped>
div {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.varSelector {
  display: flex;
  flex-wrap: wrap;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.varSelectorItem {
  padding: 2em
}

.result {
  font-size: x-large;
}

.resultContainer {
  margin-bottom: 2em;
}

.resultContainer a {
  font-size: large;
}

.credits {
  font-size: small;
  margin: 2em;
}

@media (min-width: 600px) {
  .varSelector {
    flex-direction: row;
  }
}
</style>