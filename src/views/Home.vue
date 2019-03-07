<template>
  <div class="home">
    <input v-model="startpoint" placeholder="Start"/>
    <input v-model="endpoint" placeholder="Ziel"/>
    <button v-on:click="planroute">Route planen</button>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import HelloWorld from '@/components/HelloWorld.vue'; // @ is an alias to /src
// @ts-ignore
import OrsDirections from 'openrouteservice-js/src/OrsDirections';
import axios from 'axios';


@Component({
  components: {
    //HelloWorld,
  },
})
export default class Home extends Vue {
  startpoint='';
  endpoint='';
  public lat: number|null=null;
  public lng: number|null=null;
  planroute() {
    this.getDircetions(this.startpoint, this.endpoint);
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
