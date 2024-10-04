<script setup>
import { ref, onMounted } from 'vue';
import mapboxgl from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';

// Référence pour l'instance de la carte et pour les données API
const map = ref(null);
const markersData = ref([]);
const selectedMarker = ref(null);

const mapboxToken = import.meta.env.VITE_MAPBOX_API_KEY;
mapboxgl.accessToken = mapboxToken;

// Fonction pour récupérer les données de l'API
const getData = async () => {
  const response = await fetch('https://opendata.paris.fr/api/explore/v2.1/catalog/datasets/dechets-menagers-points-dapport-volontaire-recycleries-et-ressourceries/records?limit=20');
  const data = await response.json();
  markersData.value = data.results;
  return data.results;
};

// Fonction pour initialiser la carte
const initMap = async () => {
  if (!map.value) {
    map.value = new mapboxgl.Map({
      container: 'map',
      style: 'mapbox://styles/mapbox/streets-v12',
      center: [2.3522, 48.8566], // Coordonnées de Paris
      zoom: 11
    });

    map.value.addControl(new mapboxgl.NavigationControl());

    // Attendre le chargement de la carte avant d'ajouter des marqueurs
    map.value.on('load', async () => {
      const apiMarkersData = await getData();
      addMarkers(apiMarkersData);
    });
  }
};

// Fonction pour ajouter des marqueurs à la carte
const addMarkers = (markersData) => {
  markersData.forEach(markerData => {
    const {longitude, latitude, operateur, adresse} = markerData;

    // Ajouter un tooltip au marqueur avec l'opérateur et l'adresse
    const popup = new mapboxgl.Popup({offset: 25}).setText(`${operateur}, ${adresse}`);

    // Créer et ajouter le marqueur à la carte (marqueur basique de Mapbox)
    const marker = new mapboxgl.Marker()
        .setLngLat([longitude, latitude])
        .setPopup(popup)
        .addTo(map.value);

    // Ajouter un événement de clic sur chaque marqueur pour mettre à jour `selectedMarker` et faire un flyTo
    marker.getElement().addEventListener('click', () => {
      selectedMarker.value = markerData;

      map.value.flyTo({
        center: [longitude, latitude],
        zoom: 14,
        speed: 1.5,
        essential: true
      });
    });
  });
};

// Générer le lien vers Google Maps pour l'itinéraire
const getGoogleMapsLink = (latitude, longitude) => {
  const currentLocation = 'current+location'; // Utilise la position actuelle de l'utilisateur dans Google Maps
  return `https://www.google.com/maps/dir/?api=1&origin=${currentLocation}&destination=${latitude},${longitude}&travelmode=driving`;
};

onMounted(() => {
  initMap();
});
</script>

<template>
  <div class="map-container">
    <div id="map"></div>
  </div>
  <div class="card">
    <h3>Points de collecte des métaux à Paris</h3>
    <ul v-if="selectedMarker">
      <li>
        <strong>Opérateur:</strong> {{ selectedMarker.operateur }}
      </li>
      <li>
        <strong>Adresse:</strong> {{ selectedMarker.adresse }}
      </li>
      <li v-if="selectedMarker.code_postal">
        <strong>Code Postal:</strong> {{ selectedMarker.code_postal }}
      </li>
    </ul>
    <p v-else>
      Sélcetionnez un marqueur ✨
    </p>

    <!-- Bouton pour afficher l'itinéraire dans Google Maps -->
    <a v-if="selectedMarker"
       :href="getGoogleMapsLink(selectedMarker.latitude, selectedMarker.longitude)"
       target="_blank"
       class="btn-itinerary">
      Afficher l'itinéraire
    </a>
  </div>
</template>

<style scoped>
.map-container {
  height: 80vh;
  z-index: 2;
}

#map {
  width: 100%;
  height: 100%;
}

.card {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  background-color: white;
  height: 25vh;
  border-radius: 20px 20px 0 0;
  z-index: 10;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  box-shadow: 0 -2px 10px rgba(0, 0, 0, 0.2);
  padding: 10px;
}

.card h3 {
  font-size: 1.2rem;
  margin-bottom: 20px;
}

.card ul {
  list-style-type: none;
  padding: 0;
  margin: 0;
  font-size: 0.9rem;
  overflow-y: auto;
  max-height: 60%;
  text-align: start;
}

.card li {
  margin-bottom: 5px;
}

.card p {
  color: #333;
  text-align: center;
}

.btn-itinerary {
  margin-top: 20px;
  background-color: #0D744E;
  color: white;
  padding: 10px 15px;
  text-decoration: none;
  width: 25%;
  text-align: center;
  transition: background-color 0.3s;
}

.btn-itinerary:hover {
  background-color: rgba(13, 116, 78, 0.88);
}
</style>
