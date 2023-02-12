<template>
  <v-modal
    ref="modalRef"
    class="resources-modal"
    variant="long"
  >
    <template #header>
      <h1 class="resources-modal__title">{{ info.title }}</h1>
    </template>
    <template #content>
      <div :class="['resources-modal__content', { 'resources-modal__content--loading': loading }]">
        <div class="resources-modal__number">№{{ counter.serialNumber }}</div>
        <v-input
          v-model="counter.label"
          :placeholder="info.placeholder"
        >Имя счетчика</v-input>
        <v-button @click="onSave">Сохранить</v-button>
      </div>
    </template>
  </v-modal>
</template>

<script>
import { computed, ref } from 'vue'
import VInput from '../../ui/v-input/v-input'
import VButton from '../../ui/v-button/v-button'
import VModal from '../../ui/v-modal/v-modal'

export default {
  name: 'resources-edit',
  components: { VInput, VModal, VButton },
  emit: ['onSave'],
  props: {
    loading: {
      type: Boolean,
      required: false,
      default: false,
    },
    counter: {
      type: Object,
      required: true,
    },
  },
  setup(props, { emit }) {
    const modalRef = ref()

    const info = computed(() => {
      switch (props.counter.name) {
        case 'Электроэнергия':
          return { title: 'Счетчик электроэнергии', placeholder: 'Электроэнергия' }
        case 'Холодная':
          return { title: 'Счетчик холодного водоснабжения', placeholder: 'Холодная вода' }

        case 'Горячая':
          return { title: 'Счетчик горячего водоснабжения', placeholder: 'Горячая вода' }

        case 'Отопление':
          return { title: 'Счетчик отопления', placeholder: 'Отопление' }
      }
    })

    const onCloseModal = () => {
      modalRef.value.closeModal()
    }

    const onSave = () => {
      emit('onSave')
    }

    return {
      info,
      modalRef,
      onSave,
      onCloseModal
    }
  }
}
</script>

<style lang="scss" src="./resources-edit.scss" scoped />

