<script setup>
import { onMounted, ref } from "vue";
import { v4 as uuidv4 } from 'uuid';
import "leaflet/dist/leaflet.css";
import L from "leaflet";
import markers from "./markers.json";
import polygons from "./polygons.json";

const map = ref(null);
const newMarker = ref({
  marker: null,
  icon: 'keep',
  title: null,
  description: null,
  lat: null,
  lng: null,
});
const markerEditing = ref(false);
const userMarkers = ref([]);

const icons = {
  'keep': L.icon({ iconUrl: '/icons/keep.png', iconSize: [32, 32], iconAnchor: [16, 16], popupAnchor: [0, -8] }),
  'tower': L.icon({ iconUrl: '/icons/tower.png', iconSize: [32, 32], iconAnchor: [16, 16], popupAnchor: [0, -8] }),
  'camp': L.icon({ iconUrl: '/icons/camp.png', iconSize: [32, 32], iconAnchor: [16, 16], popupAnchor: [0, -8] }),
  'castle': L.icon({ iconUrl: '/icons/castle.png', iconSize: [32, 32], iconAnchor: [16, 16], popupAnchor: [0, -8] }),
  'gate': L.icon({ iconUrl: '/icons/gate.png', iconSize: [32, 32], iconAnchor: [16, 16], popupAnchor: [0, -8] }),
  'supply': L.icon({ iconUrl: '/icons/supply.png', iconSize: [32, 32], iconAnchor: [16, 16], popupAnchor: [0, -8] }),
};

onMounted(() => {
  initMap();
});

function initMap() {
  const initMap = L.map("map", {
    crs: L.CRS.Simple,
    minZoom: 2,
    maxZoom: 5,
  });

  initMap.setView(initMap.unproject([4095, 4095], initMap.getMaxZoom()), 3);

  L.tileLayer("https://v2.lebusmagique.fr/img/map/wvw-tiles/{z}/{x}/{y}.jpg", {
    minZoom: 0,
    maxZoom: 5,
    maxBoundsViscosity: 0.8,
    noWrap: true
  }).addTo(initMap);

  initMap.setMaxBounds(new L.LatLngBounds(initMap.unproject([1466, 2173], initMap.getMaxZoom()), initMap.unproject([6724, 6015], initMap.getMaxZoom())));

  markers.forEach(m => {
    const marker = L.marker(initMap.unproject([m.lat, m.lng], initMap.getMaxZoom()), { icon: icons[m.icon] }).addTo(initMap);
    if (typeof m.title !== 'undefined') {
      marker.bindPopup(`<h4>${m.title}</h4>`)
    }
  });

  polygons.forEach(p => {
    L.polygon(p.coords, { color: p.color }).addTo(initMap);
  });

  initMap.on('click', onMapClick);

  map.value = initMap;

  initUserMarkers();
}

function submitMarker() {
  if (!newMarker.value.lat || !newMarker.value.lng) {
    alert('Tu dois positionner un marqueur sur la carte !');
  }
  console.log(newMarker.value);

  addUserMarker(newMarker.value);

  // Reset marker sans fermer le popup
  if (newMarker.value.marker) {
    map.value.removeLayer(newMarker.value.marker);

    newMarker.value = {
      marker: null,
      icon: 'keep',
      title: null,
      description: null,
      lat: null,
      lng: null,
    }
  }


}

function onMapClick(e) {
  const coords = map.value.project([e.latlng.lat, e.latlng.lng], map.value.getMaxZoom());
  console.log("Point @ [" + coords.x + ", " + coords.y + "]");
  console.log("Coords @ [" + e.latlng.lat + ", " + e.latlng.lng + "]");

  if (markerEditing.value) {
    if (newMarker.value.marker) {
      map.value.removeLayer(newMarker.value.marker)
    }

    newMarker.value.marker = L.marker(map.value.unproject([coords.x, coords.y], map.value.getMaxZoom()), { icon: icons[newMarker.value.icon] }).addTo(map.value);

    let title = null, description = null;

    if (newMarker.value.title) {
      title = `<h4>${newMarker.value.title}</h4>`;
    }

    if (newMarker.value.description) {
      description = `<p>${newMarker.value.description}</p>`;
    }

    if (title || description) {
      newMarker.value.marker.bindPopup(((title) ? title : '') + ((description) ? description : '')).openPopup();
    }

    newMarker.value.lat = coords.x;
    newMarker.value.lng = coords.y;
  }

}

function initUserMarkers() {
  const localMarkers = localStorage.getItem('markers');
  if (localMarkers) {
    userMarkers.value = JSON.parse(localMarkers);

    userMarkers.value.forEach((m) => {
      const marker = L.marker(map.value.unproject([m.lat, m.lng], map.value.getMaxZoom()), { icon: icons[m.icon] }).addTo(map.value);

      let title = null, description = null;

      if (m.title) {
        title = `<h4>${m.title}</h4>`;
      }

      if (m.description) {
        description = `<p>${m.description}</p>`;
      }

      if (title || description) {
        marker.bindPopup(((title) ? title : '') + ((description) ? description : ''));
      }
    });
  }
}

function addUserMarker(m) {
  let newUserMarker = {
    "id": uuidv4(),
    "lat": m.lat,
    "lng": m.lng,
    "icon": m.icon,
    "title": m.title,
    "description": m.description
  }

  userMarkers.value.push(newUserMarker);
  localStorage.setItem('markers', JSON.stringify(userMarkers.value));
  console.log(newUserMarker);
}

