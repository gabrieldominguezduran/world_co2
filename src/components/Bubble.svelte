<script>
  import { getContext, onMount, onDestroy, afterUpdate } from "svelte";

  export let countryData;
  export let projection;
  export let contextName = "canvas";
  export let selectedMetric;
  export let maxValues;
  export let color;
  export let abbreviateNumber;

  let cx, cy;
  let hovered = false;
  const {
    register,
    deregister,
    invalidate,
    registerMouseMove,
    deregisterMouseMove,
  } = getContext(contextName);

  function draw(ctx) {
    if (cx !== undefined && cy !== undefined) {
      let metricValue, maxValue;
      switch (selectedMetric) {
        case "gdp":
          metricValue = countryData.gdp;
          maxValue = maxValues.gdp;
          break;
        case "population":
          metricValue = countryData.population;
          maxValue = maxValues.population;
          break;
        case "emissions":
        default:
          metricValue = countryData.emissions;
          maxValue = maxValues.emissions;
          break;
      }

      const baseRadius = Math.sqrt(metricValue / maxValue) * 100;
      const radius = hovered ? baseRadius + (60 - baseRadius) : baseRadius; // Larger growth for smaller bubbles

      ctx.beginPath();
      ctx.arc(cx, cy, radius, 0, 2 * Math.PI);
      ctx.fillStyle = color;
      ctx.fill();
      ctx.strokeStyle = "#fff";
      ctx.lineWidth = 1;
      ctx.stroke();

      ctx.fillStyle = "#fff";
      ctx.font = `bold ${radius / 3.5}px Arial`;
      ctx.textAlign = "center";
      ctx.textBaseline = "middle";

      const lines = [
        `${countryData.rank}`,
        countryData.country,
        abbreviateNumber(metricValue) +
          (selectedMetric === "emissions" ? " CO₂" : ""),
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

  function handleMouseMove(mouseX, mouseY) {
    const baseRadius =
      Math.sqrt(countryData[selectedMetric] / maxValues[selectedMetric]) * 50;
    const distance = Math.sqrt((mouseX - cx) ** 2 + (mouseY - cy) ** 2);

    if (distance <= baseRadius) {
      if (!hovered) {
        hovered = true;
        document.body.classList.add("pointer-cursor");
        invalidate(); // Redraw the bubble
      }
    } else {
      if (hovered) {
        hovered = false;
        document.body.classList.remove("pointer-cursor");
        invalidate(); // Redraw the bubble
      }
    }
  }

  onMount(() => {
    register(draw);
    registerMouseMove(handleMouseMove);
    invalidate();
  });

  onDestroy(() => {
    deregister(draw);
    deregisterMouseMove(handleMouseMove);
  });

  afterUpdate(invalidate);
</script>
