<template>
  <div class="home row no-gutters">
    <div class="col-sm-3">
      <div v-if="cityName.length" class="toolbox">
        <div class="sticky-top bg-white shadow-sm p-2">
          <div class="form-group d-flex">
            <label for="cityName" class="mr-2 col-form-label text-right">City</label>
            <div class="flex-fill">
              <select id="cityName" class="form-control"
              v-model="select.city" @change="select.area = ''">
                <option value="">-- Select One --</option>
                <option :value="c.CityName" v-for="c in cityName" :key="c.CityName">
                  {{ c.CityName }}
                </option>
              </select>
            </div>
          </div>
          <div class="form-group d-flex">
            <label for="area" class="mr-2 col-form-label text-right">District</label>
            <div class="flex-fill">
              <select id="area" class="form-control" v-if="select.city.length"
                v-model="select.area" @change="updateSelect">
                <option value="">-- Select One --</option>
                <option :value="a.AreaName"
                  v-for="a in cityName.find((city) => city.CityName === select.city).AreaList"
                  :key="a.AreaName">
                  {{ a.AreaName }}
                </option>
              </select>
            </div>
          </div>
          <div class="form-group d-flex">
            <div class="ml-auto">
              <div class="form-check form-check-inline">
                <input class="form-check-input" type="checkbox"
                id="checkAdult" value="adultTrue" checked>
                <label class="form-check-label" for="checkAdult">Adult</label>
              </div>
              <div class="form-check form-check-inline">
                <input class="form-check-input" type="checkbox"
                id="checkChild" value="childTrue" checked>
                <label class="form-check-label" for="checkChild">Child</label>
              </div>
            </div>
            <div class="mr-auto">
              <div class="form-check form-check-inline">
                <input class="form-check-input" type="radio" v-model="inlineRadioOptions"
                id="check3km" value="3" @change="updateMarker">
                <label class="form-check-label" for="check3km">3km</label>
              </div>
              <div class="form-check form-check-inline">
                <input class="form-check-input" type="radio" v-model="inlineRadioOptions"
                id="check5km" value="5" @change="updateMarker">
                <label class="form-check-label" for="check5km">5km</label>
              </div>
              <div class="form-check form-check-inline">
                <input class="form-check-input" type="radio" v-model="inlineRadioOptions"
                id="checkAll" value="1000" checked @change="updateMarker">
                <label class="form-check-label" for="checkAll">all</label>
              </div>
            </div>
          </div>
          <p class="mb-0 small text-muted text-right">Search by district(green means available)</p>
        </div>
        <ul class="list-group">
          <template v-for="(item, key) in data">
            <a class="list-group-item text-left" :key="key"
              v-if="item.properties.county === select.city
                && item.properties.town === select.area
                && inlineRadioOptions === '1000'"
              :class="{ 'highlight': item.properties.mask_adult || item.properties.mask_child}"
              @click="penTo(item)">
              <h3>{{ item.properties.name }}</h3>
              <p class="mb-0">
                Adult: {{ item.properties.mask_adult}}  |  Child: {{ item.properties.mask_child}}
              </p>
              <p class="mb-0">
                 Distance：{{
                  getDistance(item.geometry.coordinates[0],
                  item.geometry.coordinates[1], 121.55, 25.03)}}
                  Km
              </p>
              <p class="mb-0">Address: <a :href="`https://www.google.com.tw/maps/place/${item.properties.address}`"
                target="_blank" title="Google Map">
                {{ item.properties.address }}</a>
              </p>
            </a>
            <a class="list-group-item text-left" :key="key"
              v-else-if="item.properties.county === select.city
                && item.properties.town === select.area
                && inlineRadioOptions === '3'
                && getDistance(item.geometry.coordinates[0],
                item.geometry.coordinates[1], 121.55, 25.03) < 3"
              :class="{ 'highlight': item.properties.mask_adult || item.properties.mask_child}"
              @click="penTo(item)">
              <h3>{{ item.properties.name }}</h3>
              <p class="mb-0">
                Adult: {{ item.properties.mask_adult}}  |  Child: {{ item.properties.mask_child}}
              </p>
              <p class="mb-0">
                 Distance：{{
                  getDistance(item.geometry.coordinates[0],
                  item.geometry.coordinates[1], 121.55, 25.03)}}
                  Km
              </p>
              <p class="mb-0">Address: <a :href="`https://www.google.com.tw/maps/place/${item.properties.address}`"
                target="_blank" title="Google Map">
                {{ item.properties.address }}</a>
              </p>
            </a>
            <a class="list-group-item text-left" :key="key"
              v-else-if="item.properties.county === select.city
                && item.properties.town === select.area
                && inlineRadioOptions === '5'
                && getDistance(item.geometry.coordinates[0],
                item.geometry.coordinates[1], 121.55, 25.03) < 5"
              :class="{ 'highlight': item.properties.mask_adult || item.properties.mask_child}"
              @click="penTo(item)">
              <h3>{{ item.properties.name }}</h3>
              <p class="mb-0">
                Adult: {{ item.properties.mask_adult}}  |  Child: {{ item.properties.mask_child}}
              </p>
              <p class="mb-0">
                 Distance：{{
                  getDistance(item.geometry.coordinates[0],
                  item.geometry.coordinates[1], 121.55, 25.03)}}
                  Km
              </p>
              <p class="mb-0">Address: <a :href="`https://www.google.com.tw/maps/place/${item.properties.address}`"
                target="_blank" title="Google Map">
                {{ item.properties.address }}</a>
              </p>
            </a>
          </template>
        </ul>
      </div>
    </div>
    <div class="col-sm-9">
      <div id="map"></div>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import L from 'leaflet';
