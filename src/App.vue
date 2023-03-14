<script setup>
import { onMounted, ref } from "vue";
import "leaflet/dist/leaflet.css";
import L from "leaflet";
import markers from "./markers.json";
import polygons from "./polygons.json";

const map = ref(null);
const newMarkerLat = ref(null);
const newMarkerLng = ref(null);
const newMarkerTitle = ref(null);
const newMarkerDescription = ref(null);
const newMarkerIcon = ref('keep')
const newMarker = ref(null);

const icons = {
  'keep': L.icon({ iconUrl: '/icons/keep.png', iconSize: [32, 32], iconAnchor: [16, 16], popupAnchor: [0, -8] }),
  'tower': L.icon({ iconUrl: '/icons/tower.png', iconSize: [32, 32], iconAnchor: [16, 16], popupAnchor: [0, -8] }),
  'camp': L.icon({ iconUrl: '/icons/camp.png', iconSize: [32, 32], iconAnchor: [16, 16], popupAnchor: [0, -8] }),
  'castle': L.icon({ iconUrl: '/icons/castle.png', iconSize: [32, 32], iconAnchor: [16, 16], popupAnchor: [0, -8] }),
  'gate': L.icon({ iconUrl: '/icons/gate.png', iconSize: [32, 32], iconAnchor: [16, 16], popupAnchor: [0, -8] }),
  'supply': L.icon({ iconUrl: '/icons/supply.png', iconSize: [32, 32], iconAnchor: [16, 16], popupAnchor: [0, -8] }),
};

onMounted(() => {
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

});

function submitMarker() {
  alert('TODO!')
  // console.log('newMarker:', newMarkerTitle.value, newMarkerLat.value, newMarkerLng.value);
}

function onMapClick(e) {
  const coords = map.value.project([e.latlng.lat, e.latlng.lng], map.value.getMaxZoom());
  console.log("Point @ [" + coords.x + ", " + coords.y + "]");
  console.log("Coords @ [" + e.latlng.lat + ", " + e.latlng.lng + "]");

  console.log('clickMap:', newMarker.value, newMarkerTitle.value, newMarkerDescription.value, newMarkerLat.value, newMarkerLng.value)

  if (newMarker.value) {
    map.value.removeLayer(newMarker.value)
  }

  newMarker.value = L.marker(map.value.unproject([coords.x, coords.y], map.value.getMaxZoom()), { icon: icons[newMarkerIcon.value] }).addTo(map.value);

  let title = null, description = null;

  if (newMarkerTitle.value) {
    title = `<h4>${newMarkerTitle.value}</h4>`;
  }

  if (newMarkerDescription.value) {
    description = `<p>${newMarkerDescription.value}</p>`;
  }

  if (title || description) {
    newMarker.value.bindPopup(((title) ? title : '') + ((description) ? description : '')).openPopup();
  }

  // newMarkerLat.value = coords.x;
  // newMarkerLng.value = coords.y;
}

function updateNewMarkerIcon() {
  if (newMarker.value) {
    newMarker.value.setIcon(icons[newMarkerIcon.value])
  }
}

function updateNewMarkerPopup() {
  if (newMarker.value) {
    let title = null, description = null;

    if (newMarkerTitle.value) {
      title = `<h4>${newMarkerTitle.value}</h4>`;
    }

    if (newMarkerDescription.value) {
      description = `<p>${newMarkerDescription.value}</p>`;
    }

    if (title || description) {
      newMarker.value.bindPopup(((title) ? title : '') + ((description) ? description : '')).openPopup();
    }
  }
}

function resetNewMarker() {
  if (newMarker.value) {
    map.value.removeLayer(newMarker.value);

    newMarker.value = null;
    newMarkerTitle.value = null;
    newMarkerDescription.value = null;
    newMarkerIcon.value = 'keep';
    newMarkerLat.value = null;
    newMarkerLng.value = null;
  }
}

// export default {
//   data: () => {
//     return {
//       map: null,
//       newMarkerTitle: null,
//       newMarkerDescription: null,
//       newMarkerLat: null,
//       newMarkerLng: null,
//     }
//   },
//   mounted() {
//     const map = L.map("map", {
//       crs: L.CRS.Simple,
//       minZoom: 2,
//       maxZoom: 5,
//     });

//     map.setView(map.unproject([4095, 4095], map.getMaxZoom()), 3)

//     L.tileLayer("https://v2.lebusmagique.fr/img/map/wvw-tiles/{z}/{x}/{y}.jpg", {
//       minZoom: 0,
//       maxZoom: 5,
//       continuousWorld: true,
//       maxBoundsViscosity: 0.8,
//       noWrap: true
//     }).addTo(map);

//     map.setMaxBounds(new L.LatLngBounds(map.unproject([1466, 2173], map.getMaxZoom()), map.unproject([6724, 6015], map.getMaxZoom())));

//     const icons = {
//       'keep': L.icon({ iconUrl: '/icons/keep.png', iconSize: [32, 32], iconAnchor: [16, 10] }),
//       'tower': L.icon({ iconUrl: '/icons/tower.png', iconSize: [32, 32], iconAnchor: [16, 10] }),
//       'camp': L.icon({ iconUrl: '/icons/camp.png', iconSize: [32, 32], iconAnchor: [16, 10] }),
//       'castle': L.icon({ iconUrl: '/icons/castle.png', iconSize: [32, 32], iconAnchor: [16, 10] }),
//       'gate': L.icon({ iconUrl: '/icons/gate.png', iconSize: [32, 32], iconAnchor: [16, 10] }),
//       'supply': L.icon({ iconUrl: '/icons/supply.png', iconSize: [32, 32], iconAnchor: [16, 10] }),
//     };

//     markers.forEach(m => {
//       const marker = L.marker(map.unproject([m.lat, m.lng], map.getMaxZoom()), { icon: icons[m.icon] }).addTo(map);

//       if (typeof m.title !== 'undefined') {
//         marker.bindPopup(`<strong>${m.title}</strong>`)
//       }
//     });

//     polygons.forEach(p => {
//       L.polygon(p.coords, { color: p.color }).addTo(map);
//     });

//     map.on('click', this.onMapClick);

//     this.map = map;

//   },
//   methods: {
//     addMarker() {
//       const icon = L.icon({ iconUrl: '/icons/keep.png', iconSize: [32, 32], iconAnchor: [16, 10] });
//       L.marker(this.map.unproject([3528, 4344], this.map.getMaxZoom()), { icon: icon }).addTo(this.map)
//       console.log('addMarker')
//     },
//     submitMarker() {
//       console.log('newMarker:', this.newMarkerTitle, this.newMarkerDescription, this.newMarkerLat, this.newMarkerLng)
//     },
//     onMapClick(e) {
//       const coords = this.map.project([e.latlng.lat, e.latlng.lng], this.map.getMaxZoom());
//       console.log("Point @ [" + coords.x + ", " + coords.y + "]");
//       console.log("Coords @ [" + e.latlng.lat + ", " + e.latlng.lng + "]");

//       this.newMarkerLat = coords.x;
//       this.newMarkerLng = coords.y;
//     }
//   }
// };
</script>

<template>
  <form @submit.prevent="submitMarker" class="absolute bottom-4 left-4 bg-white rounded-xl p-3 w-64">
    <div>
      <input type="text" v-model="newMarkerTitle" placeholder="Titre" @input="updateNewMarkerPopup">
    </div>
    <div>
      <textarea v-model="newMarkerDescription" rows="5" placeholder="Description"
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
      <select id="icon" v-model="newMarkerIcon" @change="updateNewMarkerIcon">
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

form {
  z-index: 999;
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
</style>