<script>
  import logoUrl from './assets/transicion.png'
  import PresetGrid from './components/PresetGrid.svelte'
  import QuickConverter from './components/QuickConverter.svelte'
  import ThemeToggle from './components/ThemeToggle.svelte'
  import { presets } from './data/presets.js'

  let selectedPresetId = 'meter-foot'
  let isDarkMode = true

  $: selectedPreset = presets.find((preset) => preset.id === selectedPresetId) || presets[0]
</script>

<main class:theme-dark={isDarkMode} class:theme-light={!isDarkMode} class="app-shell">
  <header class="app-header">
    <div class="brand-lockup">
      <img class="brand-logo" src={logoUrl} alt="Logo de MyUnits" />

      <div>
        <p class="eyebrow">Grupo 4 - Frameworks Vue y Svelte</p>
        <h1>MyUnits Express</h1>
        <p class="intro">
          Version Svelte enfocada en conversiones rapidas. Elige una tarjeta frecuente, ingresa
          un numero y ve el resultado sin configurar prefijos.
        </p>
      </div>
    </div>

    <ThemeToggle bind:isDarkMode />
  </header>

  <section class="express-layout">
    <QuickConverter preset={selectedPreset} />
    <PresetGrid {presets} bind:selectedPresetId />
  </section>
</main>
