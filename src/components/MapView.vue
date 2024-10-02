<script setup>
import { ref, onMounted } from 'vue';
import { MapboxMap, MapboxMarker } from '@studiometa/vue-mapbox-gl';
import '@mapbox/mapbox-gl-geocoder/lib/mapbox-gl-geocoder.css'
import axios from 'redaxios'

const tokenMapBox = import.meta.env.VITE_MAPBOX_API_KEY

const mapCenter = ref([2.3522, 48.8566])

const getData = async () => {
  const response = await axios.get('https://opendata.paris.fr/api/explore/v2.1/catalog/datasets/dechets-menagers-points-dapport-volontaire-recycleries-et-ressourceries/records?limit=20')
  console.log(response.data)
}

onMounted(() => {
  getData()
})
</script>

<template>
  <MapboxMap
      style="height: 600px"
      :access-token="tokenMapBox"
      map-style="mapbox://styles/mapbox/streets-v12"
      :center="mapCenter"
      :zoom="10"
      :min-zoom="5"
      :max-zoom="15">
      <MapboxMarker/>
  </MapboxMap>
</template>