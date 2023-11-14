<template>
  <!-- <chooseFeature></chooseFeature>//选择要素 -->
  <!-- <MOdel_Coloring></MOdel_Coloring>//模型样式 -->
  <!-- <selectedPlane></selectedPlane> //处理平面-->
  <!-- <DrawModel></DrawModel> //绘制模型-->
  <!-- <modelCzml></modelCzml> //卫星模型 -->
  <!-- <CallbackProperty></CallbackProperty> //回调函数-->
  <ClampTo3DModel></ClampTo3DModel>
</template>

<script setup>
import chooseFeature from "../src/Test/1.chooseFeature.vue";
import MOdel_Coloring from "../src/Test/2.3D_Model_Coloring.vue";
import selectedPlane from "../src/Test/3.裁剪平面.vue"
import DrawModel from "../src/Test/4.绘制模型.vue"
import modelCzml from "../src/Test/5.卫星轨迹.vue"
import CallbackProperty from "../src/Test/6.回调函数.vue"
import ClampTo3DModel from "../src/Test/7.添加标签.vue"
const components = [
  chooseFeature,
  MOdel_Coloring,
  selectedPlane,
  DrawModel,
  modelCzml,
  CallbackProperty,
  ClampTo3DModel,
]
</script>
<style scoped></style>
