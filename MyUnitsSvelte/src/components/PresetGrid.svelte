<script>
  export let presets = []
  export let selectedPresetId = ''
</script>

<section class="preset-panel" aria-labelledby="presets-title">
  <div class="section-heading">
    <p class="eyebrow">Express</p>
    <h2 id="presets-title">Conversiones rapidas</h2>
  </div>

  <div class="preset-grid">
    {#each presets as preset (preset.id)}
      <button
        class:active={selectedPresetId === preset.id}
        class="preset-card"
        type="button"
        onclick={() => (selectedPresetId = preset.id)}
      >
        <span>{preset.group}</span>
        <strong>{preset.from} -> {preset.to}</strong>
        <small>{preset.hint}</small>
      </button>
    {/each}
  </div>
</section>
