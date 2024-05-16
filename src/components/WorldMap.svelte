<script>
  import { json, geoMercator, geoPath } from "d3";
  import Canvas from "./Canvas.svelte";
  import Country from "./Country.svelte";
  import Bubble from "./Bubble.svelte";
  import countriesData from "../data/top10_countries_emissions_2023.json";

  let width, height;

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
      rank: index + 1,
      country: d.country,
      emissions: +d.emissions,
      population: +d.population,
      gdp: +d.gdp,
      emissionsPerCapita: +d.emissions_per_capita,
      emissionsPerGDP: +d.emissions_per_gdp,
      gdpPerCapita: +d.gdp_per_capita,
      coordinates: d.coordinates,
    }))
    .filter((d) => d.coordinates);

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
</script>

<div class="main" bind:clientWidth={width} bind:clientHeight={height}>
  <Canvas {width} {height}>
    {#each countries as { id, path }}
      <Country {path} fill="#FF5733" stroke="#fff" />
    {/each}
    {#each data as countryData}
      {#if projection && countryData.coordinates}
        <Bubble {countryData} {projection} />
      {/if}
    {/each}
  </Canvas>
</div>

<style>
  .main {
    background-color: #fff;
    width: 100vw;
    height: 100vh;
    overflow: hidden;
  }
</style>
