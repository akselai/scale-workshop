Interval
<script setup lang="ts">
import { OCTAVE } from '@/constants'
import {
  daughterMos,
  mos,
  mosWithDaughter,
  mosWithParent,
  parentMos,
  type MosScaleInfo
} from 'moment-of-symmetry'
import Modal from '@/components/ModalDialog.vue'
import ScaleLineInput from '@/components/ScaleLineInput.vue'
import { Scale } from 'scale-workshop-core'
import { useModalStore } from '@/stores/modal'

const COLORS = {
  parent: 'white',
  sharp: 'navy',
  flat: 'maroon',
  both: 'black',
  unknown: 'indigo'
}

const emit = defineEmits(['update:scale', 'update:scaleName', 'update:keyColors', 'cancel'])

const modal = useModalStore()

function generate() {
  let name: string
  let steps: number[]
  if (modal.colorMethod === 'none') {
    steps = mos(modal.safeNumLarge, modal.safeNumSmall, {
      sizeOfLargeStep: modal.safeSizeLarge,
      sizeOfSmallStep: modal.safeSizeSmall,
      up: modal.safeUp
    })
  } else if (modal.colorMethod === 'parent') {
    const map = mosWithParent(modal.safeNumLarge, modal.safeNumSmall, {
      sizeOfLargeStep: modal.safeSizeLarge,
      sizeOfSmallStep: modal.safeSizeSmall,
      up: modal.safeUp,
      accidentals: modal.parentColorAccidentals
    })
    steps = [...map.keys()]
    steps.sort((a, b) => a - b)
    const colors = steps.map((step) => (map.get(step) ? 'white' : 'black'))
    colors.unshift(colors.pop()!)
    emit('update:keyColors', colors)
  } else {
    let accidentals = modal.daughterColorAccidentals
    if (accidentals === 'all') {
      accidentals = 'both'
    }
    const map = mosWithDaughter(modal.safeNumLarge, modal.safeNumSmall, {
      sizeOfLargeStep: modal.safeSizeLarge,
      sizeOfSmallStep: modal.safeSizeSmall,
      up: modal.safeUp,
      accidentals
    })
    if (modal.daughterColorAccidentals === 'all') {
      steps = [...Array(Math.max(...map.keys())).keys()]
      steps.push(steps.length)
      steps.shift()
    } else {
      steps = [...map.keys()]
      steps.sort((a, b) => a - b)
    }
    let colors: string[]
    if (modal.daughterColorAccidentals === 'sharp' || modal.daughterColorAccidentals === 'flat') {
      // Piano-style layers expect black & white
      colors = steps.map((step) => (map.get(step) === 'parent' ? 'white' : 'black'))
    } else {
      // Multi-layer piano not implemented in v2 series
      colors = steps.map((s) => COLORS[map.get(s) ?? 'unknown'])
    }
    colors.unshift(colors.pop()!)
    emit('update:keyColors', colors)
  }

  if (modal.colorMethod === 'parent') {
    const parent = parentMos(modal.safeNumLarge, modal.safeNumSmall)
    name = `MOS ${modal.safeNumLarge}L ${modal.safeNumSmall}s (${parent.mosPattern} on white)`
  } else if (modal.colorMethod === 'daughter') {
    const daughter = daughterMos(
      modal.safeNumLarge,
      modal.safeNumSmall,
      modal.safeSizeLarge,
      modal.safeSizeSmall
    )
    name = 'MOS '
    if (daughter.hardness === 'equalized' || modal.daughterColorAccidentals === 'all') {
      const equaveSteps = steps[steps.length - 1]
      name += `${equaveSteps}ED`
      if (modal.equave.equals(OCTAVE)) {
        name += 'O'
      } else {
        name += modal.equave.toString()
      }
    } else {
      name += daughter.mosPattern
    }
    name += ` (${modal.safeNumLarge}L ${modal.safeNumSmall}s on white)`
  } else {
    name = `MOS ${modal.safeNumLarge}L ${modal.safeNumSmall}s`
  }
  emit('update:scaleName', name)

  const scale = Scale.fromEqualTemperamentSubset(steps, modal.equave)
  emit('update:scale', scale)
}