import cityName from './assets/cityName.json';

let osmMap = {};

const iconsConfig = {
  iconSize: [25, 41],
  iconAnchor: [12, 41],
  popupAnchor: [1, -34],
  shadowSize: [41, 41],
};
const icons = {
  green: new L.Icon({
    iconUrl: 'https://cdn.rawgit.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-green.png',
    ...iconsConfig,
  }),
  grey: new L.Icon({
    iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-grey.png',
    ...iconsConfig,
  }),
};

const osm = {
  rad(d) {
    return d * (Math.PI / 180.0);
  },
  getDistance(lat1, lng1, lat2, lng2) {
    const radLat1 = this.rad(lat1);
    const radLat2 = this.rad(lat2);
    const a = radLat1 - radLat2;
    const b = this.rad(lng1) - this.rad(lng2);
    let s = 2 * Math.asin(Math.sqrt(Math.sin(a / 2) * Math.sin(a / 2)
    + Math.cos(radLat1) * Math.cos(radLat2) * Math.sin(b / 2) * Math.sin(b / 2)));
    s *= 6378.137; //  地球半径
    s = Math.round(s * 10000) / 10000; // 输出为公里
    return s;
  },
  addMapMarker(x, y, item) {
    const icon = item.mask_adult || item.mask_child ? icons.green : icons.grey;
    L.marker([y, x], {
      icon,
    }).addTo(osmMap).bindPopup(`<strong>${item.name}</strong> <br>
    Mask left：<strong>Adult - ${item.mask_adult ? `${item.mask_adult}` : 'access error'}/ Child - ${item.mask_child ? `${item.mask_child}` : 'access error'}</strong><br>
    Address: <a href="https://www.google.com.tw/maps/place/${item.address}" target="_blank">${item.address}</a><br>
    Tele: ${item.phone}<br>
    Distance：${this.getDistance(x, y, 121.55, 25.03)}Km<br>
    <small>Last updated: ${item.updated}</small>`);
  },
  removeMapMarker() {
    osmMap.eachLayer((layer) => {
      if (layer instanceof L.Marker) {
        osmMap.removeLayer(layer);
      }
    });
  },
  penTo(x, y, item) {
    const icon = item.mask_adult || item.mask_child ? icons.green : icons.grey;
    osmMap.panTo(new L.LatLng(y, x));
    L.marker([y, x], {
      icon,
    }).addTo(osmMap).bindPopup(`<strong>${item.name}</strong><br>
    Mask left：<strong>Adult - ${item.mask_adult ? `${item.mask_adult}` : 'access error'}/ Child - ${item.mask_child ? `${item.mask_child}` : 'access error'}</strong><br>
    Address: <a href="https://www.google.com.tw/maps/place/${item.address}" target="_blank">${item.address}</a><br>
    Tele: ${item.phone}<br>
    Distance：${this.getDistance(x, y, 121.55, 25.03)}Km<br>
    Note: ${item.note}
    <br>
    <small>Last updated: ${item.updated}</small>`).openPopup();
  },
};

