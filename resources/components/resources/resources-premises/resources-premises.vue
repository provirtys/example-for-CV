<template>
  <filters v-model:model-value="filters.ID" />
  <div :class="['resources__premises', { 'resources__premises--loading': loading }]">
    <resources-premise @loaded="loading = false"
      :premise="premiseToShow"
      :counters="countersToShow"
      @onEditCounter="counter => $emit('onEditCounter', counter)" />
  </div>
</template>

<script>
import { onBeforeMount, ref, watch } from 'vue'
import { useResources } from "../../../store/resources";
import ResourcesEdit from '../resources-edit/resources-edit'
import ResourcesPremise from '../resources-premise/resources-premise'
import Filters from '../../filters/filters'

export default {
  name: 'resources-premises',
  components: { ResourcesEdit, ResourcesPremise, Filters },
  emits: ['onSave', 'onEditCounter'],
  setup() {
    const storeResources = useResources()
    const { getPremises, getCounter } = storeResources

    const premises = ref([])
    const loading = ref(false)

    const premiseToShow = ref([])

    const filters = ref({ ID: 0 })

    const counters = ref([])

    const countersToShow = ref([])

    onBeforeMount(() => {
      getPremises().then(res => {
        premises.value = res.data.data

        const fillCounters = async () => {
          await Promise.all(premises.value.map(async prem => await getCounter(prem.ID).then(res => counters.value.push({ ID: prem.ID, info: res.data.data }))))
          filters.value.ID = premises.value[0].ID
          premiseToShow.value = premises.value[0]
          countersToShow.value = counters.value[0]
        }

        fillCounters()
      })
    })

    watch(() => filters, val => {
      premiseToShow.value = premises.value.find(prem => prem.ID === val.value.ID)
      countersToShow.value = counters.value.find(count => count.ID === val.value.ID)?.info ? counters.value.find(count => count.ID === val.value.ID)?.info : []
    }, { deep: true })

    return {
      premises,
      loading,
      premiseToShow,
      filters,
      countersToShow,
      counters
    }
  }
}
</script>

<style lang="scss" src="./resources-premises.scss" scoped />

