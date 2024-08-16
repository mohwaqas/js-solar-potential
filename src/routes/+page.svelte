<!--
 Copyright 2023 Google LLC

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at

      https://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
 -->
<script context="module">
    export const prerender = false;
</script>
<script lang="ts">
	/* global google */
	import { Loader } from '@googlemaps/js-api-loader';
	import { onMount } from 'svelte';
	import { page } from '$app/stores'

	import SearchBar from './components/SearchBar.svelte';
	import Sections from './sections/Sections.svelte';
	const address = $page?.url?.searchParams?.get('address')

	if(address) {
		console.log('address found in url')
	}

	const googleMapsApiKey = import.meta.env.VITE_GOOGLE_MAPS_API_KEY;
	const defaultPlace = {
		name: '',
    	address,
	};
	let location: google.maps.LatLng | undefined;
	const zoom = 19;

	// Initialize app.
	let mapElement: HTMLElement;
	let map: google.maps.Map;
	let geometryLibrary: google.maps.GeometryLibrary;
	let mapsLibrary: google.maps.MapsLibrary;
	let placesLibrary: google.maps.PlacesLibrary;
	onMount(async () => {
		// Load the Google Maps libraries.
		const loader = new Loader({ apiKey: googleMapsApiKey });
		const libraries = {
			geometry: loader.importLibrary('geometry'),
			maps: loader.importLibrary('maps'),
			places: loader.importLibrary('places'),
		};
		geometryLibrary = await libraries.geometry;
		mapsLibrary = await libraries.maps;
		placesLibrary = await libraries.places;

		// Get the address information for the default location.
		if (!defaultPlace.address) {
			alert('Pls enter address')
			return
		}
		const geocoder = new google.maps.Geocoder();
		const geocoderResponse = await geocoder.geocode({ address: defaultPlace.address });
		const geocoderResult = geocoderResponse.results[0];
		console.log('geocoderResult', geocoderResponse)

		// Initialize the map at the desired location.
		location = geocoderResult.geometry.location;
		map = new mapsLibrary.Map(mapElement, {
			center: location,
			zoom: zoom,
			tilt: 0,
			mapTypeId: 'satellite',
			mapTypeControl: false,
			fullscreenControl: false,
			rotateControl: false,
			streetViewControl: false,
			zoomControl: false,
		});
	});
</script>

<!-- Top bar -->
<div class="flex flex-row h-full">
	<!-- Main map -->
	<div bind:this={mapElement} class="w-full" />

	<!-- Side bar -->
	<aside class="flex-none md:w-96 w-80 p-2 pt-3 overflow-auto">
		<div class="flex flex-col space-y-2 h-full">
			{#if placesLibrary && map}
				<SearchBar bind:location {placesLibrary} {map} initialValue={defaultPlace.name} />
			{/if}

			{#if location}
				<Sections {location} {map} {geometryLibrary} {googleMapsApiKey} />
			{/if}

			<div class="grow" />

		</div>
	</aside>
</div>
