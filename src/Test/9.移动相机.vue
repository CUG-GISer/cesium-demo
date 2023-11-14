<template>
  <div id="cesiumContainer" class="fullSize"></div>
    <div id="loadingOverlay"><h1>Loading...</h1></div>
    <div id="toolbar">
      <table class="infoPanel">
        <tbody>
          <tr>
            <td>Click on the Cesium display to start.</td>
          </tr>
          <tr>
            <td>w/s - move forward/backward</td>
          </tr>
          <tr>
            <td>a/d - move left/right</td>
          </tr>
          <tr>
            <td>q/e - move up/down</td>
          </tr>
          <tr>
            <td>
              left mouse button down plus mouse move changes the look direction
            </td>
          </tr>
        </tbody>
      </table>
    </div>
</template>
  
<script setup>
import * as Cesium from 'cesium';
import { onMounted } from 'vue';
import '../assets/Apps/Sandcastle/Sandcastle-header'
onMounted(() => {
  createViewer1()//3D Model Coloring沙盒
})

async function createViewer1() {
  const viewer = new Cesium.Viewer("cesiumContainer");

  const scene = viewer.scene;
  const canvas = viewer.canvas;

  
  canvas.setAttribute("tabindex", "0"); //它允许用户通过键盘与 canvas 进行交互。
  canvas.onclick = function () {
    canvas.focus();
  };
  const ellipsoid = scene.globe.ellipsoid;

  // disable the default event handlers
  scene.screenSpaceCameraController.enableRotate = false;
  scene.screenSpaceCameraController.enableTranslate = false;
  scene.screenSpaceCameraController.enableZoom = false;
  scene.screenSpaceCameraController.enableTilt = false;
  scene.screenSpaceCameraController.enableLook = false;

  let startMousePosition;
  let mousePosition;
  const flags = {
    looking: false,
    moveForward: false,
    moveBackward: false,
    moveUp: false,
    moveDown: false,
    moveLeft: false,
    moveRight: false,
  };

  const handler = new Cesium.ScreenSpaceEventHandler(canvas);

  handler.setInputAction(function (movement) {
    flags.looking = true;
    mousePosition = startMousePosition = Cesium.Cartesian3.clone(
      movement.position
    );
  }, Cesium.ScreenSpaceEventType.LEFT_DOWN);

  handler.setInputAction(function (movement) {
    mousePosition = movement.endPosition;
  }, Cesium.ScreenSpaceEventType.MOUSE_MOVE);

  handler.setInputAction(function (position) {
    flags.looking = false;
  }, Cesium.ScreenSpaceEventType.LEFT_UP);

  function getFlagForKeyCode(keyCode) {
    switch (keyCode) {
      case "W".charCodeAt(0):
        return "moveForward";
      case "S".charCodeAt(0):
        return "moveBackward";
      case "Q".charCodeAt(0):
        return "moveUp";
      case "E".charCodeAt(0):
        return "moveDown";
      case "D".charCodeAt(0):
        return "moveRight";
      case "A".charCodeAt(0):
        return "moveLeft";
      default:
        return undefined;
    }
  }

  document.addEventListener(
    "keydown",
    function (e) {
      const flagName = getFlagForKeyCode(e.keyCode);
      if (typeof flagName !== "undefined") {
        flags[flagName] = true;
      }
    },
    false
  );

  document.addEventListener(
    "keyup",
    function (e) {
      const flagName = getFlagForKeyCode(e.keyCode);
      if (typeof flagName !== "undefined") {
        flags[flagName] = false;
      }
    },
    false
  );

  viewer.clock.onTick.addEventListener(function (clock) {
    const camera = viewer.camera;

    if (flags.looking) {
      const width = canvas.clientWidth;
      const height = canvas.clientHeight;

      // Coordinate (0.0, 0.0) will be where the mouse was clicked.
      const x = (mousePosition.x - startMousePosition.x) / width;
      const y = -(mousePosition.y - startMousePosition.y) / height;

      const lookFactor = 0.05;
      camera.lookRight(x * lookFactor);
      camera.lookUp(y * lookFactor);
    }

    // Change movement speed based on the distance of the camera to the surface of the ellipsoid.
    const cameraHeight = ellipsoid.cartesianToCartographic(
      camera.position
    ).height;
    const moveRate = cameraHeight / 100.0;

    if (flags.moveForward) {
      camera.moveForward(moveRate);
    }
    if (flags.moveBackward) {
      camera.moveBackward(moveRate);
    }
    if (flags.moveUp) {
      camera.moveUp(moveRate);
    }
    if (flags.moveDown) {
      camera.moveDown(moveRate);
    }
    if (flags.moveLeft) {
      camera.moveLeft(moveRate);
    }
    if (flags.moveRight) {
      camera.moveRight(moveRate);
    }
  });
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
}
</style>