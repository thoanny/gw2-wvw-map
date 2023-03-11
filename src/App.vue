<script>
import "leaflet/dist/leaflet.css";
import L from "leaflet";
import markers from "./markers.json";
import polygons from "./polygons.json";

export default {
  mounted() {
    const map = L.map("map", {
      crs: L.CRS.Simple,
      minZoom: 2,
      maxZoom: 5,
    });

    map.setView(map.unproject([4095, 4095], map.getMaxZoom()), 3)

    L.tileLayer("https://v2.lebusmagique.fr/img/map/wvw-tiles/{z}/{x}/{y}.jpg", {
      minZoom: 0,
      maxZoom: 5,
      continuousWorld: true,
      maxBoundsViscosity: 0.8,
      noWrap: true
    }).addTo(map);

    map.setMaxBounds(new L.LatLngBounds(map.unproject([1466, 2173], map.getMaxZoom()), map.unproject([6724, 6015], map.getMaxZoom())));

    const icons = {
      'keep': L.icon({ iconUrl: '/icons/keep.png', iconSize: [32, 32], iconAnchor: [16, 16] }),
      'tower': L.icon({ iconUrl: '/icons/tower.png', iconSize: [32, 32], iconAnchor: [16, 16] }),
      'camp': L.icon({ iconUrl: '/icons/camp.png', iconSize: [32, 32], iconAnchor: [16, 16] }),
      'castle': L.icon({ iconUrl: '/icons/castle.png', iconSize: [32, 32], iconAnchor: [16, 16] }),
      'gate': L.icon({ iconUrl: '/icons/gate.png', iconSize: [32, 32], iconAnchor: [16, 16] }),
      'supply': L.icon({ iconUrl: '/icons/supply.png', iconSize: [32, 32], iconAnchor: [16, 16] }),
    };

    markers.forEach(m => {
      L.marker(map.unproject([m.lat, m.lng], map.getMaxZoom()), { icon: icons[m.icon] }).addTo(map);
    });

    polygons.forEach(p => {
      L.polygon(p.coords, { color: p.color }).addTo(map);
    })

    map.on('click', function (e) {
      console.log("Position @ " + map.project([e.latlng.lat, e.latlng.lng], map.getMaxZoom()));
      console.log("Coords @ [" + e.latlng.lat + ", " + e.latlng.lng + "]");
    });

  }
};
</script>

<template>
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
</style>