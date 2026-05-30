<script setup>
import { computed, ref } from 'vue'
import ThemeToggle from './components/ThemeToggle.vue'
import UnitConverter from './components/UnitConverter.vue'
import SupportedUnits from './components/SupportedUnits.vue'
import logoUrl from './assets/cinta-metrica.png'

const categories = [
  {
    id: 'data',
    name: 'Datos digitales',
    description: 'Aplica prefijos binarios a bytes: KB, MB, GB y TB.',
    baseUnit: 'byte',
    prefixes: [
      { id: 'none', label: 'Sin prefijo', symbol: '', factor: 1 },
      { id: 'kilo', label: 'Kilo', symbol: 'K', factor: 1024 },
      { id: 'mega', label: 'Mega', symbol: 'M', factor: 1024 ** 2 },
      { id: 'giga', label: 'Giga', symbol: 'G', factor: 1024 ** 3 },
      { id: 'tera', label: 'Tera', symbol: 'T', factor: 1024 ** 4 },
    ],
    units: [
      { id: 'byte', label: 'Byte', symbol: 'B', factor: 1, supportsPrefixes: true },
    ],
  },
  {
    id: 'length',
    name: 'Longitud',
    description: 'Usa prefijos metricos sobre metro y unidades comunes sin prefijo.',
    baseUnit: 'meter',
    prefixes: [
      { id: 'none', label: 'Sin prefijo', symbol: '', factor: 1 },
      { id: 'milli', label: 'Mili', symbol: 'm', factor: 0.001 },
      { id: 'centi', label: 'Centi', symbol: 'c', factor: 0.01 },
      { id: 'kilo', label: 'Kilo', symbol: 'k', factor: 1000 },
      { id: 'mega', label: 'Mega', symbol: 'M', factor: 1000 ** 2 },
      { id: 'giga', label: 'Giga', symbol: 'G', factor: 1000 ** 3 },
      { id: 'tera', label: 'Tera', symbol: 'T', factor: 1000 ** 4 },
    ],
    units: [
      { id: 'meter', label: 'Metro', symbol: 'm', factor: 1, supportsPrefixes: true },
      { id: 'inch', label: 'Pulgada', symbol: 'in', factor: 0.0254, supportsPrefixes: false },
      { id: 'foot', label: 'Pie', symbol: 'ft', factor: 0.3048, supportsPrefixes: false },
      { id: 'mile', label: 'Milla', symbol: 'mi', factor: 1609.344, supportsPrefixes: false },
    ],
  },
  {
    id: 'mass',
    name: 'Masa',
    description: 'Usa prefijos metricos sobre gramo y unidades imperiales sin prefijo.',
    baseUnit: 'gram',
    prefixes: [
      { id: 'none', label: 'Sin prefijo', symbol: '', factor: 1 },
      { id: 'milli', label: 'Mili', symbol: 'm', factor: 0.001 },
      { id: 'centi', label: 'Centi', symbol: 'c', factor: 0.01 },
      { id: 'kilo', label: 'Kilo', symbol: 'k', factor: 1000 },
      { id: 'mega', label: 'Mega', symbol: 'M', factor: 1000 ** 2 },
      { id: 'giga', label: 'Giga', symbol: 'G', factor: 1000 ** 3 },
      { id: 'tera', label: 'Tera', symbol: 'T', factor: 1000 ** 4 },
    ],
    units: [
      { id: 'gram', label: 'Gramo', symbol: 'g', factor: 1, supportsPrefixes: true },
      { id: 'pound', label: 'Libra', symbol: 'lb', factor: 453.59237, supportsPrefixes: false },
      { id: 'ounce', label: 'Onza', symbol: 'oz', factor: 28.349523125, supportsPrefixes: false },
    ],
  },
  {
    id: 'speed',
    name: 'Velocidad',
    description: 'Unidades frecuentes para movimiento.',
    baseUnit: 'meter-second',
    prefixes: [],
    units: [
      { id: 'meter-second', label: 'Metro/segundo', symbol: 'm/s', factor: 1, supportsPrefixes: false },
      { id: 'kilometer-hour', label: 'Kilometro/hora', symbol: 'km/h', factor: 1 / 3.6, supportsPrefixes: false },
      { id: 'mile-hour', label: 'Milla/hora', symbol: 'mph', factor: 0.44704, supportsPrefixes: false },
      { id: 'knot', label: 'Nudo', symbol: 'kn', factor: 0.514444, supportsPrefixes: false },
    ],
  },
  {
    id: 'temperature',
    name: 'Temperatura',
    description: 'Celsius, Fahrenheit y Kelvin.',
    baseUnit: 'celsius',
    prefixes: [],
    units: [
      { id: 'celsius', label: 'Celsius', symbol: 'C', supportsPrefixes: false },
      { id: 'fahrenheit', label: 'Fahrenheit', symbol: 'F', supportsPrefixes: false },
      { id: 'kelvin', label: 'Kelvin', symbol: 'K', supportsPrefixes: false },
    ],
  },
]

const selectedCategoryId = ref('data')
const isDarkMode = ref(true)

const selectedCategory = computed(() =>
  categories.find((category) => category.id === selectedCategoryId.value),
)
</script>

<template>
  <div class="app-shell" :class="{ 'theme-dark': isDarkMode, 'theme-light': !isDarkMode }">
    <header class="app-header">
      <div class="brand-lockup">
        <img class="brand-logo" :src="logoUrl" alt="Logo de MyUnits" />

        <div>
          <p class="eyebrow">Grupo 4 - Frameworks Vue y Svelte</p>
          <h1>MyUnits</h1>
          <p class="intro">
            Convertidor de unidades hecho en Vue. Esta base ya tiene estado, componentes y tema
            claro/oscuro para seguir construyendo la version final.
          </p>
        </div>
      </div>

      <ThemeToggle v-model="isDarkMode" />
    </header>

    <main class="workspace">
      <section class="converter-panel" aria-labelledby="converter-title">
        <div class="section-heading">
          <p class="eyebrow">Conversor</p>
          <h2 id="converter-title">Selecciona un area y convierte</h2>
        </div>

        <div class="category-tabs" aria-label="Categorias de conversion">
          <button
            v-for="category in categories"
            :key="category.id"
            class="category-tab"
            :class="{ active: selectedCategoryId === category.id }"
            type="button"
            @click="selectedCategoryId = category.id"
          >
            {{ category.name }}
          </button>
        </div>

        <UnitConverter :category="selectedCategory" />
      </section>

      <SupportedUnits :categories="categories" />
    </main>
  </div>
</template>
