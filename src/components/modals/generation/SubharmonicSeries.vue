<script setup lang="ts">
import { DEFAULT_NUMBER_OF_COMPONENTS } from '@/constants'
import Modal from '@/components/ModalDialog.vue'
import { clamp } from 'xen-dev-utils'
import { Scale } from 'scale-workshop-core'
import { useModalStore } from '@/stores/modal'

const emit = defineEmits(['update:scale', 'update:scaleName', 'cancel'])

const modal = useModalStore()

function generate() {
  const leastDenominator = Math.max(1, Math.round(modal.lowInteger))
  const numerator = clamp(
    leastDenominator + 1,
    leastDenominator + 1000,
    Math.round(modal.highInteger)
  )
  const scale = Scale.fromSubharmonicSeries(
    numerator,
    leastDenominator,
    DEFAULT_NUMBER_OF_COMPONENTS
  )
  emit('update:scaleName', `Subharmonics ${leastDenominator}-${numerator}`)
  emit('update:scale', scale)
}
</script>

<template>
  <Modal @confirm="generate" @cancel="$emit('cancel')">
    <template #header>
      <h2>Generate subharmonic series segment</h2>
    </template>
    <template #body>
      <div class="control-group">
        <div class="control">
          <label for="lowest-subharmonic">Lowest subharmonic</label>
          <input
            id="lowest-subharmonic"
            type="number"
            min="1"
            class="control"
            v-model="modal.lowInteger"
          />
        </div>
        <div class="control">
          <label for="highest-subharmonic">Highest subharmonic</label>
          <input
            id="highest-subharmonic"
            type="number"
            min="1"
            class="control"
            v-model="modal.highInteger"
          />
        </div>
      </div>
    </template>
  </Modal>
</template>
