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
  // This example illustrates a Callback Property, a property whose
  // value is lazily evaluated by a callback function.
  // Use a CallbackProperty when your data can't be pre-computed
  // or needs to be derived from other properties at runtime.
  const viewer = new Cesium.Viewer("cesiumContainer");
  viewer.clock.shouldAnimate = true;

  const startLatitude = 35;
  const startLongitude = -120;
  let endLongitude;
  const startTime = Cesium.JulianDate.now();

  // Add a polyline to the scene. Positions are dynamic.
  const isConstant = false;
  const redLine = viewer.entities.add({
    polyline: {
      // This callback updates positions each frame.
      positions: new Cesium.CallbackProperty(function (time, result) {
        endLongitude =
          startLongitude +
          0.001 * Cesium.JulianDate.secondsDifference(time, startTime);

        //返回给定的经度和纬度值数组（以度为单位），该数组由Cartesian3位置组成。
        return Cesium.Cartesian3.fromDegreesArray(
          [startLongitude, startLatitude, endLongitude, startLatitude],
          Cesium.Ellipsoid.WGS84,
          result
        );
      }, isConstant),
      width: 5,
      material: Cesium.Color.RED,
    },
  });

  const startCartographic = Cesium.Cartographic.fromDegrees(
    startLongitude,
    startLatitude
  );

  // use scratch object to avoid new allocations per frame.
  let endCartographic = new Cesium.Cartographic();
  const scratch = new Cesium.Cartographic();

  //在连接两个提供的行星点的椭球上初始化一个测地线。
  const geodesic = new Cesium.EllipsoidGeodesic();

  // Calculate the length of the line
  function getLength(time, result) {
    // Get the end position from the polyLine's callback.
    const endPoint = redLine.polyline.positions.getValue(time, result)[1];
    endCartographic = Cesium.Cartographic.fromCartesian(endPoint);

    //设置测地线的起点和终点
    geodesic.setEndPoints(startCartographic, endCartographic);

    //获取起点和终点之间的表面距离
    const lengthInMeters = Math.round(geodesic.surfaceDistance);
    return `${(lengthInMeters / 1000).toFixed(1)} km`;
  }

  function getMidpoint(time, result) {
    // Get the end position from the polyLine's callback.
    const endPoint = redLine.polyline.positions.getValue(time, result)[1];
    endCartographic = Cesium.Cartographic.fromCartesian(endPoint);

    geodesic.setEndPoints(startCartographic, endCartographic);

    //提供沿测地线在指示部分处的点的位置。
    const midpointCartographic = geodesic.interpolateUsingFraction(
      0.5,  //起点和终点之间的距离的一部分。
      scratch  //存储结果的对象。
    );

    //什么时候用弧度什么时候用经纬度
    return Cesium.Cartesian3.fromRadians(
      midpointCartographic.longitude,
      midpointCartographic.latitude
    );
  }

  // Label the polyline with calculated length.
  const label = viewer.entities.add({
    position: new Cesium.CallbackProperty(getMidpoint, isConstant),
    label: {
      // This callback updates the length to print each frame.
      text: new Cesium.CallbackProperty(getLength, isConstant),
      font: "20px sans-serif",
      pixelOffset: new Cesium.Cartesian2(0.0, 20),
    },
  });

  // Keep the view centered.
  viewer.trackedEntity = label;
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