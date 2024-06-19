<script setup lang="ts">

import { computed, ref, toRefs } from 'vue'

const searchText = ref('')
const props = defineProps<{ id: string, entries: Entry[] }>()
const selectedEntryValue = defineModel<string | null>()

type Entry = {
  label: string,
  value: string,
}

const { entries } = toRefs(props)

const filteredEntries = computed(() => {
  if (searchText.value === '') {
    return entries.value
  }

  return entries.value.filter(entry =>
    entry.label.toLowerCase().includes(searchText.value.toLowerCase())
    || entry.value.toLowerCase().includes(searchText.value.toLowerCase())
    || entry.value === selectedEntryValue.value
  )
})



const totalEntries = computed(() => entries.value.length)
const shownEntries = computed(() => filteredEntries.value.length)

</script>

<template>
  <div>
  <span>
    <label :for="'select-'+id+'-input'">Search </label>
    <input :id="'select-'+id+'-input'" type="text" v-model="searchText">
  </span>
    <p><template v-if="totalEntries !== shownEntries">Showing {{ shownEntries }} of</template>
      {{ totalEntries }} Entries
    </p>

    <select v-model="selectedEntryValue">
      <option :value="null"> Select one...</option>
      <template v-for="({label, value}) in filteredEntries" :key=value>
        <option :value="value.toString()">{{ label }}</option>
      </template>
    </select>
  </div>
</template>

<style scoped>
select {
  width: 100%;
}
</style>