<script>
  import WorldMap from "./components/WorldMap.svelte";

  let width, height;
</script>

<main bind:clientWidth={width} bind:clientHeight={height}>
  <WorldMap />
</main>

<style>
</style>
