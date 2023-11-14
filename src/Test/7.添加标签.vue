<template>
  <div id="cesiumContainer" class="fullSize"></div>
    <div id="loadingOverlay"><h1>Loading...</h1></div>
    <div id="toolbar"></div>
</template>
  
<script setup>
import * as Cesium from 'cesium';
import { onMounted } from 'vue';
import '../assets/Apps/Sandcastle/Sandcastle-header'
onMounted(() => {
  createViewer1()//3D Model Coloring沙盒
})

async function createViewer1() {
  const viewer = new Cesium.Viewer("cesiumContainer", {
  infoBox: false,
  selectionIndicator: false,
  shadows: true,
  shouldAnimate: true,
});
const scene = viewer.scene;
scene.globe.depthTestAgainstTerrain = true;

if (!scene.sampleHeightSupported) {
  window.alert("This browser does not support sampleHeight.");
}

const longitude = -2.1480545852753163;
const latitude = 0.7688240036937101;
const range = 0.000001;
const duration = 4.0;

const entity = viewer.entities.add({
  position: Cesium.Cartesian3.fromRadians(longitude, latitude),
  model: {
    uri: "http://192.168.10.11:8888/cesium/gltf/GroundVehicle.glb",
  },
});

const point = viewer.entities.add({
  position: new Cesium.CallbackProperty(updatePosition, false),
  point: {
    pixelSize: 10,
    color: Cesium.Color.YELLOW,

    //获取或设置与相机的距离，在深度处禁用深度测试，例如，以防止剪切地形。设置为零时，将始终应用深度测试。
    //设置为Number.POSITIVE_INFINITY时，永远不会应用深度测试。
    disableDepthTestDistance: Number.POSITIVE_INFINITY,
  },
  label: {
    show: false,
    showBackground: true,
    font: "14px monospace",
    
    //获取或设置此标签的水平原点，确定标签是否绘制在其锚定位置的左侧，中心或右侧。
    horizontalOrigin: Cesium.HorizontalOrigin.LEFT,

//获取或设置此标签的垂直原点，以确定标签是否为到其锚定位置的上方，下方或中心。
    verticalOrigin: Cesium.VerticalOrigin.BOTTOM,

    //获取或设置屏幕空间中距此标签原点的像素偏移量。
    //这是常用的在同一位置对齐多个标签和广告牌，例如图像和文本。的屏幕空间原点是画布的左上角；
    //x 从从左到右， y 从上到下增加。
    pixelOffset: new Cesium.Cartesian2(5, 5),
    disableDepthTestDistance: Number.POSITIVE_INFINITY,
  },
});

const objectsToExclude = [point];
const cartographic = new Cesium.Cartographic();

function updatePosition(time, result) {
  
  //secondsOfDay 获取或设置当前日期的秒数。
  const offset = (time.secondsOfDay % duration) / duration;

  cartographic.longitude = longitude - range + offset * range * 2.0;
  cartographic.latitude = latitude;

  let height;
  if (scene.sampleHeightSupported) {
    //返回给定制图位置处场景几何的高度；如果没有，则返回 undefined 场景几何要从其采样高度。
    //输入位置的高度被忽略。可用于将物体夹在地球，3D瓷砖或场景中的图元。
    height = scene.sampleHeight(cartographic, objectsToExclude);
    
  }

  if (Cesium.defined(height)) {
    cartographic.height = height;
    point.label.text = `${Math.abs(height).toFixed(2).toString()} m`;
    point.label.show = true;
    
  } else {
    cartographic.height = 0.0;
    point.label.show = false;
  }

  return Cesium.Cartographic.toCartesian(
    cartographic,
    Cesium.Ellipsoid.WGS84,
    result
  );
}
viewer.trackedEntity = entity;
}

</script>
  
<style scoped>
@import url(../assets/Apps/Sandcastle/templates/bucketRaw.css);

#toolbar {
  background: rgba(42, 42, 42, 0.8);
  padding: 4px;
  border-radius: 4px;
}

#toolbar input {
  vertical-align: middle;
  padding-top: 2px;
  padding-bottom: 2px;
}

#toolbar .header {
  font-weight: bold;
}</style>