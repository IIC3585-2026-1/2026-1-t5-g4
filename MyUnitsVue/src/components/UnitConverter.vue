<script setup>
import { computed, ref, watch } from 'vue'

const props = defineProps({
  category: {
    type: Object,
    required: true,
  },
})

const inputValue = ref(1)
const fromUnitId = ref(props.category.units[0].id)
const toUnitId = ref(props.category.units[1]?.id || props.category.units[0].id)
const fromPrefixId = ref('none')
const toPrefixId = ref('kilo')
const history = ref([])
const copied = ref(false)
const isResettingCategory = ref(false)
let copyTimerId = null
let lastLoggedSignature = ''

watch(
  () => props.category,
  (category) => {
    isResettingCategory.value = true
    history.value = []
    copied.value = false
    lastLoggedSignature = ''

    fromUnitId.value = category.units[0].id
    toUnitId.value = category.units[1]?.id || category.units[0].id
    fromPrefixId.value = 'none'
    toPrefixId.value = category.prefixes[1]?.id || 'none'
    inputValue.value = 1

    isResettingCategory.value = false
  },
)

const fromUnit = computed(() => props.category.units.find((unit) => unit.id === fromUnitId.value))
const toUnit = computed(() => props.category.units.find((unit) => unit.id === toUnitId.value))
const hasPrefixes = computed(() => props.category.prefixes.length > 0)
const fromUnitSupportsPrefixes = computed(() => Boolean(fromUnit.value?.supportsPrefixes && hasPrefixes.value))
const toUnitSupportsPrefixes = computed(() => Boolean(toUnit.value?.supportsPrefixes && hasPrefixes.value))

const fromPrefix = computed(() => findPrefix(fromPrefixId.value))
const toPrefix = computed(() => findPrefix(toPrefixId.value))

watch(fromUnitId, () => {
  if (!fromUnitSupportsPrefixes.value) {
    fromPrefixId.value = 'none'
  }
})

watch(toUnitId, () => {
  if (!toUnitSupportsPrefixes.value) {
    toPrefixId.value = 'none'
  }
})

const result = computed(() => {
  const value = Number(inputValue.value)

  if (!Number.isFinite(value)) {
    return ''
  }

  if (props.category.id === 'temperature') {
    return convertTemperature(value, fromUnitId.value, toUnitId.value)
  }

  const baseValue = value * fromUnit.value.factor * prefixFactor(fromUnit.value, fromPrefix.value)
  return baseValue / (toUnit.value.factor * prefixFactor(toUnit.value, toPrefix.value))
})

const formattedResult = computed(() => {
  if (result.value === '') {
    return 'Ingresa un numero valido'
  }

  return new Intl.NumberFormat('es-CL', {
    maximumFractionDigits: 6,
  }).format(result.value)
})

watch(
  [result, inputValue, fromUnitId, toUnitId, fromPrefixId, toPrefixId, () => props.category.id],
  () => {
    if (isResettingCategory.value || result.value === '') {
      return
    }

    const signature = buildConversionSignature()

    if (signature === lastLoggedSignature) {
      return
    }

    lastLoggedSignature = signature
    history.value = [createHistoryEntry(), ...history.value].slice(0, 5)
  },
  { flush: 'sync' },
)

function invertUnits() {
  const previousFrom = fromUnitId.value
  const previousFromPrefix = fromPrefixId.value

  fromUnitId.value = toUnitId.value
  fromPrefixId.value = toPrefixId.value
  toUnitId.value = previousFrom
  toPrefixId.value = previousFromPrefix
}

async function copyResult() {
  if (result.value === '' || !navigator.clipboard?.writeText) {
    return
  }

  try {
    await navigator.clipboard.writeText(String(result.value))
  } catch {
    return
  }

  copied.value = true

  if (copyTimerId) {
    clearTimeout(copyTimerId)
  }

  copyTimerId = window.setTimeout(() => {
    copied.value = false
    copyTimerId = null
  }, 1500)
}

function findPrefix(prefixId) {
  return props.category.prefixes.find((prefix) => prefix.id === prefixId) || defaultPrefix()
}

function defaultPrefix() {
  return { id: 'none', label: 'Sin prefijo', symbol: '', factor: 1 }
}

function prefixFactor(unit, prefix) {
  return unit.supportsPrefixes ? prefix.factor : 1
}

function unitDisplayName(unit, prefix) {
  if (!unit.supportsPrefixes || prefix.id === 'none') {
    return unit.symbol
  }

  return `${prefix.symbol}${unit.symbol}`
}

