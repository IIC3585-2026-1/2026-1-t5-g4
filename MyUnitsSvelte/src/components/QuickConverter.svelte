<script>
  export let preset

  let inputValue = 1

  $: numericValue = Number(inputValue)
  $: result = Number.isFinite(numericValue) ? preset.convert(numericValue) : ''
  $: formattedResult =
    result === ''
      ? 'Ingresa un numero valido'
      : new Intl.NumberFormat('es-CL', { maximumFractionDigits: 6 }).format(result)
</script>

<section class="quick-panel" aria-labelledby="quick-title">
  <div class="section-heading">
    <p class="eyebrow">{preset.group}</p>
    <h2 id="quick-title">{preset.label}</h2>
  </div>

  <label class="quick-input">
    <span>Cantidad en {preset.from}</span>
    <input bind:value={inputValue} inputmode="decimal" type="number" />
  </label>

  <div class="quick-result">
    <span>Resultado</span>
    <strong>{formattedResult} {preset.to}</strong>
    <p>{inputValue || 0} {preset.from} equivalen a {formattedResult} {preset.to}</p>
  </div>
</section>
