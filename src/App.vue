<template>
  <div v-if="!serverLocatKnowen">

    <div class="flex justify-center">

      <div>

        <h1>What is the location of your server?</h1>
        <div><span>Server latitude:</span><input type="number" placeholder="Lat" class="bg-transparent border-b border-b-black placeholder:text-gray-300 m-2 p-2 rounded focus:outline-none" v-model="serverLocat.lat"></div>
        <div><span>Server longitude:</span><input type="number" placeholder="Lon" class="bg-transparent border-b border-b-black placeholder:text-gray-300 m-2 p-2 rounded focus:outline-none" v-model="serverLocat.lon"></div>

        <button class="block m-auto mt-4 bg-gradient-to-br from-pink-400 via-purple-400 to-indigo-500 p-2 rounded transition-all ease-linear hover:scale-110 duration-150 shadow" @click="continuebtn">Continue</button>

        <button class="block m-auto mt-8 bg-slate-900 p-2 rounded cursor-pointer transition-all ease-in hover:scale-110 duration-150 shadow" @click="skipbtn">Skip</button>

      </div>

    </div>

  </div>
  <div v-else>

    <div class="flex justify-center">

      <div class="relative">

        <h1 class="font-mono m-2">Server map</h1>
        <span class="m-2">Select a CDN</span>
        <select class="bg-slate-800 text-white p-2 rounded cursor-pointer w-fit h-8 hover:h-48 focus:h-48 focus:shadow ring-slate-500 focus:ring absolute transition-all duration-75 z-40 top-[5ch]" v-model="cuurentCdn" @change="updateCdn" multiple>
          <option value="none">None</option>
          <option v-for="cdn in allCdnNames" :key="cdn" :value="cdn">{{ cdn }}</option>
        </select>
        <div class="m-2"></div>
        <div v-if="!addedyourlocaiton && !erroraddinglocaiton">
          <button class="block bg-gradient-to-r from-cyan-300 via-sky-400 to-blue-500 p-2 m-auto rounded mt-2 mb-2 shadow transition-all ease-in hover:scale-110 duration-150" @click="addyourlocationbtn">Add your location</button>
        </div>
        <div v-else-if="erroraddinglocaiton">
          <div class="m-auto bg-red-500 text-white font-bold w-fit p-2 rounded shadow">

            <span>Error adding your location!</span>

          </div>
        </div>

        
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
  <div id="map" class="mt-3 z-20"></div>
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
      cuurentCdn: ["none"],
      cdnMarkers: [],
      map: null,
      cdnInfo: cdnData,
      addedyourlocaiton: false,
      erroraddinglocaiton: false

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
  
        for (let j = 0; j < this.cuurentCdn.length; j++) {
          
          let cdnsPlaces = cdnData[this.cuurentCdn[j]].cdns;

          for (let i = 0; i < cdnsPlaces.length; i++) {

            let d = cdnsPlaces[i]

            let m = L.circle([d.lat, d.lon], {

              color: cdnData[this.cuurentCdn[j]].color,
              fillColor: cdnData[this.cuurentCdn[j]].fillColor,
              fillOpacity: 0.5,
              radius: 50000

            }).addTo(this.map).bindPopup(`${this.cuurentCdn[j]} - ${d.name}`)

            this.cdnMarkers.push(m)

          }

        }

      }

    },

    skipbtn() {

      this.serverLocatKnowen = true;

      let map = L.map("map").setView([this.serverLocat.lat, this.serverLocat.lon], 2);
      this.map = map;
      L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
          maxZoom: 19,
          attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
      }).addTo(map);


    },

    addyourlocationbtn() {

      
      if ("geolocation" in navigator) {
        
        navigator.geolocation.getCurrentPosition((pos) => {
          
          const lat = pos.coords.latitude;
          const lon = pos.coords.longitude;
          
          let marker = L.marker([lat, lon]).addTo(this.map)
          
          marker.bindPopup("Your location")

          let circle = L.circle([lat, lon], {

            color: "#312e81",
            fillColor: "#4338ca",
            fillOpacity: 0.5,
            radius: pos.coords.accuracy

          }).addTo(this.map)

          circle.bindPopup("Your locaiton")

          this.map.setView([lat, lon], 13);

          this.addedyourlocaiton = true;
          
        }, () => this.erroraddinglocaiton = true)

      } else {

        this.erroraddinglocaiton = true;

      }

    }
    
  },

  mounted() {

    this.allCdnNames = Object.keys(cdnData);

  }
  
}

</script>