function updateNewMarkerIcon() {
  if (newMarker.value.marker) {
    newMarker.value.marker.setIcon(icons[newMarker.value.icon])
  }
}

function updateNewMarkerPopup() {
  if (newMarker.value.marker) {
    let title = null, description = null;

    if (newMarker.value.title) {
      title = `<h4>${newMarker.value.title}</h4>`;
    }

    if (newMarker.value.description) {
      description = `<p>${newMarker.value.description}</p>`;
    }

    if (title || description) {
      newMarker.value.marker.bindPopup(((title) ? title : '') + ((description) ? description : '')).openPopup();
    }
  }
}

function resetNewMarker() {
  if (newMarker.value.marker) {
    map.value.removeLayer(newMarker.value.marker);

    newMarker.value = {
      marker: null,
      icon: 'keep',
      title: null,
      description: null,
      lat: null,
      lng: null,
    }
  }

  markerEditing.value = !markerEditing.value
}

</script>

<template>
  <button class="absolute bottom-4 left-4 left-4 bg-white rounded-xl p-2"
    @click="() => { markerEditing = !markerEditing }" v-if="!markerEditing">
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="w-5 h-5">
      <path
        d="M21.731 2.269a2.625 2.625 0 00-3.712 0l-1.157 1.157 3.712 3.712 1.157-1.157a2.625 2.625 0 000-3.712zM19.513 8.199l-3.712-3.712-8.4 8.4a5.25 5.25 0 00-1.32 2.214l-.8 2.685a.75.75 0 00.933.933l2.685-.8a5.25 5.25 0 002.214-1.32l8.4-8.4z" />
      <path
        d="M5.25 5.25a3 3 0 00-3 3v10.5a3 3 0 003 3h10.5a3 3 0 003-3V13.5a.75.75 0 00-1.5 0v5.25a1.5 1.5 0 01-1.5 1.5H5.25a1.5 1.5 0 01-1.5-1.5V8.25a1.5 1.5 0 011.5-1.5h5.25a.75.75 0 000-1.5H5.25z" />
    </svg>
  </button>
  <form @submit.prevent="submitMarker" class="absolute bottom-4 left-4 bg-white rounded-xl p-3 w-64" v-if="markerEditing">
    <div>
      <input type="text" v-model="newMarker.title" placeholder="Titre" @input="updateNewMarkerPopup">
    </div>
    <div>
      <textarea v-model="newMarker.description" rows="5" placeholder="Description"
        @input="updateNewMarkerPopup"></textarea>
    </div>
    <div>
      <label for="icon">
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" class="w-5 h-5">
          <path fill-rule="evenodd"
            d="M9.69 18.933l.003.001C9.89 19.02 10 19 10 19s.11.02.308-.066l.002-.001.006-.003.018-.008a5.741 5.741 0 00.281-.14c.186-.096.446-.24.757-.433.62-.384 1.445-.966 2.274-1.765C15.302 14.988 17 12.493 17 9A7 7 0 103 9c0 3.492 1.698 5.988 3.355 7.584a13.731 13.731 0 002.273 1.765 11.842 11.842 0 00.976.544l.062.029.018.008.006.003zM10 11.25a2.25 2.25 0 100-4.5 2.25 2.25 0 000 4.5z"
            clip-rule="evenodd" />
        </svg>
      </label>
      <select id="icon" v-model="newMarker.icon" @change="updateNewMarkerIcon">
        <option value="keep">Fort</option>
        <option value="tower">Tour</option>
        <option value="camp">Camp</option>
        <option value="castle">Château</option>
        <option value="gate">Porte</option>
        <option value="supply">Ravitaillement</option>
      </select>
    </div>
    <div>
      <button type="submit">
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" class="w-5 h-5">
          <path
            d="M10.75 4.75a.75.75 0 00-1.5 0v4.5h-4.5a.75.75 0 000 1.5h4.5v4.5a.75.75 0 001.5 0v-4.5h4.5a.75.75 0 000-1.5h-4.5v-4.5z" />
        </svg>
        Ajouter
      </button>
      <button type="reset" @click="resetNewMarker">
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" class="w-5 h-5">
          <path
            d="M6.28 5.22a.75.75 0 00-1.06 1.06L8.94 10l-3.72 3.72a.75.75 0 101.06 1.06L10 11.06l3.72 3.72a.75.75 0 101.06-1.06L11.06 10l3.72-3.72a.75.75 0 00-1.06-1.06L10 8.94 6.28 5.22z" />
        </svg>
        Annuler
      </button>
    </div>

  </form>
  <div id="map"></div>
</template>

<style>
#map {
  height: 100vh;
  width: 100vw;
  cursor: crosshair;
}

.leaflet-container {
  background: #000;
  outline: 0;
}

form input,
form select,
form textarea {
  @apply border rounded py-2 px-3 w-full;
}

form>div:not(:last-of-type) {
  @apply mb-2;
}

form>div {
  @apply flex gap-2 items-center;
}

form button {
  @apply py-2 px-3 w-full text-white rounded flex gap-1 items-center justify-center uppercase text-xs font-semibold;
}

form button[type="submit"] {
  @apply bg-emerald-500;
}

form button[type="reset"] {
  @apply bg-gray-500;
}

.leaflet-popup h4 {
  @apply text-base font-bold;
}

.leaflet-popup p {
  @apply my-0 text-sm;
}

.absolute {
  z-index: 999;
}
</style>