<style>

#list_inner .swiper-slide {
  width: 100% !important;
  }
#list_inner .swiper-button-prev {
  opacity: 0;
  /* always hidden, since we use the loop: true option ; */
}
#list_inner .swiper-button-prev {
  opacity:  0;
  color: #999
}
#list_inner .swiper-button-next {
  opacity:  0.8;
  color: #999
}
#list_inner .swiper-button-next:after {
  font-size: 25px;
}
#list_inner .swiper-button-prev.swiper-button-disabled,
#list_inner .swiper-button-next.swiper-button-disabled {
  opacity: 0;
}
</style>

<template>
  <div class="">
    <ul class="pb-10">
      <li v-if="places.length == 0" class="bg-a100c-white px-4 py-2 rounded shadow mt-4">
        <svg class="inline" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="20" height="20"><path fill="none" d="M0 0h24v24H0z"/><path d="M14.935 7.204l-6-3L4 6.319v12.648l5.065-2.17 6 3L20 17.68V5.033l-5.065 2.17zM2 5l7-3 6 3 6.303-2.701a.5.5 0 0 1 .697.46V19l-7 3-6-3-6.303 2.701a.5.5 0 0 1-.697-.46V5zm4 6h2v2H6v-2zm4 0h2v2h-2v-2zm5.998-.063L17.236 9.7l1.06 1.06-1.237 1.238 1.237 1.238-1.06 1.06-1.238-1.237-1.237 1.237-1.061-1.06 1.237-1.238-1.237-1.237L14.76 9.7l1.238 1.237z"/></svg>
        Please go left to see
        <nuxt-link :to="{ path: '/main', hash:'map'}" class="text-link">
          our map
        </nuxt-link>
        and select a place or layer to see more details here.
      </li>
      <li v-if="!places" class="bg-a100c-white px-4 py-2 rounded shadow mt-4">
        <svg class="inline" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="20" height="20"><path fill="none" d="M0 0h24v24H0z"/><path d="M15 4H5v16h14V8h-4V4zM3 2.992C3 2.444 3.447 2 3.999 2H16l5 5v13.993A1 1 0 0 1 20.007 22H3.993A1 1 0 0 1 3 21.008V2.992zM11 11h2v6h-2v-6zm0-4h2v2h-2V7z"/></svg> Or check the
        <nuxt-link :to="{ path: '/main', hash:'info'}" class="text-link">
          info page
        </nuxt-link>
        to learn more about this map :)
      </li>
      <li v-else class="bg-a100c-white px-4 py-2 rounded shadow mt-4 ">
        <h2 class="font-semibold pl-0 md:pl-12"><span v-if="data.title != data.layer[parseInt(layerindex)].title">{{data.title}}</span><span v-else><nuxt-link :to="{ path: '/'}">Start</nuxt-link></span> <span v-if="data.layer[parseInt(layerindex)]">— {{ data.layer[parseInt(layerindex)].title}}</span></h2>
      </li>

      <li v-for="(place,index) in places" :id="'list-place-'+place.id" class="bg-a100c-white px-4 py-2 rounded shadow mt-4">
        <div v-swiper:[index]="getSwiperOptions(index)" :ref="`mySwiperRef${index}`" class="md:px-12">
          <div class="swiper-wrapper" v-if="place.images">
            <div v-for="(image,iindex) in place.images" :key="iindex" class="swiper-slide px-0 pb-4 pt-2 sm:px-4 sm:pt-4">
              <p v-if="place.images.length > 1" class="text-sm text-gray max-w-60 text-left">({{iindex+1}}/{{place.images.length}})</p>
              <span v-if="image">
                <img v-bind:src="image.image_url" :alt="image.alt" class="max-w-full sm:max-w-ws max-h-72 sm:max-h-80 lg:max-h-96">
                <span class="text-sm leading-tight text-gray max-w-60">{{image.title}}</span>
              </span>
              <span v-else>
                <img src="https://via.placeholder.com/585x870?text=Platzhalter_585x870px" alt="">
              </span>
            </div>
          </div>
          <div v-if="place.images.length > 1">
            <div tabindex="0" :class="`swiper-button-prev swiper-button-prev${index}`" role="button"></div>
            <div tabindex="1" :class="`swiper-button-next swiper-button-next${index}`" role="button"></div>
          </div>
        </div>
        <h3 class="font-semibold text-lg px-4 py-2 sm:px-16 sm:pt-6">{{ place.title }}</h3>
        <div class="text-gray-500 px-4 sm:px-16 sm:py-3" v-html="place.teaser"></div>
        <div :id="'list-audio-'+place.id" class="player-wrapper px-4 sm:px-16 sm:py-3" v-if="place.audiolink" v-html="place.audiolink">
        </div>
        <ul v-if="place.annotations.length > 0" class="pb-0 sm:px-8">
          <li v-for="(annotation,aindex) in place.annotations" class="bg-a100c-3 px-4 py-6 rounded shadow mt-4 mb-6">
            <h4 v-if="annotation.title" class="font-semibold text-md px-4 py-2">{{ annotation.title }}</h4>
            <div class="text-gray-500 px-4" v-html="annotation.text"></div>
          </li>
        </ul>
        <footer class="pb-4">
          <p class="text-gray-500 px-4 py-2 sm:px-16 sm:py-3">
            <button @click="recenterMap(place.lat,place.lon,index)" class="text-link">Show on the map</button>
          </p>
        </footer>
      </li>
    </ul>
  </div>
</template>


<script>


export default {
  props: {
    places: {
      type: Array,
      required: false
    },
    data: {
      type: Object,
      required: false
    },
    layerindex: {
      type: Number,
      required: false
    },
    map: {
      type: Object,
      required: false
    }
  },
  components: {
  },
  methods: {
    recenterMap(lat,lon,index) {
      this.$nextTick(() => {
        console.log("Recenter map to "+ lat +"/"+lon);
        console.log("Predefined zoom level: "+ this.data.zoom);
        let z = 10;
        if ( this.data.zoom > 10 ) {
          z = this.data.zoom + 3;
          console.log("Set zoom level to: "+ z);
        }
        this.$router.push({ name: 'main', hash: '#map', query: { flyto: "true" } });
        // TODO: carefully adapt zoom level
        this.map.flyTo([lat,lon],z);

      })
    },
    getSwiperOptions(index){
     return {
        slidesPerView: 1,
        spaceBetween: 10,
        pagination: {
          el: '.swiper-pagination'+index,
          clickable: true
        },
        paginationClickable: true,
        navigation: {
          nextEl: ".swiper-button-next"+index,
          prevEl: ".swiper-button-prev"+index
        },
        spaceBetween: 50,
        loop: true
      }
    }
  },
  data() {
    return {
    }
  },
  computed: {

  },
  mounted() {
  }
}
</script>
