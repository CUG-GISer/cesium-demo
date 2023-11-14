<template>
  <div id="cesiumContainer" class="fullSize"></div>
    <div id="loadingOverlay"><h1>Loading...</h1></div>
    <div id="toolbar">
      <table>
        <tbody>
          <tr>
            <td>Background Transparency</td>
            <td>
              <input type="range" min="0.0" max="1.0" step="0.01" data-bind="value: backgroundTransparency, valueUpdate: 'input'">
            </td>
          </tr>
          <tr>
            <td>Band Transparency</td>
            <td>
              <input type="range" min="0.0" max="1.0" step="0.01" data-bind="value: bandTransparency, valueUpdate: 'input'">
            </td>
          </tr>
          <tr>
            <td>Band Thickness</td>
            <td>
              <input type="range" min="10" max="1000" step="1" data-bind="value: bandThickness, valueUpdate: 'input'">
            </td>
          </tr>
          <tr>
            <td>Band 1 Position</td>
            <td>
              <input type="range" min="4000" max="8848" step="1" data-bind="value: band1Position, valueUpdate: 'input'">
            </td>
          </tr>
          <tr>
            <td>Band 2 Position</td>
            <td>
              <input type="range" min="4000" max="8848" step="1" data-bind="value: band2Position, valueUpdate: 'input'">
            </td>
          </tr>
          <tr>
            <td>Band 3 Position</td>
            <td>
              <input type="range" min="4000" max="8848" step="1" data-bind="value: band3Position, valueUpdate: 'input'">
            </td>
          </tr>
          <tr>
            <td>Gradient</td>
            <td>
              <input type="checkbox" data-bind="checked: gradient">
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
  const viewer = new Cesium.Viewer("cesiumContainer", {
    terrain: Cesium.Terrain.fromWorldTerrain({
      requestVertexNormals: true, //Needed to visualize slope
    }),
  });

  viewer.camera.setView({
    destination: new Cesium.Cartesian3(
      290637.5534733206,
      5637471.593707632,
      2978256.8126927214
    ),
    orientation: {
      heading: 4.747266966349747,
      pitch: -0.2206998858596192,
      roll: 6.280340554587955,
    },
  });

  const viewModel = {
    gradient: false,
    band1Position: 7000.0,
    band2Position: 7500.0,
    band3Position: 8000.0,
    bandThickness: 100.0,
    bandTransparency: 0.5,
    backgroundTransparency: 0.75,
  };

  //这一行代码使用 Knockout 的 track 函数来追踪 viewModel 对象中的属性变化。
  Cesium.knockout.track(viewModel);
  const toolbar = document.getElementById("toolbar");

  //将 viewModel 对象绑定到页面上的 "toolbar" 元素，
  //使得该元素内部的内容能够反映 viewModel 对象的状态。
  Cesium.knockout.applyBindings(viewModel, toolbar);
  for (const name in viewModel) {

    //确保只处理 viewModel 对象自身的属性，而不包括从原型链上继承的属性。
    if (viewModel.hasOwnProperty(name)) {
      Cesium.knockout
        .getObservable(viewModel, name)
        .subscribe(updateMaterial);
        //对于每个属性，使用 Cesium.knockout.getObservable 获取该属性的可观察对象，
        //然后调用 subscribe 方法订阅这个可观察对象的变化。
        //当属性的值发生变化时，将调用 updateMaterial 函数，
    }
  }

  function updateMaterial() {
    const gradient = Boolean(viewModel.gradient);
    const band1Position = Number(viewModel.band1Position);
    const band2Position = Number(viewModel.band2Position);
    const band3Position = Number(viewModel.band3Position);
    const bandThickness = Number(viewModel.bandThickness);
    const bandTransparency = Number(viewModel.bandTransparency);
    const backgroundTransparency = Number(
      viewModel.backgroundTransparency
    );

    const layers = [];
    const backgroundLayer = {
      entries: [
        {
          height: 4200.0,
          color: new Cesium.Color(0.0, 0.0, 0.1, backgroundTransparency),
        },
        {
          height: 8000.0,
          color: new Cesium.Color(1.0, 1.0, 1.0, backgroundTransparency),
        },
        {
          height: 8500.0,
          color: new Cesium.Color(1.0, 0.0, 0.0, backgroundTransparency),
        },
      ],
      extendDownwards: true,
      extendUpwards: true,
    };
    layers.push(backgroundLayer);

    const gridStartHeight = 4200.0;
    const gridEndHeight = 8848.0;
    const gridCount = 50;
    for (let i = 0; i < gridCount; i++) {
      const lerper = i / (gridCount - 1);
      const heightBelow = Cesium.Math.lerp(
        gridStartHeight,
        gridEndHeight,
        lerper
      );
      const heightAbove = heightBelow + 10.0;
      const alpha =
        Cesium.Math.lerp(0.2, 0.4, lerper) * backgroundTransparency;
      layers.push({
        entries: [
          {
            height: heightBelow,
            color: new Cesium.Color(1.0, 1.0, 1.0, alpha),
          },
          {
            height: heightAbove,
            color: new Cesium.Color(1.0, 1.0, 1.0, alpha),
          },
        ],
      });
    }

    const antialias = Math.min(10.0, bandThickness * 0.1);

    if (!gradient) {
      const band1 = {
        entries: [
          {
            height: band1Position - bandThickness * 0.5 - antialias,
            color: new Cesium.Color(0.0, 0.0, 1.0, 0.0),
          },
          {
            height: band1Position - bandThickness * 0.5,
            color: new Cesium.Color(0.0, 0.0, 1.0, bandTransparency),
          },
          {
            height: band1Position + bandThickness * 0.5,
            color: new Cesium.Color(0.0, 0.0, 1.0, bandTransparency),
          },
          {
            height: band1Position + bandThickness * 0.5 + antialias,
            color: new Cesium.Color(0.0, 0.0, 1.0, 0.0),
          },
        ],
      };

      const band2 = {
        entries: [
          {
            height: band2Position - bandThickness * 0.5 - antialias,
            color: new Cesium.Color(0.0, 1.0, 0.0, 0.0),
          },
          {
            height: band2Position - bandThickness * 0.5,
            color: new Cesium.Color(0.0, 1.0, 0.0, bandTransparency),
          },
          {
            height: band2Position + bandThickness * 0.5,
            color: new Cesium.Color(0.0, 1.0, 0.0, bandTransparency),
          },
          {
            height: band2Position + bandThickness * 0.5 + antialias,
            color: new Cesium.Color(0.0, 1.0, 0.0, 0.0),
          },
        ],
      };

      const band3 = {
        entries: [
          {
            height: band3Position - bandThickness * 0.5 - antialias,
            color: new Cesium.Color(1.0, 0.0, 0.0, 0.0),
          },
          {
            height: band3Position - bandThickness * 0.5,
            color: new Cesium.Color(1.0, 0.0, 0.0, bandTransparency),
          },
          {
            height: band3Position + bandThickness * 0.5,
            color: new Cesium.Color(1.0, 0.0, 0.0, bandTransparency),
          },
          {
            height: band3Position + bandThickness * 0.5 + antialias,
            color: new Cesium.Color(1.0, 0.0, 0.0, 0.0),
          },
        ],
      };

      layers.push(band1);
      layers.push(band2);
      layers.push(band3);
    } else {
      const combinedBand = {
        entries: [
          {
            height: band1Position - bandThickness * 0.5,
            color: new Cesium.Color(0.0, 0.0, 1.0, bandTransparency),
          },
          {
            height: band2Position,
            color: new Cesium.Color(0.0, 1.0, 0.0, bandTransparency),
          },
          {
            height: band3Position + bandThickness * 0.5,
            color: new Cesium.Color(1.0, 0.0, 0.0, bandTransparency),
          },
        ],
      };

      layers.push(combinedBand);
    }

    const material = Cesium.createElevationBandMaterial({
      scene: viewer.scene,
      layers: layers,
    });
    viewer.scene.globe.material = material;
  }

  updateMaterial();
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