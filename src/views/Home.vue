<template>
  <div class="home" id="app">
    <input v-model="startpoint" placeholder="Start"/>
    <input v-model="endpoint" placeholder="Ziel"/>
    <button v-on:click="planroute">Route planen</button>
    <button v-on:click="getPosition">aktuelle Position</button>
    <l-map :zoom=13 :center="center" ref="map" @leaflet:load="insertPolyline">
      <l-tile-layer :url="url"></l-tile-layer>
      <l-polyline :lat-lngs="polyline.latlngs" :color="polyline.color"></l-polyline>
      <!-- <l-marker :lat-lng="marker"></l-marker> -->
    </l-map>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import HelloWorld from '@/components/HelloWorld.vue'; // @ is an alias to /src
// @ts-ignore
import OrsDirections from 'openrouteservice-js/src/OrsDirections';
import axios from 'axios';
import { LMap, LTileLayer, LMarker, LPopup, LTooltip, LImageOverlay, LPolyline} from 'vue2-leaflet';
import 'leaflet/dist/leaflet.css'
//import * as polyutil from 'polyline-encoded';


@Component({
  components: {
    LMap,
    LTileLayer,
    LMarker, 
    LImageOverlay,
    LPolyline,
    LPopup,
    LTooltip,
    },
})

export default class Home extends Vue {
  startpoint='';
  endpoint='';
  startpos='';
  currentPosLong: number|undefined;
  currentPosLat: number|undefined;
  latlngPos='';
  polylinearray: []|undefined;
  public pos:any;
  //map='';
  //marker='';
  lat: number|undefined;
  lng: number|undefined;
  planroute() {
    this.getDircetions(this.startpoint, this.endpoint);
    /* var map = this.$refs.map.mapObject;
    var polyline = L.polyline(this.polylinearray,{
      color:'blue',
      weight: 6,
      opacity: 0.9
    }).addTo(map);
    map.fitBounds(polyline.getBounds()); */
  }
  
  insertPolyline(){
    /*var map = this.$refs.map.mapObject;
    L.polyline(this.polylinearray,{
      color:'blue'
    }).addTo(map);*/
  }

  async getReverseGeocode(){
    await axios.get('https://api.openrouteservice.org/geocode/reverse?',{
      params:{
        'point.lat': this.currentPosLat,
        'point.lon': this.currentPosLong,
        api_key: '5b3ce3597851110001cf6248284f5c01dda445a9a0406fe843f2ccb5',
      }
    })
    .then((response) => {
      console.log(response);
      this.startpos=response.data.features[0].properties.name + ',' + response.data.features[0].properties.postalcode;
      console.log(this.startpos);
    })
    .catch(function (error:any) {
      console.log(error);
    });
    return this.startpos;
  }


  getPosition(){
    navigator.geolocation.getCurrentPosition(
      async (position)=>{
        this.currentPosLong = position.coords.longitude;
        this.currentPosLat = position.coords.latitude;
        this.latlngPos = this.currentPosLat + ','+ this.currentPosLong;
        console.log(this.currentPosLat);
        console.log(this.currentPosLong); 
        this.startpoint = await this.getReverseGeocode();
        //this.$refs.map.mapObject.setView(this.latlngPos, 13);
      },
      function(error){
        alert(error.message);
      }
    );
  }

 data() {
    return {
      zoom: 13,
      center: [48,9],
      url: 'http://{s}.tile.osm.org/{z}/{x}/{y}.png',
      polyline:{
        //latlngs: await this.getDircetions(this.startpoint,this.endpoint),
        //color: 'green'
      }
      //attribution: '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      //marker: L.latLng(47.413220, -1.219482),
      //markers: []
    }
  }
  mounted () {
    //this.$refs.map.mapObject.setView(navigator.geolocation.getCurrentPosition((position)=>{this.pos = position}), 13);
  }

  //Vue.config.productionTip = false;


async getgeocode(address: string) {
  await axios.get('https://api.openrouteservice.org/geocode/search?', {
      params: {
        text: address,
        api_key: '5b3ce3597851110001cf6248284f5c01dda445a9a0406fe843f2ccb5',
      }
    })
    .then((response) => {
      console.log(response);
      this.lat = response.data.features[0].geometry.coordinates[1];
      this.lng = response.data.features[0].geometry.coordinates[0];
      console.log(this.lat);
      console.log(this.lng);
    })
    .catch(function (error:any) {
      console.log(error);
    });
    return this.lng +','+ this.lat;
  }

async getDircetions(startaddress: string, endaddress: string){
  await axios.get('https://api.openrouteservice.org/v2/directions/cycling-regular?',{
    params:{
      api_key:  '5b3ce3597851110001cf6248284f5c01dda445a9a0406fe843f2ccb5',
      start: await this.getgeocode(startaddress),
      end: await this.getgeocode(endaddress),
    }
  })
  .then((response)=>{
    console.log(response);
    this.polylinearray = response.data.features[0].geometry.coordinates;
    console.log(this.polylinearray);
  })
  return this.polylinearray;
  }
}
</script>
<style lang="scss">
html, body, #app {
  height: 100%
}

</style>

