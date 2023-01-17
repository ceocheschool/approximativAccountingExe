<script>
  function toggle() {
    window.document.body.classList.toggle("dark-mode");
    localStorage.setItem("dark-mode", window.document.body.classList.contains("dark-mode"));
  }
</script>

<button on:click={toggle}>
  <slot />
</button>

<style>
  button {
    position: absolute;
    background-color: #f76027;
    color: white;
    border: none;
    border-radius: 4px;
    padding: 0.5rem;
    right: 10px;
    margin-top: 15px;
    text-transform: uppercase;
  }
  :global(body.dark-mode) button {
    background-color: #0084f6;
    color: white;
  }
</style>
