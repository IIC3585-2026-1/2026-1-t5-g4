<script>
  export let preset;

  let inputValue = 1;
  let copied = false;
  let copyTimerId = null;

  let history = [];
  let lastSignature = "";
  let currentPresetId = "";
  let hasUserEditedInput = false;
  const HISTORY_STORAGE_PREFIX = "myunits-history-";

  $: numericValue = Number(inputValue);
  $: result = Number.isFinite(numericValue) ? preset.convert(numericValue) : "";
  $: formattedResult =
    result === ""
      ? "Ingresa un numero valido"
      : new Intl.NumberFormat("es-CL", { maximumFractionDigits: 6 }).format(
          result,
        );

  // Limpiar el historial si se cambia de conversión rápida
  $: if (preset && preset.id !== currentPresetId) {
    currentPresetId = preset.id;
    history = loadHistoryForPreset(preset.id);
    lastSignature = "";
    hasUserEditedInput = false;
    inputValue = 1;
  }

  // Registrar el historial reactivamente
  $: {
    if (result !== "" && preset && hasUserEditedInput) {
      const signature = `${preset.id}|${inputValue}|${result}`;

      if (signature !== lastSignature) {
        lastSignature = signature;

        const newEntry = {
          id: Date.now() + "-" + Math.random().toString(36).slice(2, 8),
          inputLabel: formatHistoryValue(inputValue),
          fromLabel: preset.from,
          resultLabel: formattedResult,
          toLabel: preset.to,
        };

        const isDuplicate = history.some(
          (entry) =>
            entry.inputLabel === newEntry.inputLabel &&
            entry.fromLabel === newEntry.fromLabel &&
            entry.resultLabel === newEntry.resultLabel &&
            entry.toLabel === newEntry.toLabel,
        );

        if (!isDuplicate) {
          history = [newEntry, ...history].slice(0, 5);
          saveHistoryForPreset(preset.id, history);
        }
      }
    }
  }

  function onInputChange() {
    hasUserEditedInput = true;
  }

  function loadHistoryForPreset(presetId) {
    if (typeof localStorage === "undefined") return [];

    try {
      const raw = localStorage.getItem(`${HISTORY_STORAGE_PREFIX}${presetId}`);
      if (!raw) return [];

      const parsed = JSON.parse(raw);
      return Array.isArray(parsed) ? parsed : [];
    } catch (e) {
      console.error("Error al leer historial", e);
      return [];
    }
  }

  function saveHistoryForPreset(presetId, entries) {
    if (typeof localStorage === "undefined") return;

    try {
      localStorage.setItem(
        `${HISTORY_STORAGE_PREFIX}${presetId}`,
        JSON.stringify(entries),
      );
    } catch (e) {
      console.error("Error al guardar historial", e);
    }
  }

  function formatHistoryValue(val) {
    if (val === "") return "0";
    const num = Number(val);
    if (!Number.isFinite(num)) return String(val);
    return new Intl.NumberFormat("es-CL", { maximumFractionDigits: 6 }).format(
      num,
    );
  }

  async function copyResult() {
    if (result === "" || !navigator.clipboard?.writeText) return;

    try {
      await navigator.clipboard.writeText(String(result));
      copied = true;

      if (copyTimerId) clearTimeout(copyTimerId);
      copyTimerId = setTimeout(() => {
        copied = false;
      }, 1500);
    } catch (e) {
      console.error("Error al copiar", e);
    }
  }
</script>

<section class="quick-panel" aria-labelledby="quick-title">
  <div class="section-heading">
    <p class="eyebrow">{preset.group}</p>
    <h2 id="quick-title">{preset.label}</h2>
  </div>

  <label class="quick-input">
    <span>Cantidad en {preset.from}</span>
    <input
      bind:value={inputValue}
      oninput={onInputChange}
      inputmode="decimal"
      type="number"
    />
  </label>

  <div class="quick-result">
    <span>Resultado</span>
    <div class="result-value-row">
      <strong>{formattedResult} {preset.to}</strong>

      <button
        class="copy-button"
        type="button"
        onclick={copyResult}
        disabled={result === ""}
      >
        <svg viewBox="0 0 24 24" aria-hidden="true">
          <path
            d="M16 1H6a2 2 0 0 0-2 2v12h2V3h10V1Zm3 4H10a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h9a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2Zm0 16h-9V7h9v14Z"
            fill="currentColor"
          />
        </svg>
        {copied ? "Copiado" : "Copiar"}
      </button>
    </div>
    <p>
      {inputValue || 0}
      {preset.from} equivalen a {formattedResult}
      {preset.to}
    </p>

    {#if history.length > 0}
      <div class="history-block">
        <h4>Historial reciente</h4>
        <ul class="history-list">
          {#each history as entry (entry.id)}
            <li class="history-item">
              <span
                >{entry.inputLabel}
                {entry.fromLabel} &rarr; {entry.resultLabel}
                {entry.toLabel}</span
              >
            </li>
          {/each}
        </ul>
      </div>
    {/if}
  </div>
</section>
