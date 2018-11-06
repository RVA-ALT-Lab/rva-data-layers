<template>
    <div>
      <b-container fluid>
        <b-row>
        <b-col>
          <h3>Select Data Layers</h3>
          <p>Mix and match a bunch of different data layers to produce your own unique perspective on the Richmond metro area.</p>
          <div role="tablist">
            <b-card no-body class="mb-1">
              <b-card-header header-tag="header" class="p-1" role="tab">
                <b-btn block href="#" v-b-toggle.accordion1 variant="info">Geographical Boundaries</b-btn>
              </b-card-header>
              <b-collapse id="accordion1" visible accordion="my-accordion" role="tabpanel">
                <b-card-body>
                  <input type="radio" @change="updateGeoJson" v-model="features" value="zipcode"  id="geojson-zipcode"><label for="geojson-zipcode">Zipcode</label>
                  <input type="radio" @change="updateGeoJson" v-model="features" value="congress"  id="geojson-congress"><label for="geojson-congress">Congressional Districts</label>
                  <input type="radio" @change="updateGeoJson" v-model="features" value="neighborhood"  id="geojson-neighborhood"><label for="geojson-neighborhood">Neighborhood</label>
                  <input type="radio" @change="updateGeoJson" v-model="features" value="council"  id="geojson-council"><label for="geojson-council">Council</label>
                </b-card-body>
              </b-collapse>
            </b-card>
            <b-card no-body class="mb-1">
              <b-card-header header-tag="header" class="p-1" role="tab">
                <b-btn block href="#" v-b-toggle.accordion2 variant="info">Geographical Analysis</b-btn>
              </b-card-header>
              <b-collapse id="accordion2" accordion="my-accordion" role="tabpanel">
                <b-card-body>
                  <p class="card-text">Geographical analysis layers usually combine on of the above geographical boundaries with a statistical variable, like average income level by census tract, or voter turnout by congressional district. Please note that we are not able to analyze all data sets at every geographical level, so selecting on these options may override your choice of geographical boundary.</p>
                  <input type="radio" @change="updateGeoJson" v-model="features" value="zipcode"  id="geojson-zipcode"><label for="geojson-zipcode">Zipcode</label>
                  <input type="radio" @change="updateGeoJson" v-model="features" value="congress"  id="geojson-congress"><label for="geojson-congress">Congressional Districts</label>
                  <input type="radio" @change="updateGeoJson" v-model="features" value="neighborhood"  id="geojson-neighborhood"><label for="geojson-neighborhood">Neighborhood</label>
                  <input type="radio" @change="updateGeoJson" v-model="features" value="council"  id="geojson-council"><label for="geojson-council">Council</label>
                </b-card-body>
              </b-collapse>
            </b-card>
            <b-card no-body class="mb-1">
              <b-card-header header-tag="header" class="p-1" role="tab">
                <b-btn block href="#" v-b-toggle.accordion3 variant="info">Point Features</b-btn>
              </b-card-header>
              <b-collapse id="accordion3" accordion="my-accordion" role="tabpanel">
                <b-card-body>
                  <p class="card-text">
                    Point features are things that exist at a specific latitude and longitude, think schools, fire stations, or sites of car accidents.</p><p class="card-text"> By looking at point features superimposed on top of statistical surfaces, we can answer questions like "Do more car accidents happen in neighborhoods with higher than average income levels?"
                  </p>
                </b-card-body>
              </b-collapse>
            </b-card>
          </div>

          <b-btn type="button" :block="true" variant="outline-primary">Export these data layers</b-btn>
        </b-col>
        <b-col>
          <l-map :zoom="zoom" :center="center" style="height: 800px;">
            <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
            <l-marker :lat-lng="marker"></l-marker>
            <l-geo-json :geojson="geoJson" :options="options"></l-geo-json>
        </l-map>
        </b-col>
        </b-row>
      </b-container>
    </div>
</template>
<script>
import 'leaflet/dist/leaflet.css'
import 'leaflet-defaulticon-compatibility/dist/leaflet-defaulticon-compatibility.webpack.css'
import {LMap, LTileLayer, LMarker, LGeoJson} from 'vue2-leaflet'
import 'leaflet-defaulticon-compatibility'
import * as L from 'leaflet'
import axios from 'axios'

export default {
  name: 'Map',
  data () {
    return {
      zoom: 12,
      center: L.latLng(37.531399, -77.476009),
      url: 'https://api.mapbox.com/styles/v1/jeffeverhart383/cj9sxi40c2g3s2skby2y6h8jh/tiles/256/{z}/{x}/{y}?access_token=pk.eyJ1IjoiamVmZmV2ZXJoYXJ0MzgzIiwiYSI6IjIwNzVlOTA3ODI2MTY0MjM3OTgxMTJlODgzNjg5MzM4In0.QA1GsfWZccIB8u0FbhJmRg',
      attribution: 'Map data &copy; <a href="http://openstreetmap.org">OpenStreetMap</a> contributors, <a href="http://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="http://mapbox.com">Mapbox</a>',
      marker: L.latLng(37.531399, -77.476009),
      features: 'zipcode',
      enableTooltip: true,
      urls: {
        zipcode: 'https://raw.githubusercontent.com/RVA-ALT-Lab/Richmond-GIS/master/Thematic/ZipCode.shp.geojson',
        congress: 'https://raw.githubusercontent.com/RVA-ALT-Lab/Richmond-GIS/master/Thematic/CongressDistrict.shp.geojson',
        neighborhood: 'https://raw.githubusercontent.com/RVA-ALT-Lab/Richmond-GIS/master/Thematic/Neighborhood.shp.geojson',
        council: 'https://raw.githubusercontent.com/RVA-ALT-Lab/Richmond-GIS/master/Thematic/CouncilDistrict.shp.geojson'
      },
      geoJson: ''
    }
  },
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LGeoJson
  },
  computed: {
    options () {
      return {
        onEachFeature: this.onEachFeatureFunction
      }
    },
    onEachFeatureFunction () {
      return (feature, layer) => {
        layer.bindTooltip('<div>ID:' + feature.properties.ID + '</div><div>Name: ' + feature.properties.Name + '</div>', { permanent: false, sticky: true })
      }
    }
  },
  created: function () {
    axios.get('https://raw.githubusercontent.com/RVA-ALT-Lab/Richmond-GIS/master/Thematic/ZipCode.shp.geojson')
      .then(response => {
        this.geoJson = response.data
      })
  },
  methods: {
    updateGeoJson () {
      const url = this.urls[this.features]
      axios.get(url)
        .then(response => {
          this.geoJson = response.data
        })
    }
  }

}
</script>
<style>

</style>
