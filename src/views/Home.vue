<template>
  <div class="map">
    <div ref="mapContent" class="mapContent" @mousedown="mapStartInteractive($event)" @mousemove="mapInteractive($event)" @mouseup="mapStopInteractive($event)" @dblclick="setMapZoom($event)" @mouseleave="mapStopInteractive($event)">
      <!-- <div class="country" title="Россия">
      </div>
      <div class="country" title="Америка">
      </div>
      <div class="country" title="Китай">
      </div> -->
      <div v-for="country in countries" :key="country.name" class="country" :title="country.name" :style="`background-image: url('${country.thumbnail}'); background-size: 100% 100%;`">
      </div>
      <div ref="myself" class="myself" title="Вы">
        <span>
          Вы
        </span>
      </div>
    </div>  
  </div>
</template>

<script>
// import HelloWorld from '@/components/HelloWorld.vue'

const openGeocoder = require('node-open-geocoder')

export default {
  name: 'Home',
  data() {
    return {
      isInteractive: false,
      zoomDelta: 0.5,
      countries: [
        {
          name: 'Россия',
          thumbnail: 'data:image/png;base64,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'
        },
        {
          name: 'Китай',
          thumbnail: 'https://upload.wikimedia.org/wikipedia/commons/thumb/f/fa/Flag_of_the_People%27s_Republic_of_China.svg/1200px-Flag_of_the_People%27s_Republic_of_China.svg.png'
        },
        {
          name: 'Америка',
          thumbnail: 'https://gtmarket.ru/files/flags/Flag_of_United_States.png'
        }
      ]
    }
  },
  mounted () {
    window.addEventListener('scroll', (event) => {
      console.log(`scroll: ${event}`)
    })
    // document.querySelectorAll('.country').forEach(country => {
    //   let countryPosition = {
    //     x: 0,
    //     y: 0
    //   }
    //   let contryPositionX = Math.floor(Math.random() * 3500)
    //   let contryPositionY = Math.floor(Math.random() * 3500)
    //   countryPosition.x = contryPositionX
    //   countryPosition.y = contryPositionY
    //   country.style = `
    //     border-radius: 100%;
    //     width: 35px;
    //     height: 35px;
    //     background-color: rgb(255, 0, 0);
    //     position: relative;
    //     top: ${countryPosition.y}px;
    //     left: ${countryPosition.x}px;
    //   `
    // })
    document.querySelectorAll('.country').forEach((country, countryIdx) => {
      let countryPosition = {
        x: 0,
        y: 0
      }
      let contryPositionX = Math.floor(Math.random() * 3500)
      let contryPositionY = Math.floor(Math.random() * 3500)
      countryPosition.x = contryPositionX
      countryPosition.y = contryPositionY
      country.style = `
        border-radius: 100%;
        width: 35px;
        height: 35px;
        background-color: rgb(0, 150, 0);
        position: relative;
        top: ${countryPosition.y}px;
        left: ${countryPosition.x}px;
        background-image: url('${this.countries[countryIdx].thumbnail}');
        background-size: 100% 100%;
      `
    })
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition((position) => {
        let lat = position.coords.latitude
        let lng = position.coords.longitude
        console.log(`lat: ${lat}; lng: ${lng}`)
        let decodedLat = Math.floor(lat)
        let decodedLng = Math.floor(lng)
        
        let previousZoom = this.$refs.mapContent.style.zoom
        if(!previousZoom.length) {
          previousZoom = 1.0
        }
        let digitalPreviousZoom = Number(previousZoom)
        let newZoom = digitalPreviousZoom + this.zoomDelta
        
        let previousCoord = this.$refs.mapContent.getBoundingClientRect()
        let previousCoordY = previousCoord.top
        let previousCoordX = previousCoord.left
        this.$refs.mapContent.style = `
          width: calc(100% + 400%);
          height: calc(100% + 400%);
          background-color: rgb(100, 25, 225);
          position: relative;
          top: ${previousCoordY}px;
          left: ${previousCoordX}px;
          cursor: crosshair;
          zoom: ${newZoom};
        `
        // top: ${previousCoordY + decodedLng}px;
        // left: ${previousCoordX + decodedLat}px;
        this.$refs.myself.style = `
          display: flex;
          align-items: center;
          justify-content: center;
          font-weight: bolder;
          color: rgb(255, 255, 255);
          border-radius: 100%;
          width: 35px;
          height: 35px;
          background-color: rgb(255, 0, 0);
          border: 1px solid rgb(0, 0, 0);
          position: relative;
          top: ${previousCoordY}px;
          left: ${previousCoordX}px;
        `
        // top: ${previousCoordY + decodedLng}px;
        // left: ${previousCoordX + decodedLat}px;

        let destination = 'Неудается получить местоположение'
        openGeocoder()
          .reverse(lng, lat)
            .end((err, res) => {
              destination = `${res.address.city}, ${res.address.state}, ${res.address.country}`
              alert(`Сейчас вы находитесь: ${destination}`)
            })

      })
    }
  },
  methods: {
    mapStartInteractive(event) {
      this.isInteractive = true
    },
    mapInteractive(event) {
      if (this.isInteractive) {
        let previousCoord = this.$refs.mapContent.getBoundingClientRect()
        let previousCoordY = previousCoord.top
        let previousCoordX = previousCoord.left
        let previousZoom = this.$refs.mapContent.style.zoom
        if (!previousZoom.length) {
          previousZoom = 1.0
        }
        this.$refs.mapContent.style = `
          width: calc(100% + 400%);
          height: calc(100% + 400%);
          background-color: rgb(100, 25, 225);
          position: relative;
          top: calc(${previousCoordY}px + ${event.movementY}px);
          left: calc(${previousCoordX}px + ${event.movementX}px);
          cursor: crosshair;
          zoom: ${previousZoom};
        `
      }
    },
    mapStopInteractive(event) {
      this.isInteractive = false
    },
    setMapZoom(event) {
      let previousZoom = this.$refs.mapContent.style.zoom
      if(!previousZoom.length) {
        previousZoom = 1.0
      }
      let digitalPreviousZoom = Number(previousZoom)
      let newZoom = digitalPreviousZoom + this.zoomDelta
      
      let previousCoord = this.$refs.mapContent.getBoundingClientRect()
      let previousCoordY = previousCoord.top
      let previousCoordX = previousCoord.left
      this.$refs.mapContent.style = `
        width: calc(100% + 400%);
        height: calc(100% + 400%);
        background-color: rgb(100, 25, 225);
        position: relative;
        top: ${previousCoordY}px;
        left: ${previousCoordX}px;
        cursor: crosshair;
        zoom: ${newZoom};
      `
    }
  },
  components: {
    
  }
}
</script>
<style scoped>
  
  .map {
    width: 100%;
    height: 100%;
    background-color: rgb(200, 240, 240);
    position: fixed;
    top: 0px;
    left: 0px;
  }

  .mapContent {
    width: calc(100% + 400%);
    height: calc(100% + 400%);
    background-color: rgb(100, 25, 225);
    position: relative;
    /* top: calc(500% / 2 * -1); */
    top: 50%;
    /* left: calc(500% / 2 * -1); */
    left: 50%;
    cursor: crosshair;
    zoom: 1.0;
  }

  .country {
    border-radius: 100%;
    width: 35px;
    height: 35px;
    background-color: rgb(0, 150, 0);
    position: relative;
    top: 0px;
    left: 0px;
  }

  .myself {
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: bolder;
    color: rgb(255, 255, 255);
    border-radius: 100%;
    width: 35px;
    height: 35px;
    background-color: rgb(255, 0, 0);
    border: 1px solid rgb(0, 0, 0);
    position: relative;
    top: 0px;
    left: 0px;
  }

</style>