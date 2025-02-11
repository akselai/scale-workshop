<script setup lang="ts">
import Modal from '@/components/ModalDialog.vue'
import ScaleLineInput from '@/components/ScaleLineInput.vue'
import { type Scale } from 'scale-workshop-core'
import { FIFTH, FIFTH_12TET } from '@/constants'
import { useModalStore } from '@/stores/modal'

const props = defineProps<{
  scale: Scale
  centsFractionDigits: number
  decimalFractionDigits: number
}>()

const emit = defineEmits(['update:scale', 'cancel'])

const modal = useModalStore()

function modify() {
  emit(
    'update:scale',
    props.scale.stretch(modal.stretchAmount).mergeOptions({
      centsFractionDigits: props.centsFractionDigits,
      decimalFractionDigits: props.decimalFractionDigits
    })
  )
}
</script>

<template>
  <Modal @confirm="modify" @cancel="$emit('cancel')">
    <template #header>
      <h2>Stretch/compress scale</h2>
    </template>
    <template #body>
      <div class="control-group">
        <p>Stretch or compress the whole scale evenly.</p>
        <p>Entering 1 will cause no change; entering 2 will make every interval twice as large.</p>
        <div class="control">
          <label for="amount">Stretch ratio</label>
          <input
            class="real-valued"
            type="number"
            id="amount"
            min="0.001"
            max="999.999"
            step="0.001"
            v-model="modal.stretchAmount"
          />
        </div>
        <hr />
        <div class="control">
          <label for="reference">Reference interval</label>
          <ScaleLineInput
            id="reference"
            placeholder="7\12"
            :defaultValue="FIFTH_12TET"
            @update:value="modal.reference = $event"
            v-model="modal.referenceString"
          />
        </div>
        <div class="control">
          <label for="reference">Target interval</label>
          <ScaleLineInput
            id="reference"
            placeholder="3/2"
            :defaultValue="FIFTH"
            @update:value="modal.target = $event"
            v-model="modal.targetString"
          />
        </div>
        <div class="control">
          <label for="stretch-into">Stretch reference into target</label>
          <button @click="modal.calculateStretchAmount">Calculate</button>
        </div>
      </div>
    </template>
  </Modal>
</template>

<style scoped>
.real-valued:invalid {
  background-color: var(--color-background);
}
</style>
