<script>
  import { json, geoMercator, geoPath } from "d3";
  import Canvas from "./Canvas.svelte";
  import Country from "./Country.svelte";
  import Bubble from "./Bubble.svelte";
  import countriesData from "../data/top_emitters_2013.json";

  let width, height;
  let selectedMetric = "emissions";

  const colors = {
    emissions: "rgba(128, 0, 128, 0.8)", // Purple
    gdp: "rgba(0, 128, 128, 0.8)", // Teal
    population: "rgba(255, 165, 0, 0.8)", // Orange
  };

  const geojsonPath =
    "https://raw.githubusercontent.com/datasets/geo-countries/master/data/countries.geojson";

  const unwantedCountries = ["Antarctica", "Greenland"];

  function filterGeoJSON(data) {
    return {
      ...data,
      features: data.features.filter(
        (feature) => !unwantedCountries.includes(feature.properties.ADMIN)
      ),
    };
  }

  let geojson;
  let data = [];

  json(geojsonPath)
    .then((data) => {
      geojson = filterGeoJSON(data);
    })
    .catch((error) => console.error("Error loading GeoJSON:", error));

  data = countriesData
    .map((d, index) => ({
      country: d.Country_Name,
      emissions: +d.CO2kt,
      population: +d.population,
      gdp: +d.GDP,
      coordinates: d.coordinates,
    }))
    .filter((d) => d.coordinates || d.country === "World");

  let worldData = data.find((d) => d.country === "World");
  let top10Data = data.filter((d) => d.country !== "World").slice(0, 10); // Exclude "World" and take the top 10
  let top10Total = top10Data.reduce(
    (acc, curr) => acc + curr[selectedMetric],
    0
  );
  let restOfWorld = Math.max(0, worldData[selectedMetric] - top10Total);

  function updateDataForSelectedMetric() {
    data.sort((a, b) => b[selectedMetric] - a[selectedMetric]);
    data.forEach((d, index) => {
      d.rank = index;
    });
    top10Data = data.filter((d) => d.country !== "World").slice(0, 10); // Exclude "World" and take the top 10
    top10Total = top10Data.reduce((acc, curr) => acc + curr[selectedMetric], 0);
    restOfWorld = Math.max(0, worldData[selectedMetric] - top10Total); // Ensure restOfWorld is not negative
  }

  let projection, pathGenerator;
  $: if (geojson) {
    projection = geoMercator().fitSize([width, height], geojson);
    pathGenerator = geoPath(projection);
  }

  let countries = [];
  $: if (geojson && pathGenerator) {
    countries = geojson.features.map((feature) => {
      return {
        ...feature,
        path: pathGenerator(feature),
      };
    });
  }

  const maxValues = {
    emissions: Math.max(...data.map((d) => d.emissions)),
    gdp: Math.max(...data.map((d) => d.gdp)),
    population: Math.max(...data.map((d) => d.population)),
  };

  function setSelectedMetric(metric) {
    selectedMetric = metric;
    updateDataForSelectedMetric();
  }

  function getButtonColor(metric) {
    return colors[metric];
  }

  function abbreviateNumber(value) {
    const suffixes = ["", "K", "M", "B", "T"];
    let suffixNum = 0;
    let shortValue = value;
    while (shortValue >= 1000 && suffixNum < suffixes.length - 1) {
      suffixNum++;
      shortValue /= 1000;
    }
    return shortValue.toFixed(2) + suffixes[suffixNum];
  }

  // Initialize the data with the default selected metric
  updateDataForSelectedMetric();
</script>

<div class="main" bind:clientWidth={width} bind:clientHeight={height}>
  <Canvas {width} {height}>
    <div class="title">
      <h1>Countries with the highest CO₂ emissions in 2013</h1>
    </div>
    <div class="world-data" style="color: {colors[selectedMetric]};">
      World: {abbreviateNumber(worldData[selectedMetric])}<br />
      Top 10 total: {abbreviateNumber(top10Total)}<br />
      Rest of the world: {abbreviateNumber(restOfWorld)}
    </div>
    {#each countries as { id, path }}
      <Country {path} fill="#FF5733" stroke="#fff" />
    {/each}
    {#each data as countryData}
      {#if projection && countryData.coordinates}
        <Bubble
          {countryData}
          {projection}
          {selectedMetric}
          {maxValues}
          color={colors[selectedMetric]}
          {abbreviateNumber}
        />
      {/if}
    {/each}
  </Canvas>
  <div class="labels">
    <button
      on:click={() => setSelectedMetric("emissions")}
      style="background-color: {getButtonColor('emissions')}">CO₂</button
    >
    <button
      on:click={() => setSelectedMetric("gdp")}
      style="background-color: {getButtonColor('gdp')}">GDP</button
    >
    <button
      on:click={() => setSelectedMetric("population")}
      style="background-color: {getButtonColor('population')}"
      >Population</button
    >
  </div>
</div>

<style>
  .main {
    background-color: #fff;
    width: 100vw;
    height: 100vh;
    overflow: hidden;
    position: relative;
  }

  .title {
    position: absolute;
    width: 100%;
    top: 0;
    left: 90%;
    transform: translateX(-50%);
    font-size: 1rem;
    font-weight: bold;
  }

  .world-data {
    position: absolute;
    bottom: 1rem;
    left: 1%;
    font-size: 1.3rem;
    font-weight: bold;
  }

  .labels {
    position: absolute;
    bottom: 3rem;
    right: 20%;
    display: flex;
    gap: 10px;
    flex-direction: row;
  }

  .labels button {
    padding: 1rem;
    color: #fff;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }

  .labels button:hover {
    filter: brightness(85%);
  }

  @media (max-width: 1200px) {
    .title {
      font-size: 0.8rem;
      left: 50%;
    }

    .world-data {
      font-size: 1rem;
    }

    .labels {
      bottom: 1rem;
      right: 1rem;
    }

    .labels button {
      padding: 0.5rem;
    }
  }
</style>