export default {
  name: 'home',
  data: () => ({
    cityName,
    data: {},
    osmMap: {},
    select: {
      city: '臺北市',
      area: '大安區',
    },
    inlineRadioOptions: '1000',
  }),
  methods: {
    rad(d) {
      return d * (Math.PI / 180.0);
    },
    getDistance(lat1, lng1, lat2, lng2) {
      const radLat1 = this.rad(lat1);
      const radLat2 = this.rad(lat2);
      const a = radLat1 - radLat2;
      const b = this.rad(lng1) - this.rad(lng2);
      let s = 2 * Math.asin(Math.sqrt(Math.sin(a / 2) * Math.sin(a / 2)
      + Math.cos(radLat1) * Math.cos(radLat2) * Math.sin(b / 2) * Math.sin(b / 2)));
      s *= 6378.137; //  地球半径
      s = Math.round(s * 10000) / 10000; // 输出为公里
      return s;
    },
    updateMarker() {
      this.removeMarker();
      const pharmacies = this.data.filter((pharmacy) => {
        if (!this.select.area) {
          return pharmacy.properties.county === this.select.city
          && this.getDistance(121.55, 25.03, pharmacy.geometry.coordinates[0],
            pharmacy.geometry.coordinates[1]) < this.inlineRadioOptions;
        }
        return pharmacy.properties.town === this.select.area
          && this.getDistance(121.55, 25.03, pharmacy.geometry.coordinates[0],
            pharmacy.geometry.coordinates[1]) < this.inlineRadioOptions;
      });
      pharmacies.forEach((pharmacy) => {
        const { properties, geometry } = pharmacy;
        osm.addMapMarker(
          geometry.coordinates[0],
          geometry.coordinates[1],
          properties,
        );
      });
    },
    removeMarker() {
      osm.removeMapMarker();
    },
    penTo(pharmacy) {
      const { properties, geometry } = pharmacy;
      osm.penTo(geometry.coordinates[0], geometry.coordinates[1], properties);
    },
    updateSelect() {
      this.removeMarker();
      this.updateMarker();
      const pharmacy = this.data.find(item => item.properties.town === this.select.area);
      const { geometry, properties } = pharmacy;
      osm.penTo(geometry.coordinates[0], geometry.coordinates[1], properties);
    },
  },
  mounted() {
    // OSM
    osmMap = L.map('map', {
      center: [25.03, 121.55],
      zoom: 15,
    });

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '<a target="_blank" href="https://www.openstreetmap.org/">© OpenStreetMap 貢獻者</a>',
      maxZoom: 18,
    }).addTo(osmMap);

    this.$http.get('https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json')
      .then((res) => {
        this.data = res.data.features;
        this.updateMarker();
      });
  },
};
</script>

<style lang="scss">
@import 'bootstrap/scss/bootstrap';

#map {
  height: 100vh;
}

.home {
  position: relative;
}

.highlight {
  background: #e9ffe3;
}

.toolbox {
  height: 100vh;
  overflow-y: auto;

  a {
    cursor: pointer;
  }
}
</style>
