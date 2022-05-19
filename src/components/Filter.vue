<script setup lang="ts">
import { computed } from 'vue';

const props = defineProps(['modelValue', 'name']);
const emit = defineEmits(['update:modelValue']);
const max = computed<number>(() => props.modelValue.max);
const step = computed<number>(() => {
  if (max.value > 100) {
    return Math.floor(max.value / 100);
  } else {
    return Math.round(max.value * 100) / 10000;
  }
});
const from = computed<number>({
  get() {
    return props.modelValue.from;
  },
  set(value) {
    const { from, ...data } = props.modelValue;
    emit('update:modelValue', { ...data, from: Number(value) });
  },
});
const to = computed<number>({
  get() {
    return props.modelValue.to;
  },
  set(value) {
    const { to, ...data } = props.modelValue;
    emit('update:modelValue', { ...data, to: Number(value) });
  },
});
</script>

<template>
  <div>
    <span>{{ name }}</span>
    <label for="from">От</label>
    <input type="number" id="from" min="0" :max="max" v-model="from" />
    <label for="to">До</label>
    <input type="number" id="to" min="0" :max="max" v-model="to" />
    <div class="ranges">
      <input type="range" :max="max" :step="step" v-model="from" />
      <input type="range" :max="max" :step="step" v-model="to" />
    </div>
  </div>
</template>
