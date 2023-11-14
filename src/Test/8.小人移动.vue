<template>
  <div id="cesiumContainer" class="fullSize"></div>
  <div id="loadingOverlay">
    <h1>Loading...</h1>
  </div>
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
    shouldAnimate: true,
  });
  const scene = viewer.scene;

  const position = Cesium.Cartesian3.fromDegrees(
    -123.0744619,
    44.0503706
  );
  const url = "../SampleData/models/CesiumMan/Cesium_Man.glb";
  const entity = (viewer.trackedEntity = viewer.entities.add({
    name: url,
    position: position,
    model: {
      uri: url,
    },
  }));

  // Shade selected model with highlight.
  const fragmentShaderSource = `
  uniform sampler2D colorTexture;
  in vec2 v_textureCoordinates;
  uniform vec4 highlight;
  void main() {
      vec4 color = texture(colorTexture, v_textureCoordinates);
      if (czm_selected()) {
          vec3 highlighted = highlight.a * highlight.rgb + (1.0 - highlight.a) * color.rgb;
          out_FragColor = vec4(highlighted, 1.0);
      } else { 
          out_FragColor = color;
      }
  }
  `;

  const stage = scene.postProcessStages.add(
    new Cesium.PostProcessStage({
      fragmentShader: fragmentShaderSource,
      uniforms: {
        highlight: function () {
          return new Cesium.Color(1.0, 0.0, 0.0, 0.5);
        },
      },
    })
  );
  stage.selected = [];

  const handler = new Cesium.ScreenSpaceEventHandler(viewer.scene.canvas);
  handler.setInputAction(function (movement) {
    const pickedObject = viewer.scene.pick(movement.endPosition);
    if (Cesium.defined(pickedObject)) {
      stage.selected = [pickedObject.primitive];
    } else {
      stage.selected = [];
    }
  }, Cesium.ScreenSpaceEventType.MOUSE_MOVE);
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