<script setup lang="ts">
import { DEFAULT_NUMBER_OF_COMPONENTS } from '@/constants'
import { computedAndError, setAndReportValidity } from '@/utils'
import { parseLine, type Interval } from 'scale-workshop-core'
import { ref, watch } from 'vue'

const props = defineProps<{
  modelValue: string
  defaultValue: Interval
  placeholder?: string
}>()

const emit = defineEmits(['update:value', 'update:modelValue'])

const element = ref<HTMLInputElement | null>(null)
const [value, error] = computedAndError(
  () => parseLine(props.modelValue, DEFAULT_NUMBER_OF_COMPONENTS),
  props.defaultValue
)
watch(value, (newValue) => emit('update:value', newValue), { immediate: true })
watch(element, (newElement) => setAndReportValidity(newElement, error.value), { immediate: true })
watch(error, (newError) => setAndReportValidity(element.value, newError), { immediate: true })
</script>

<template>
  <input
    ref="element"
    type="text"
    :placeholder="placeholder"
    :value="modelValue"
    @input="$emit('update:modelValue', ($event.target as HTMLInputElement)!.value)"
  />
</template>
