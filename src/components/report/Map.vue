<template>
  <div class="com-container">
    <div id="container">
      <div id="amap-container" />
      <div id="map-imgs">
        <el-carousel :interval="4000" type="card" height="100px" indicator-position="none">
          <el-carousel-item v-for="item in mapImgs" :key="item.url">
            <img class="el-carousel-item-img" :src="item.url" alt="" />
          </el-carousel-item>
        </el-carousel>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import AMapLoader from '@amap/amap-jsapi-loader'

export default {
  name: 'Map',
  data() {
    return {
      AMap: null,
      amap: null,
      markersCache: [],
      mapImgs: [
        { url: './static/images/1.png' },
        { url: './static/images/2.png' },
        { url: './static/images/3.jpeg' },
        { url: './static/images/4.jpeg' },
        { url: './static/images/5.jpeg' },
        { url: './static/images/6.png' },
      ],
    }
  },
  computed: {
    ...mapState(['theme', 'markers']),
    markers1() {
      return this.markers
    },
  },
  watch: {
    theme() {},
    markers1(newData, oldData) {
      let time = 1500
      if (this.markersCache.length > 0) {
        this.amap.remove(this.markersCache)
        this.markersCache = []
        time = 0
      }
      setTimeout(() => {
        let markerArr = newData.map(item => {
          return new this.AMap.Marker({
            position: new this.AMap.LngLat(item.lat, item.lng),
            title: item.title,
          })
        })
        this.markersCache = markerArr
        this.amap.add(markerArr)
      }, time)
    },
  },
  created() {},
  mounted() {
    this.initMap()
  },
  destroyed() {},
  methods: {
    // 初始化高德地图
    initMap() {
      AMapLoader.load({
        key: 'a5c6b2b3643802c20b493eb33ab5d91c', // 申请好的Web端开发者Key，首次调用 load 时必填
        version: '2.0', // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
        plugins: [''], // 需要使用的的插件列表，如比例尺'AMap.Scale'等
      })
        .then(AMap => {
          this.AMap = AMap
          this.amap = new AMap.Map('amap-container', {
            //设置地图容器id
            viewMode: '3D', //是否为3D地图模式
            zoom: 14, //初始化地图级别
            center: [106.506277, 29.52528], //初始化地图中心点位置
          })
        })
        .catch(e => {
          console.log(e)
        })
    },
  },
}
</script>

<style lang="less" scoped>
#container {
  width: 100%;
  height: 100%;
  border-radius: 20px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}
#amap-container {
  width: 100%;
  height: 75%;
  border-radius: 20px;
}
#map-imgs {
  width: 100%;
  height: 23%;
}
.el-carousel-item-img {
  width: 100%;
  height: 100%;
}

.el-carousel__item {
  box-sizing: border-box;
  border: 1px dashed #ffffff;
}
</style>
