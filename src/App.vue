<template>
  <div v-show="!serverLocatKnowen">

    <div class="flex justify-center">

      <div>

        <h1>What is the location of your server?</h1>
        <input type="number" placeholder="Lat" class="bg-transparent border-b border-b-black placeholder:text-gray-300 m-2 p-2 rounded focus:outline-none" v-model="serverLocat.lat">
        <input type="number" placeholder="Lon" class="bg-transparent border-b border-b-black placeholder:text-gray-300 m-2 p-2 rounded focus:outline-none" v-model="serverLocat.lon">

        <button class="block m-auto mt-4 bg-gradient-to-br from-pink-400 via-purple-400 to-indigo-500 p-2 rounded transition-all ease-linear hover:scale-110 duration-150" @click="continuebtn">Continue</button>

      </div>

    </div>

  </div>
  <div v-show="serverLocatKnowen">

    <div class="flex justify-center">

      <div>

        <h1>Server map</h1>
        <span>Select a CDN</span>
        <select class="bg-slate-800 text-white p-2 m-2 rounded cursor-pointer" v-model="cuurentCdn" @change="updateCdn">
          <option value="none">None</option>
          <option v-for="cdn in allCdnNames" :key="cdn" :value="cdn">{{ cdn }}</option>
        </select>

        
      </div>
      
    </div>
    <div class="flex justify-center">

      <div v-if="cuurentCdn != 'none'">
    
        <p>Note: {{ cdnInfo[cuurentCdn].info }}</p>
        
      </div>
      <div v-else>

        <p>Note: You are not using a CDN</p>

      </div>

    </div>
    
  </div>
  <div id="map" class="mt-3"></div>
</template>

<script>
import cdnData from "./assets/cdns.json"

export default {

  data() {

    return {

      serverLocatKnowen: false,
      serverLocat: {
        lat: 0,
        lon: 0,
      },
      allCdnNames: null,
      cuurentCdn: "none",
      cdnMarkers: [],
      map: null,
      cdnInfo: cdnData

    }

  },

  methods: {

    continuebtn() {

      this.serverLocatKnowen = true;

      let map = L.map("map").setView([this.serverLocat.lat, this.serverLocat.lon], 2);
      this.map = map;
      L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
          maxZoom: 19,
          attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
      }).addTo(map);

      let marker = L.marker([this.serverLocat.lat, this.serverLocat.lon]).addTo(map);

      marker.bindPopup("Your server").openPopup();

    },

    updateCdn() {

      if (this.cuurentCdn == "none") {

        for (let i = 0; i < this.cdnMarkers.length; i++) {

          let m = this.cdnMarkers[i];

          m.remove();

        }

      } else {

        for (let i = 0; i < this.cdnMarkers.length; i++) {

          let m = this.cdnMarkers[i];

          m.remove();

        }

        let cdnsPlaces = cdnData[this.cuurentCdn].cdns;

        for (let i = 0; i < cdnsPlaces.length; i++) {

          let d = cdnsPlaces[i]

          let m = L.circle([d.lat, d.lon], {

            color: cdnData[this.cuurentCdn].color,
            fillColor: cdnData[this.cuurentCdn].fillColor,
            fillOpacity: 0.5,
            radius: 50000

          }).addTo(this.map).bindPopup(`${this.cuurentCdn} - ${d.name}`)

          this.cdnMarkers.push(m)

        }

      }

    }
    
  },

  mounted() {

    this.allCdnNames = Object.keys(cdnData);

  }
  
}

</script>