function convertTemperature(value, from, to) {
  let celsius = value

  if (from === 'fahrenheit') {
    celsius = (value - 32) * (5 / 9)
  }

  if (from === 'kelvin') {
    celsius = value - 273.15
  }

  if (to === 'fahrenheit') {
    return celsius * (9 / 5) + 32
  }

  if (to === 'kelvin') {
    return celsius + 273.15
  }

  return celsius
}

function buildConversionSignature() {
  return [
    props.category.id,
    inputValue.value,
    fromUnitId.value,
    toUnitId.value,
    fromPrefixId.value,
    toPrefixId.value,
    result.value,
  ].join('|')
}

function createHistoryEntry() {
  return {
    id: `${Date.now()}-${Math.random().toString(36).slice(2, 8)}`,
    fromLabel: unitDisplayName(fromUnit.value, fromPrefix.value),
    toLabel: unitDisplayName(toUnit.value, toPrefix.value),
    inputLabel: formatHistoryValue(inputValue.value),
    resultLabel: formattedResult.value,
  }
}

function formatHistoryValue(value) {
  if (value === '') {
    return '0'
  }

  const numericValue = Number(value)

  if (!Number.isFinite(numericValue)) {
    return String(value)
  }

  return new Intl.NumberFormat('es-CL', {
    maximumFractionDigits: 6,
  }).format(numericValue)
}
</script>

<template>
  <article class="converter-card">
    <div class="converter-summary">
      <h3>{{ category.name }}</h3>
      <p>{{ category.description }}</p>
    </div>

    <div class="converter-grid">
      <label class="field amount-field">
        <span>Cantidad</span>
        <input v-model="inputValue" type="number" inputmode="decimal" />
      </label>

      <div class="unit-control">
        <span class="control-label">Desde</span>
        <div class="unit-selects">
          <select v-if="hasPrefixes" v-model="fromPrefixId" :disabled="!fromUnitSupportsPrefixes">
            <option v-for="prefix in category.prefixes" :key="prefix.id" :value="prefix.id">
              {{ prefix.label }}
            </option>
          </select>

          <select v-model="fromUnitId">
            <option v-for="unit in category.units" :key="unit.id" :value="unit.id">
              {{ unit.label }} ({{ unit.symbol }})
            </option>
          </select>
        </div>
      </div>

      <button class="swap-button" type="button" @click="invertUnits">Invertir</button>

      <div class="unit-control">
        <span class="control-label">Hacia</span>
        <div class="unit-selects">
          <select v-if="hasPrefixes" v-model="toPrefixId" :disabled="!toUnitSupportsPrefixes">
            <option v-for="prefix in category.prefixes" :key="prefix.id" :value="prefix.id">
              {{ prefix.label }}
            </option>
          </select>

          <select v-model="toUnitId">
            <option v-for="unit in category.units" :key="unit.id" :value="unit.id">
              {{ unit.label }} ({{ unit.symbol }})
            </option>
          </select>
        </div>
      </div>
    </div>

    <div class="result-box">
      <span>Resultado</span>
      <div class="result-value-row">
        <strong>{{ formattedResult }} {{ unitDisplayName(toUnit, toPrefix) }}</strong>
        <button
          class="copy-button"
          type="button"
          :disabled="result === ''"
          :aria-label="copied ? 'Resultado copiado' : 'Copiar resultado al portapapeles'"
          :title="copied ? 'Resultado copiado' : 'Copiar resultado'"
          @click="copyResult"
        >
          <svg viewBox="0 0 24 24" aria-hidden="true">
            <path
              d="M16 1H6a2 2 0 0 0-2 2v12h2V3h10V1Zm3 4H10a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h9a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2Zm0 16h-9V7h9v14Z"
              fill="currentColor"
            />
          </svg>
          <span>{{ copied ? 'Copiado' : 'Copiar' }}</span>
        </button>
      </div>
      <p>
        {{ inputValue || 0 }} {{ unitDisplayName(fromUnit, fromPrefix) }} equivalen a
        {{ formattedResult }} {{ unitDisplayName(toUnit, toPrefix) }}
      </p>

      <div v-if="history.length" class="history-block">
        <h4>Historial reciente</h4>
        <ul class="history-list">
          <li v-for="entry in history" :key="entry.id" class="history-item">
            <span>{{ entry.inputLabel }} {{ entry.fromLabel }} → {{ entry.resultLabel }} {{ entry.toLabel }}</span>
          </li>
        </ul>
      </div>
    </div>
  </article>
</template>
