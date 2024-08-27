<template>
  <section class="bodyMap">
    <l-map id="map" :zoom="zoom" :center="center">
      <l-tile-layer :url="url"></l-tile-layer>
      <l-marker v-for="marker in markers" :key="marker.id" :lat-lng="marker.latLng">
        <l-popup>{{ marker.info }}</l-popup>
      </l-marker>
    </l-map>
    <div class="bodyTable">
      <table>
        <transition-group name="slide" tag="tbody">
          <tr v-for="record in paginatedRecords" :key="record.id" @click="selectMarker(record.id)">
            <td>{{ record.name }}</td>
            <!-- other columns -->
          </tr>
        </transition-group>
      </table>
      <div class="pagination">
        <button v-for="n in totalPages" :key="n" @click="goToPage(n)">
          {{ n }}
        </button>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue';
import { LMap, LTileLayer, LMarker, LPopup } from '@vue-leaflet/vue-leaflet';
import 'leaflet/dist/leaflet.css';

const markers = ref([]);
const records = ref([]);
const zoom = ref(5);
const center = ref([35.6892, 51.3890]);
const url = 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png';

const currentPage = ref(1);
const recordsPerPage = 10;
let recordCounter = 3;

const paginatedRecords = computed(() => {
  const start = (currentPage.value - 1) * recordsPerPage;
  const end = start + recordsPerPage;
  return records.value.slice(start, end);
});

const totalPages = computed(() => Math.ceil(records.value.length / recordsPerPage));

const goToPage = (page) => {
  currentPage.value = page;
};

const testData = [
  { id: 1, latLng: [35.6892, 51.3890], info: 'Tehran', name: 'Marker 1', details: 'More details about Tehran', isVisible: false },
  { id: 2, latLng: [35.7000, 51.4000], info: 'Location 2', name: 'Marker 2', details: 'More details about Location 2', isVisible: false },
];
const fetchData = async () => {

  markers.value = testData;
  records.value = testData;
};

const toggleDetails = (id) => {
  const recordIndex = records.value.findIndex(record => record.id === id);
  if (recordIndex !== -1) { // Ensure the record was found
    records.value[recordIndex].isVisible = !records.value[recordIndex].isVisible;
  }
};

const updateData = () => {
  const newRecord = {
    id: recordCounter,
    latLng: [35.7100 + recordCounter * 0.01, 51.4200 + recordCounter * 0.01],
    info: `Location ${recordCounter}`,
    name: `Marker ${recordCounter}`,
    details: `More details about Marker ${recordCounter}`,
  };
  records.value.unshift(newRecord); // Prepend new record to the start of the array
  recordCounter++;
};

const selectMarker = (id) => {
  const selectedMarker = markers.value.find(marker => marker.id === id);
  if (selectedMarker) {
    center.value = selectedMarker.latLng;
    zoom.value = 14;
  }
};

onMounted(() => {
  fetchData();
  const interval = setInterval(() => {
    updateData();
    if (records.value.length >= 100) {
      clearInterval(interval);
    }
  }, 5000);
  onUnmounted(() => clearInterval(interval));
});
</script>

<style lang="scss">
#map {
  height: 400px !important;
  width: 100% !important;
}

.slide-enter-active,
.slide-leave-active {
  transition: all 0.5s ease;
}

.slide-enter-from,
.slide-leave-to {
  transform: translateY(-50px);
  opacity: 0;
}

.slide-enter-to,
.slide-leave-from {
  transform: translateY(0);
  opacity: 1;
}

.bodyMap {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
}

.detail-row td {
  background-color: #f0f0f0;
  /* Lighter shade for detail rows */
  /* Additional styling for detail content */
}

.bodyTable {
  width: 100%;
  margin-top: 20px;

  table {
    width: 100%;
    border-collapse: collapse;

    tr {
      background-color: #757373;
      border: 1px solid #ddd;
      transition: background-color 0.3s ease;

      &:hover {
        background-color: #474747;
      }
    }

    td {
      padding: 8px;
      text-align: left;
      cursor: pointer;
    }
  }

  .pagination {
    display: flex;
    justify-content: center;
    margin-top: 10px;

    button {
      margin: 0 5px;
      padding: 5px 10px;
      background-color: #f0f0f0;
      border: 1px solid #ddd;
      cursor: pointer;

      &:hover {
        background-color: #e0e0e0;
      }
    }
  }
}
</style>
