<template>
	<div :id='mapId'></div>
</template>
<script>
import {mapGetters} from 'vuex'
// 定位功能
import {location} from './function/location'
// 加载地图服务
import {addServiceLayer,addFeatureLayer} from './function/layer'
// 底图
import {basemap} from './function/baseMap'
import {xyPointToGraphic} from './function/graphic'
export default {
  props:{
    // 初始化地图的id
    mapId: {
      type: String,
      default: ''
    },
    // 底图，数组类型，默认为topo，通过类型加以判断
    baseMap: {
      type: String,
      default: 'topo'
    }, 
    // 地图中心点坐标,arcgis的Point类型
    center: {
      type: Object,
      default: {

      }
    },
    // 缩放级别父组件传值
    zoom: {
      type: Number,
      default: -1
    }
  },
  data(){
    return {
      // 整个地图
      mapC:null
    }
  },
	mounted(){
    let x = basemap()
    
    // 初始化地图，成功后进行地图操作
    if(this.mapId){
      this.mapC = new this.map(this.mapId,{ 
        logo: false,
        slider: false,
        nav: false,
        showLabels: true, //非常重要；显示标注用
        extent: new this.Extent({
            xmin: 118.25,
            ymin: 35.1,
            xmax: 118.3,
            ymax: 35.2,
            spatialReference: {
              wkid: 4326
            }
        })
      });
    }
    // debugger
     this.mapC.addLayer(x)
     xyPointToGraphic(this.mapC, -81.3765, 28.54175,4326)
     debugger
     this.mapC.on("click", function(ev) {
       debugger
    });
    //定位功能的使用
    //this.mapC.on("load",location(this.mapC));
    //添加动态图层或者贴片图层
    //addServiceLayer('http://10.10.50.67:6080/arcgis/rest/services/GDJTGHT/MapServer','Dynamic',this.mapC,{})
    //添加要素图层
    //addFeatureLayer(this.mapC, 'http://10.10.50.67:6080/arcgis/rest/services/GJJNYLTSFQGH/MapServer/0' , {}, /*(event) => {alert(event)}*/this.testClick);
    //暴漏出接口，显示extent和center
    //接收extent和center，并缩放至
  },
  methods:{
    /**
     * 示例，featurelayer点击事件
     */
		// testClick(e){
		// 	e.stopPropagation()
    //   this.mapC.infoWindow.clearFeatures()
		// 	this.mapC.infoWindow.setTitle('老子是弹窗')
    //   this.mapC.infoWindow.show(e.graphic.geometry)
    // },
    
    /**
     * 缩放至中心点
     * @param {中心点坐标} location
     * @param {缩放级别} level
     */
    centerAndZoom(location, level){
      this.mapC.centerAndZoom(location, level)
    }
  },
  watch:{
    // 监听map的四至范围和缩放，传值到vuex
    'mapC.extent': {
      handler(newVal, oldVal) {
        //传值到store中
        if(this.mapC != null&&typeof(this.mapC.extent) !== 'undefined'){
          this.$store.dispatch('set_center', this.mapC.extent.getCenter())
          this.$store.dispatch('set_zoom', this.mapC.getLevel())
        }
      },
      immediate: true,
      deep: true
    },
    // 缩放至
    zoom(val) {
       this.mapC.setLevel(val)
    },
    // 居中至
    center:{
      handler(newVal, oldVal) {
        if(newVal != null &&oldVal != null){
          if(newVal.x !== oldVal.x || newVal.y !== oldVal.y){
            this.mapC.centerAt(newVal)
          }
        }
      }
    }
  },
  computed: {
    ...mapGetters([
      'Color','Point','graphic','ArcGISDynamicMapServiceLayer','map','SimpleLineSymbol','SimpleMarkerSymbol','mapControl','FeatureLayer','Extent'
    ]),

  }
}
</script>

<style scoped>
</style>
