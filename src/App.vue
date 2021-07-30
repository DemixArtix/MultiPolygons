<template lang="pug">
  div(id="app")
    LMap(
      class="map"
      :zoom="zoom"
      :center="center"
      )
      LTileLayer(
        :url="url"
        :attribution="attribution"
        )
      LMarker(
        class="marker"
        :lat-lng="center"
        :icon="icon"
      )
        LPopup(content="Уменьшите маштаб чтобы увидеть результат")
      LGeoJson(
        :geojson="multiPolygon"
        v-if="showResult"
      )
      LControl(
        position="topright"
      )
        div(class="side-block")
          button(@click="onGetDistance(multiPolygon)" class="side-block__button") вычислить общую протяженность границы
          button(@click="onShowResult()" class="side-block__button") показать / скрыть результат
          span(class="side-block__text" v-if="totalDistance") протяженность границы {{totalDistance}} метров

</template>

<script>
  //import axios для запросов к апи
  import api from '@/plugins/api'
  //import leaflet для отрисовки карты
  import { latLng, icon } from "leaflet";
  import { LMap, LTileLayer, LMarker, LGeoJson, LPopup, LControl } from 'vue2-leaflet';
  //import data.json
  import data from './data.json'
  //импорт функции для измерения растояния между точками
  import { getDistance } from 'geolib';

  


export default {
  name: 'App',
  components: {
    LMap, LTileLayer, LMarker, LGeoJson, LPopup, LControl
  },
  async mounted() {
    // this.multiPolygon = data;
    //после вставки компонета в DOM отправить запрос на апи для получения данных
    await this.getMultiPolygon()
  },
  data: () => ({

    zoom: 12,
    center: latLng(56.852961355248524,53.21150779724121),
    url: 'https://tile.thunderforest.com/landscape/{z}/{x}/{y}.png?apikey=c5fcd839156e4816925398e1f513e089',
    attribution: '&copy; <a href="https://www.thunderforest.com/maps/landscape/">Landscape</a> contributors',
    multiPolygon: {},
    icon: icon({
           iconUrl: require('./assets/icons/kisspng-computer-icons-google-map-maker-world-map-clip-art-map-marker-5ac30934c412d5.3086608015227313168031.png'),
           iconSize: [50, 50]
    }),
    totalDistance: 0,
    showResult: false,
  }),
  computed: {
  },
  methods: {
    //запрос к api для получения данных(координат полигона) полигона
    /**
     * @async request to api
     * @return multiPolygon = response.data
     */
    async getMultiPolygon() {
      await api.get('/').then(res => {
        console.log(res);
        return this.multiPolygon = res.data;
      }).catch(err => {
        console.dir(err);
        //cors политика не пропускает ответ на локальном сервере
        return this.multiPolygon = data;
      })
    },
    //функция для рассчета расстояния меджу всеми точками
    /**
     *@param {object} data - объект фотмата geoJson
     * @return {number}
     */
    onGetDistance(data) {

      const coordinatesArr = data.features[0].geometry.coordinates;
      this.totalDistance = 0;

      coordinatesArr.map(([item], index) => {

        let params = item.map(([ latitude, longitude ]) => {
          return { latitude, longitude }
        });
        let polygonDistance = 0;

        for (let i = 0; i < params.length - 1; i++) {
          polygonDistance += getDistance(params[i], params[i + 1]);
        }

        this.totalDistance += polygonDistance;
        console.log('polygon distance:', polygonDistance, 'meters,', 'polygon index:' , index);
      });
      console.log('total distance:', this.totalDistance, 'meters');
      return this.totalDistance
    },
    //отобразить мульти полигон на карте
    onShowResult() {
      if(this.showResult)  {
        this.center = latLng(56.852961355248524, 53.21150779724121);

      } else {
        this.zoom = 3;
        this.center = latLng(63.31268278043484, 99.140625);
      }
      this.showResult = !this.showResult
    }
  }
}
</script>

<style lang="sass">
#app
  .map
    width: 100%
    height: 100vh
  .marker

  .side-block
    background-color: #fff
    padding: 10px
    border-radius: 5px
    display: flex
    flex-direction: column
    & > *
      padding: 5px
      margin: 5px
      font-size: 16px
    .side-block__button
      background-color: #000
      border: none
      border-radius: 5px
      padding: 10px
      color: #fff
    .side-block__text


</style>
