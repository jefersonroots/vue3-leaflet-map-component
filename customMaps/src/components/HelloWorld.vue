<template>
  <div class="map-wrapper">
    <div id="mapContainer"></div>
      <div id="controlModal">
  <div class="modal-header">
    <h3>Camadas (ativa/desativa)</h3>
  </div>
  <div class="modal-content">
    <div class="line-toggle">
      <label>
        <input type="checkbox" v-model="showPolyline" /> Mostrar Linha
      </label>
    </div>
    <div class="marker-toggle">
      <label>
        <input type="checkbox" v-model="showMarkers" /> Mostrar Pinos
      </label>
    </div>
  </div>
    </div>
  </div>
</template>

<script>
import "leaflet/dist/leaflet.css";
import L from "leaflet";

export default {
  name: "LeafletMap",
  data() {
    return {
      map: null,
      orsApiKey: '5b3ce3597851110001cf6248a92ea283abbd4ee3945e197543175dcc',  // Substitua pela sua chave de API
      showModal: false,
      showMarkers: true,
      showPolyline: true,
      markers: [],
      roadPolyline: null,
    };
  },
  mounted() {
    // Centralize o mapa no Rio Grande do Sul
    this.map = L.map("mapContainer").setView([-30.0346, -51.2177], 7);
    
    // Adicione a camada de mapa
    L.tileLayer("http://{s}.tile.osm.org/{z}/{x}/{y}.png", {
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
    }).addTo(this.map);

    // Função para criar um marcador com popup
    const createMarker = (lat, lng, popupContent) => {
      const marker = L.marker([lat, lng]).addTo(this.map);
      marker.bindPopup(popupContent);
      this.markers.push(marker);
    };

    // Adicione marcadores com popups
    createMarker(-30.0346, -51.2177, '<b>Porto Alegre</b><br>Capital do Rio Grande do Sul.');
    createMarker(-29.1678, -51.1794, '<b>Caxias do Sul</b><br>Uma cidade próxima a Porto Alegre.');

    // Obtenha as coordenadas da rota
    this.getRouteCoordinates([-51.1794, -29.1678], [-51.2177, -30.0346]);
  },
  methods: {
    async getRouteCoordinates(start, end) {
      const url = `https://api.openrouteservice.org/v2/directions/driving-car?api_key=${this.orsApiKey}&start=${start.join(',')}&end=${end.join(',')}`;
      try {
        const response = await fetch(url);
        const data = await response.json();
        const coordinates = data.features[0].geometry.coordinates.map(coord => [coord[1], coord[0]]);
        
        // Adicione a linha poligonal da estrada ao mapa
        this.roadPolyline = L.polyline(coordinates, {
          color: 'orange',
          weight: 5,
          opacity: 0.7,
        }).addTo(this.map);
        
        // Ajuste a visão do mapa para a rota
        this.map.fitBounds(this.roadPolyline.getBounds());
      } catch (error) {
        console.error('Erro ao obter a rota:', error);
      }
    },
    toggleModal() {
      this.showModal = !this.showModal;
    }
  },
  watch: {
    showMarkers(val) {
      this.markers.forEach(marker => {
        if (val) {
          marker.addTo(this.map);
        } else {
          this.map.removeLayer(marker);
        }
      });
    },
    showPolyline(val) {
      if (this.roadPolyline) {
        if (val) {
          this.roadPolyline.addTo(this.map);
        } else {
          this.map.removeLayer(this.roadPolyline);
        }
      }
    }
  },
  beforeUnmount() {
    if (this.map) {
      this.map.remove();
    }
  },
};
</script>

<style scoped>
.map-wrapper {
  width: 1200px;
  height: 850px;
  position: relative;
}

#mapContainer {
  width: 100%;
  height: 100%;
}

#controlModal {
  position: absolute;
  bottom: 10px;
  left: 10px;
  background: white;
  color:black;
  padding: 10px;
  z-index: 1000; /* Definindo um z-index maior */
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.3);
}
.modal-header {
  margin-bottom: 10px;
}

.modal-header h3 {
  margin: 0;
  font-weight:bold
}

.modal-content {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 80px;
}

.marker-toggle,
.polyline-toggle {
  display: flex;
  align-items: center;
}
</style>