// Actions that would take multiple lines in template code and get ruined by auto-formatting
function pyramidMouseEnter(l: number, key: number) {
  modal.previewL = l
  modal.previewS = key + 1 - l
}

function pyramidClick(l: number, key: number) {
  modal.numberOfLargeSteps = l
  modal.numberOfSmallSteps = key + 1 - l
  modal.method = 'direct'
}

function edoMouseEnter(info: MosScaleInfo) {
  modal.previewL = info.numberOfLargeSteps
  modal.previewS = info.numberOfSmallSteps
}

function edoClick(info: MosScaleInfo) {
  modal.numberOfLargeSteps = info.numberOfLargeSteps
  modal.numberOfSmallSteps = info.numberOfSmallSteps
  modal.sizeOfLargeStep = info.sizeOfLargeStep
  modal.sizeOfSmallStep = info.sizeOfSmallStep
  modal.method = 'direct'
}
</script>

<template>
  <Modal extraStyle="width: 40rem" @confirm="generate" @cancel="$emit('cancel')">
    <template #header>
      <h2>Generate MOS scale</h2>
    </template>
    <template #body>
      <div class="control-group">
        <div class="control radio-group">
          <span>
            <input type="radio" id="method-direct" value="direct" v-model="modal.method" />
            <label for="method-direct"> Direct </label>
          </span>

          <span>
            <input type="radio" id="method-pyramid" value="pyramid" v-model="modal.method" />
            <label for="method-pyramid"> Pyramid </label>
          </span>

          <span>
            <input type="radio" id="method-edo" value="edo" v-model="modal.method" />
            <label for="method-edo"> EDO </label>
          </span>
        </div>
      </div>

      <div class="control-group" v-show="modal.method === 'direct'">
        <div class="control">
          <label for="number-of-large-steps">Number of large steps</label>
          <input
            id="number-of-large-steps"
            type="number"
            min="1"
            max="1000"
            v-model="modal.numberOfLargeSteps"
          />
        </div>
        <div class="control">
          <label for="number-of-small-steps">Number of small steps</label>
          <input
            id="number-of-small-steps"
            type="number"
            min="1"
            max="1000"
            v-model="modal.numberOfSmallSteps"
          />
        </div>
        <div class="control">
          <label for="size-of-large-step">Size of large step</label>
          <input id="size-of-large-step" type="number" v-model="modal.sizeOfLargeStep" />
        </div>
        <div class="control">
          <label for="size-of-small-step">Size of small step</label>
          <input id="size-of-small-step" type="number" v-model="modal.sizeOfSmallStep" />
        </div>
        <div class="control">
          <label for="up">Bright generators up</label>
          <input
            id="up"
            type="number"
            min="0"
            :max="modal.upMax"
            :step="modal.numberOfPeriods"
            v-model="modal.up"
          />
        </div>
        <div class="control">
          <label for="equave">Equave</label>
          <ScaleLineInput
            id="equave"
            @update:value="modal.equave = $event"
            v-model="modal.equaveString"
            :defaultValue="OCTAVE"
          />
        </div>
        <div class="control radio-group">
          <label>Generate key colors</label>
          <span>
            <input type="radio" id="color-none" value="none" v-model="modal.colorMethod" />
            <label for="color-none"> Off </label>
          </span>
          <span>
            <input type="radio" id="color-parent" value="parent" v-model="modal.colorMethod" />
            <label for="color-parent"> Parent MOS </label>
          </span>
          <span>
            <input type="radio" id="color-daughter" value="daughter" v-model="modal.colorMethod" />
            <label for="color-daughter"> Daughter MOS (expand scale) </label>
          </span>
        </div>
        <div class="control radio-group" v-show="modal.colorMethod === 'parent'">
          <label>Black keys are</label>
          <span>
            <input type="radio" id="sharp" value="sharp" v-model="modal.parentColorAccidentals" />
            <label for="sharp"> Sharp </label>
          </span>
          <span>
            <input type="radio" id="flat" value="flat" v-model="modal.parentColorAccidentals" />
            <label for="flat"> Flat </label>
          </span>
        </div>
        <div class="control radio-group" v-show="modal.colorMethod === 'daughter'">
          <label>Accidentals to include</label>
          <span>
            <input type="radio" id="sharp" value="sharp" v-model="modal.daughterColorAccidentals" />
            <label for="sharp"> Sharp </label>
          </span>
          <span>
            <input type="radio" id="flat" value="flat" v-model="modal.daughterColorAccidentals" />
            <label for="flat"> Flat </label>
          </span>
          <span>
            <input type="radio" id="both" value="both" v-model="modal.daughterColorAccidentals" />
            <label for="both"> Both </label>
          </span>
          <span>
            <input type="radio" id="all" value="all" v-model="modal.daughterColorAccidentals" />
            <label for="all"> Full ET </label>
          </span>
        </div>
      </div>

      <div class="pyramid" @mouseleave="modal.previewL = 0" v-show="modal.method === 'pyramid'">
        <div v-for="key of 11" :key="key" @mousemove.self="modal.previewL = 0">
          <button
            v-for="l of key"
            :key="l"
            @mouseenter="pyramidMouseEnter(l, key)"
            @click="pyramidClick(l, key)"
          >
            {{ l }}L {{ key + 1 - l }}s
          </button>
        </div>
      </div>

      <div class="control-group" @mouseleave="modal.previewL = 0" v-show="modal.method === 'edo'">
        <div class="control">
          <label for="edo">EDO</label>
          <input id="edo" type="number" min="2" v-model="modal.edo" />
        </div>
        <span v-for="(info, i) of modal.edoList" @mouseenter="edoMouseEnter(info)" :key="i">
          <button @click="edoClick(info)">
            {{ info.mosPattern }}
          </button>
          {{ info.hardness }}
          <i v-if="info.name !== undefined">[{{ info.name.split(';')[0] }}]</i>
        </span>
        <button @click="modal.sortByHardness">Sort by hardness</button>
        <button @click="modal.sortBySize">Sort by size</button>
        <button @click="modal.moreForEdo">Discover</button>
        <div class="control">
          <label for="min-size">Minimum size</label>
          <input id="min-size" type="number" min="2" v-model="modal.minSize" />
        </div>
        <div class="control">
          <label for="max-size">Maximum size</label>
          <input id="max-size" type="number" :min="modal.minSize" v-model="modal.maxSize" />
        </div>
        <div class="control">
          <label for="max-hardness">Maximum hardness</label>
          <input id="max-hardness" type="number" min="2" v-model="modal.maxHardness" />
        </div>
      </div>
    </template>
    <template #footer>
      <div class="btn-group">
        <button @click="generate">OK</button>
        <button @click="$emit('cancel')">Cancel</button>
        <template v-if="modal.method === 'direct'">
          {{ modal.ed }}<i>{{ modal.tamnamsName }}</i
          >{{ modal.mosModeInfo.mode }} {{ modal.mosModeInfo.udp
          }}<i v-if="modal.mosModeInfo.modeName">"{{ modal.mosModeInfo.modeName }}"</i
          ><i>{{ modal.hardness }}</i>
        </template>
        <i v-else>{{ modal.previewName }}</i>
      </div>
    </template>
  </Modal>
</template>

<style scoped>
.pyramid {
  text-align: center;
  overflow-x: auto;
}
.pyramid div {
  white-space: nowrap;
}
.pyramid button {
  font-size: small;
}
@media only screen and (max-width: 38rem) {
  .pyramid {
    text-align: left;
  }
  .pyramid button {
    width: 4.5em;
  }
}
</style>
