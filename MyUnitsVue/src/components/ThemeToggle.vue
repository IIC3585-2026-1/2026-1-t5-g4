<script setup>
defineProps({
  modelValue: {
    type: Boolean,
    required: true,
  },
})

defineEmits(['update:modelValue'])
</script>

<template>
  <label class="theme-toggle">
    <span>Claro</span>
    <input
      :checked="modelValue"
      type="checkbox"
      @change="$emit('update:modelValue', $event.target.checked)"
    />
    <span class="switch" aria-hidden="true"></span>
    <span>Noche</span>
  </label>
</template>
