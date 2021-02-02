<script>
  import { onMount } from 'svelte';
	import L from 'leaflet';
	import Leaflet from './Leaflet.svelte';
	import Control from './Control.svelte';
	import Marker from './Marker.svelte';
	import Popup from './Popup.svelte';
	import Polyline from './Polyline.svelte';
	import MapToolbar from './MapToolbar.svelte';
	import { scaleSequential } from 'd3-scale';
	import { interpolateRainbow } from 'd3-scale-chromatic';

	let map;
  let parsedRows = [];
  let markerLocations = [];
  let colors;
  let lines;

  onMount(async () => {
    const res = await fetch('__CSV_FILE__')
    const text = await res.text()

    const rows = text.split('\n').map(str => str.split(','))
    const headers = rows.shift();
    parsedRows = rows
      //.filter(row => row.length === headers.length)
      .map(row => {
        let readRow = {}
        headers.forEach((key, idx) => readRow[key] = row[idx]) 
        readRow.X = +readRow.X
        readRow.Y = +readRow.Y
        return readRow
      })
      .filter(row => +row.X);
    markerLocations = parsedRows.map(item => [item.Y, item.X])
    console.log("loaded", markerLocations.length);
  })

  $: colors = scaleSequential(interpolateRainbow).domain([0, markerLocations.length - 1]);
  $: lines = markerLocations.slice(1).map((latLng, i) => {
		let prev = markerLocations[i];
		return {
			latLngs: [prev, latLng],
			color: colors(i),
		}
	});
	
	const initialView = [46.666, 4.84];
	
	let eye = true;
	let showLines = true;

	function resizeMap() {
	  if(map) { map.invalidateSize(); }
  }
	
	function resetMapView() {
		map.setView(initialView, 5);
	}

</script>
<svelte:window on:resize={resizeMap} />

<Leaflet bind:map view={initialView} zoom={6}>
  <Control position="topright">
    <MapToolbar bind:eye bind:lines={showLines} on:click-reset={resetMapView} />
  </Control>

  {#if eye }
    {#each markerLocations as latLng, ind}
      <Marker {latLng} width={30} height={30}>
        <svg style="width:20px;height:20px" class="w-6 h-6" data-darkreader-inline-fill="none" data-darkreader-inline-stroke="" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6"></path></svg>

        <Popup>{parsedRows[ind]['label']}</Popup>
      </Marker> 
    {/each}
  {/if}

  {#if showLines}
    {#each lines as {latLngs, color}}
      <Polyline {latLngs} {color} opacity={0.5} />
    {/each}
  {/if}
</Leaflet>
