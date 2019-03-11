<template>
  <div class="home" id="app">
    <input v-model="startpoint" placeholder="Start"/>
    <input v-model="endpoint" placeholder="Ziel"/>
    <button v-on:click="planroute">Route planen</button>
    <button v-on:click="getPosition">aktuelle Position</button>
    <l-map :zoom=13 :center="center">
      <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
      <l-marker :lat-lng="marker"></l-marker>
    </l-map>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import HelloWorld from '@/components/HelloWorld.vue'; // @ is an alias to /src
// @ts-ignore
import OrsDirections from 'openrouteservice-js/src/OrsDirections';
import axios from 'axios';
import { LMap, LTileLayer, LMarker, LPopup, LTooltip, L } from 'vue2-leaflet';
import 'leaflet/dist/leaflet.css'


@Component({
  components: {
    LMap,
    LTileLayer,
    LMarker,
  },
})
export default class Home extends Vue {
  startpoint='';
  endpoint='';
  startpos='';
  public currentPosLong: number|null=null;
  public currentPosLat: number|null=null;
  latlngPos='';
  //map='';
  //marker='';
  public lat: number|null=null;
  public lng: number|null=null;
  planroute() {
    this.getDircetions(this.startpoint, this.endpoint);
  }
  
  async getReverseGeocode(){
    await axios.get('https://maps.googleapis.com/maps/api/geocode/json?',{
      params:{
        latlng: this.latlngPos,
        key: 'AIzaSyC_8ddUtSwBjXjfm6dNTi4O8owp0y14Hy8',
      }
    })
    .then((response) => {
      console.log(response);
      this.startpos=response.data.results[0].formatted_address;
    })
    .catch(function (error:any) {
      console.log(error);
    });
  }


  getPosition(){
    navigator.geolocation.getCurrentPosition(
      async (position)=>{
        this.currentPosLong = position.coords.longitude;
        this.currentPosLat = position.coords.latitude;
        this.latlngPos = this.currentPosLat + ',' + this.currentPosLong;
        console.log(this.currentPosLat);
        console.log(this.currentPosLong); 
        await this.getReverseGeocode();
      },
      function(error){
        alert(error.message);
      }
    );

  }

 data() {
    return {
      zoom: 13,
      center: L.latLng(47.413220, -1.219482),
      url: 'http://{s}.tile.osm.org/{z}/{x}/{y}.png',
      attribution: '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      marker: L.latLng(47.413220, -1.219482),
      //markers: []
    }
  }
  
  //Vue.config.productionTip = false;


async getgeocode(address: string) {
  await axios.get('https://maps.googleapis.com/maps/api/geocode/json?', {
      params: {
        address: {address},
        key: 'AIzaSyC_8ddUtSwBjXjfm6dNTi4O8owp0y14Hy8',
      }
    })
    .then((response) => {
      this.lat = response.data.results[0].geometry.location.lat;
      this.lng = response.data.results[0].geometry.location.lng;
      console.log(this.lat);
      console.log(this.lng);
    })
    .catch(function (error:any) {
      console.log(error);
    });
    return [this.lng, this.lat];
  }

async getDircetions(startaddress: string, endaddress: string){
    const Directions = new OrsDirections({
      api_key: '5b3ce3597851110001cf6248284f5c01dda445a9a0406fe843f2ccb5',
    });
    Directions.calculate({
        coordinates: [await this.getgeocode(startaddress), await this.getgeocode(endaddress)],
        profile: 'cycling-regular',
      })
      .then(function(json: JSON){
        console.log(JSON.stringify(json));
      })
      .catch(function(err: any){
        console.log('An error occured: ' + err);
      });
  }
}
</script>
<style lang="scss">
html, body, #app {
  height: 100%
}

</style>

