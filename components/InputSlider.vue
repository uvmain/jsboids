<script setup lang="ts">
defineProps({
  label: { type: String, required: true},
  modelValue: { type: Number, required: true},
  min: { type: Number, default: 0},
  max: { type: Number, default: 100},
  step: { type: Number, default: 1}
});

const emit = defineEmits(['update:modelValue'])

function handleInput(e: Event) {
  const value = (e.target as HTMLInputElement).value
  emit('update:modelValue', value)
}
</script>

<template>
  <div>
    <div>{{ label }}: {{ modelValue }}</div>
    <input
      type="range"
      :value="modelValue"
      :min="min"
      :max="max"
      :step="step"
      @input="handleInput"
    />
  </div>
</template>