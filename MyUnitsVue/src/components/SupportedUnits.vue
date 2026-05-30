<script setup>
defineProps({
  categories: {
    type: Array,
    required: true,
  },
})
</script>

<template>
  <aside class="support-panel" aria-labelledby="support-title">
    <div class="section-heading">
      <p class="eyebrow">Soportadas</p>
      <h2 id="support-title">Unidades incluidas</h2>
    </div>

    <div class="unit-list">
      <section v-for="category in categories" :key="category.id" class="unit-group">
        <h3>{{ category.name }}</h3>
        <ul>
          <li v-for="unit in category.units" :key="unit.id">
            <span>{{ unit.label }}</span>
            <code>{{ unit.symbol }}</code>
          </li>
        </ul>
        <p v-if="category.prefixes.length" class="prefix-list">
          Prefijos:
          <span v-for="prefix in category.prefixes" :key="prefix.id">
            {{ prefix.symbol || 'base' }}
          </span>
        </p>
      </section>
    </div>
  </aside>
</template>
