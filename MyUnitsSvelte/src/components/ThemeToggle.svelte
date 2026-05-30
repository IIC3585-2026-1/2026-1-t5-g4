<script>
  export let isDarkMode = true
</script>

<label class="theme-toggle">
  <span>Claro</span>
  <input bind:checked={isDarkMode} type="checkbox" />
  <span class="switch" aria-hidden="true"></span>
  <span>Noche</span>
</label>
