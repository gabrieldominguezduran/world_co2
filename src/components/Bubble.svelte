<script>
  import { getContext, onMount, onDestroy, afterUpdate } from "svelte";

  export let countryData;
  export let projection;
  export let contextName = "canvas";

  let cx, cy;
  const { register, deregister, invalidate } = getContext(contextName);

  function draw(ctx) {
    if (cx !== undefined && cy !== undefined) {
      const radius = Math.sqrt(countryData.emissions) / 1400; // Aumentar el tamaño de las burbujas

      // Dibuja el círculo
      ctx.beginPath();
      ctx.arc(cx, cy, radius, 0, 2 * Math.PI);
      ctx.fillStyle = "rgba(128, 0, 128, 0.8)"; // Color púrpura
      ctx.fill();
      ctx.strokeStyle = "#fff";
      ctx.lineWidth = 1;
      ctx.stroke();

      // Dibuja el texto
      ctx.fillStyle = "#fff";
      ctx.font = `bold ${radius / 3.5}px Arial`; // Ajustar el tamaño de la fuente
      ctx.textAlign = "center";
      ctx.textBaseline = "middle"; // Centrar verticalmente el texto

      // Texto múltiple
      const lines = [
        `${countryData.rank}`,
        countryData.country,
        abbreviateNumber(countryData.emissions) + " CO2",
      ];

      lines.forEach((line, index) => {
        ctx.fillText(line, cx, cy - radius / 2 + (index * radius) / 3);
      });
    }
  }

  $: {
    if (projection && countryData.coordinates) {
      const projected = projection(countryData.coordinates);
      if (projected) {
        cx = projected[0];
        cy = projected[1];
      } else {
        cx = undefined;
        cy = undefined;
      }
    } else {
      cx = undefined;
      cy = undefined;
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

  function abbreviateNumber(value) {
    const suffixes = ["", "K", "M", "B", "T"];
    let suffixNum = 0;
    let shortValue = value;
    while (shortValue >= 1000 && suffixNum < suffixes.length - 1) {
      suffixNum++;
      shortValue /= 1000;
    }
    return shortValue.toFixed(1) + suffixes[suffixNum];
  }
</script>
