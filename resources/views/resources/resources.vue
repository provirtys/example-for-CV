<template>
  <div class="resources">
    <resources-premises @onEditCounter="counter => onEditCounter(counter)" />
    <resources-edit
      v-if="isOpen"
      :counter="counterToEdit"
      ref="editRef"
      :loading="editting"
      @onSave="onSave"
      @closeModal="closeModal"
    >
    </resources-edit>
  </div>
</template>

<script>
import { ref } from "vue";
import { useResources } from "../../store/resources";
import useModal from "../../hooks/useModal";
import ResourcesPremises from "../../components/resources/resources-premises/resources-premises.vue";
import ResourcesEdit from "../../components/resources/resources-edit/resources-edit.vue";
import { useToast } from "vue-toastification";


export default {
  name: "resources",
  components: { ResourcesPremises, ResourcesEdit },
  setup() {
    const storeResources = useResources()
    const { editCounterTitle } = storeResources
    const { isOpen, openModal, closeModal } = useModal()
    const toast = useToast()


    const counterToEdit = ref('')
    const editRef = ref()
    const editting = ref(false)

    const onEditCounter = (counter) => {
      counterToEdit.value = counter
      openModal()
    }

    const onSave = () => {
      if(counterToEdit.value.label.length === 0){
        toast.warning('Введите имя счетчика')
      }
      else{
        editting.value = true
        editCounterTitle(counterToEdit.value)
          .then(() => {
            editting.value = false
            editRef.value.onCloseModal()
            toast.success('Имя счетчика изменено')
          })
          .catch(() => {
            editting.value = false
          })
      }
    }

    return {
      isOpen,
      counterToEdit,
      editRef,
      editting,
      onEditCounter,
      closeModal,
      onSave,
    }
  }
}
</script>

<style lang="scss" src="./resources.scss" scoped/>
