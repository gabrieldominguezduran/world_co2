<script>
  import { onMount, onDestroy, setContext } from "svelte";

  export let width;
  export let height;
  export let contextName = "canvas";

  const drawFunctions = [];
  let mouseMoveFunctions = [];

  let canvas;
  let ctx;
  let pendingInvalidation = false;
  let frameId;

  function scaleCanvas(canvas, ctx, width, height) {
    const devicePixelRatio = window.devicePixelRatio || 1;

    canvas.width = width * devicePixelRatio;
    canvas.height = height * devicePixelRatio;
    canvas.style.width = width + "px";
    canvas.style.height = height + "px";

    ctx.scale(devicePixelRatio, devicePixelRatio);
  }

  function update() {
    if (!ctx) return;

    ctx.clearRect(0, 0, width, height);

    drawFunctions.forEach((fn) => {
      ctx.save();
      fn(ctx);
      ctx.restore();
    });

    pendingInvalidation = false;
  }

  function handleMouseMove(event) {
    const rect = canvas.getBoundingClientRect();
    const mouseX = event.clientX - rect.left;
    const mouseY = event.clientY - rect.top;

    mouseMoveFunctions.forEach((fn) => fn(mouseX, mouseY));
  }

  onMount(() => {
    ctx = canvas.getContext("2d");
    canvas.addEventListener("mousemove", handleMouseMove);
  });

  onDestroy(() => {
    if (frameId) {
      cancelAnimationFrame(frameId);
    }
    canvas.removeEventListener("mousemove", handleMouseMove);
  });

  $: setContext(contextName, {
    register(fn) {
      drawFunctions.push(fn);
    },
    deregister(fn) {
      drawFunctions.splice(drawFunctions.indexOf(fn), 1);
    },
    registerMouseMove(fn) {
      mouseMoveFunctions.push(fn);
    },
    deregisterMouseMove(fn) {
      mouseMoveFunctions.splice(mouseMoveFunctions.indexOf(fn), 1);
    },
    invalidate() {
      if (pendingInvalidation) return;
      pendingInvalidation = true;
      frameId = requestAnimationFrame(update);
    },
  });

  $: if (canvas && ctx) scaleCanvas(canvas, ctx, width, height);
</script>

<!-- svelte-ignore invalid_self_closing_tag -->
<canvas bind:this={canvas} />
<slot />

<style>
  canvas {
    display: block;
  }
</style>
