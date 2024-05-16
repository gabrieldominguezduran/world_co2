<script>
  import { getContext, onMount, onDestroy, afterUpdate } from "svelte";

  export let path;
  export let stroke = undefined;
  export let fill = undefined;
  export let contextName = "canvas";

  const { register, deregister, invalidate } = getContext(contextName);

  function draw(ctx) {
    const p = new Path2D(path);
    ctx.fillStyle = fill;
    ctx.fill(p);
    if (stroke) {
      ctx.strokeStyle = stroke;
      ctx.stroke(p);
    }
  }

  onMount(() => {
    register(draw);
    invalidate();
  });

  onDestroy(() => {
    deregister(draw);
  });

  afterUpdate(invalidate);
</script>
