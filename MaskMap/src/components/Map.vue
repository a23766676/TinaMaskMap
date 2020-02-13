<template>
  <div class="map" id="map">
    <font-awesome-icon class="color_gray back icon" icon="undo-alt" @click="goBackCurrentPoint"></font-awesome-icon>
    <div id="LocateButton"></div>
    <div>
      <div class></div>
    </div>
  </div>
</template>

<script>
import esriLoader from "esri-loader";
import Deferred from "vue-deferred";

const _symbolFile = {
  currentPosition: "map-marker-current.svg",
  stores: {
    many: "map-marker-many.svg",
    less: "map-marker-less.svg",
    zero: "map-marker-zero.svg"
  }
};
const _options = {
  url: "https://js.arcgis.com/3.31/"
};
var _graphicLayerSetting = {
  storesGraphicLayer: { layer: null, index: 1 },
  currentPositionGraphicLayer: {
    layer: null,
    index: 2
  }
};
export default {
  props: ["stores", "storeId", "searchPoint"],
  data: () => {
    return {
      graphics: [],
      map: null,
      screenUtils: null,
      graphicLayers: null
    };
  },
  watch: {
    searchPoint: function(point) {
      if (point[0] && point[1]) {
        var graphicLayer = this.graphicLayers.currentPositionGraphicLayer;
        graphicLayer.clear();
        addPointGraphic(this.map, "current", point, graphicLayer);
        this.map.centerAndZoom(point, 17);
      }else{
        alert('很抱歉，我們搜尋不到相關資料！');
      }
    },
    storeId: function(id) {
      var map = this.map;
      var graphic = this.graphics.find(function(graphic) {
        return graphic.attributes.properties.id === id;
      });
      var screenPoint = this.screenUtils.toScreenPoint(
        map.extent,
        map.width,
        map.height,
        graphic.geometry
      );
      openInfoWindow(map, graphic, screenPoint, graphic.geometry);
    },
    stores: function(result) {
      var vue = this;
      vue.graphics = [];
      var graphicLayer = vue.graphicLayers.storesGraphicLayer;
      graphicLayer.clear();
      result.forEach(function(store) {
        var type = getType(parseInt(store.properties.mask_adult));
        addPointGraphic(
          map,
          type,
          store.geometry.coordinates,
          graphicLayer
        ).then(function(graphic) {
          graphic.attributes = store;
          vue.graphics.push(graphic);
        });
      });
    }
  },
  methods: {},
  created() {
    esriLoader
      .loadModules(
        [
          "esri/map",
          "esri/layers/WebTiledLayer",
          "esri/layers/GraphicsLayer",
          "esri/geometry/screenUtils",
          "esri/dijit/LocateButton"
        ],
        _options
      )
      .then(
        ([Map, WebTiledLayer, GraphicsLayer, screenUtils, LocateButton]) => {
          this.map = new Map("map", {
            center: [121.525, 25.0392],
            basemap: "gray",
            logo: false,
            fadeOnZoom: true,
            minZoom: 7,
            maxZoom: 18
          });
          this.initBasemap(this.map, WebTiledLayer);
          this.graphicLayers = this.slimGraphicLayers(
            _graphicLayerSetting,
            GraphicsLayer
          );
          this.addGraphicLayersToMap(this.map, _graphicLayerSetting);
          this.addGraphicLayersEvent(this.map, this.graphicLayers);
          this.addEventListener(this);
          this.screenUtils = screenUtils;
          var vue = this;
          var graphicLayer = this.graphicLayers.currentPositionGraphicLayer;
          addCurrentPoint(vue.map, graphicLayer, LocateButton);
        },
        reason => {
          console.log(reason);
        }
      );
  },
  mounted() {},
  methods: {
    getType: getType,
    addPointGraphic: addPointGraphic,
    addCurrentPoint: addCurrentPoint,
    initBasemap: initBasemap,
    slimGraphicLayers: slimGraphicLayers,
    addGraphicLayersToMap: addGraphicLayersToMap,
    addGraphicLayersEvent: addGraphicLayersEvent,
    addEventListener: addEventListener,
    orderByGraphicLayersByIndex: orderByGraphicLayersByIndex,
    goBackCurrentPoint: function() {
      var graphicLayer = this.graphicLayers.currentPositionGraphicLayer;
      addCurrentPoint(this.map, graphicLayer);
    }
  }
};

function addCurrentPoint(map, graphicLayer, LocateButton) {
  // var geoLocate = new LocateButton(
  //   {
  //     map: map
  //   },
  //   "LocateButton"
  // );
  // geoLocate.startup();
  // geoLocate.on("load", function() {
  //   geoLocate.locate();
  // });
  graphicLayer.clear();
  if (navigator.geolocation) {
    // timeout at 60000 milliseconds (60 seconds)
    var options = { timeout: 60000, enableHighAcuracy: true };
    navigator.geolocation.getCurrentPosition(
      function(position) {
        var center = [position.coords.longitude, position.coords.latitude];
        map.centerAndZoom(center, 17);
        addPointGraphic(map, "current", center, graphicLayer);
      },
      function(e) {
        console.log(e.message);
      },
      options
    );
  }
}
function initBasemap(map, WebTiledLayer) {
  map.setLevel(15);
  var layer = new WebTiledLayer(
    "https://wmts.nlsc.gov.tw/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=EMAP&STYLE=_null&TILEMATRIXSET=EPSG:3857&TILEMATRIX=EPSG:3857:{level}&TILEROW={row}&TILECOL={col}&FORMAT=image/png"
  );
  map.addLayer(layer);
}
function slimGraphicLayers(graphicLayerSetting, GraphicsLayer) {
  var result = {};
  for (var prop in graphicLayerSetting) {
    graphicLayerSetting[prop].layer = new GraphicsLayer();
    result[prop] = graphicLayerSetting[prop].layer;
  }

  return result;
}
function addGraphicLayersToMap(map, graphicLayerSetting) {
  var layerSort = this.orderByGraphicLayersByIndex(graphicLayerSetting);
  layerSort.forEach(function(item) {
    var graphicLayer = graphicLayerSetting[item.name];
    map.addLayer(graphicLayer.layer, graphicLayer.index);
  });
}
function addEventListener(vue) {
  var map = vue.map;
  map.on("extent-change", function(data) {
    vue.$emit("callback", data.extent);
  });
  this.graphicLayers.storesGraphicLayer.on("click", function(evt) {
    var graphic = evt.graphic;
    var mapPoint = vue.screenUtils.toMapPoint(
      map.extent,
      map.width,
      map.height,
      evt.screenPoint
    );
    openInfoWindow(map, graphic, evt.screenPoint, mapPoint);
  });
}
function addGraphicLayersEvent(map, graphicLayers) {
  graphicLayers.storesGraphicLayer.on("mouse-over", function() {
    map.setMapCursor("pointer");
  });
  graphicLayers.storesGraphicLayer.on("mouse-out", function() {
    map.setMapCursor("default");
  });
}

function orderByGraphicLayersByIndex(graphicLayerSetting) {
  var result = [];
  for (var prop in graphicLayerSetting) {
    result.push({
      name: prop,
      index: graphicLayerSetting[prop].index
    });
  }
  result.sort(function(a, b) {
    return a.index - b.index;
  });

  return result;
}

function openInfoWindow(map, graphic, screenPoint, mapPoint) {
  var graphicAttributes = graphic.attributes;
  var title = "";
  var content =
    '<p class="infoWindow title">' + graphicAttributes.properties.name + "</p>";
  content +=
    '<a target="_blank" href="https://www.google.com/maps/dir/?api=1&amp;destination=' +
    graphicAttributes.geometry.coordinates[1] +
    "," +
    graphicAttributes.geometry.coordinates[0] +
    '"><p class="content infoWindow">' +
    graphicAttributes.properties.address +
    "</p></a>";
  var phoneString = graphicAttributes.properties.phone.split("-");
  var phone =
    "tel:+886-" + phoneString[0].replace("0", "") + "-" + phoneString[1];
  content +=
    '<a href="' +
    phone +
    '"><p class="content infoWindow">' +
    graphicAttributes.properties.phone +
    "</p></a>";
  content += '<div class="mask-container">';
  content +=
    '<div><p class="title infoWindow color_blue">成人口罩 ' +
    graphicAttributes.properties.mask_adult +
    '</p><p class="title infoWindow color_green">兒童口罩 ' +
    graphicAttributes.properties.mask_child +
    "</p></div>";
  if (graphicAttributes.properties.updated == "") {
    content +=
      '<div class="update"><p class="note infoWindow color_gray">更新時間：無資料</p></div>';
  } else {
    content +=
      '<div class="update"><p class="note infoWindow color_gray">更新時間：' +
      graphicAttributes.properties.updated +
      "</p></div>";
  }
  content += "</div>";
  map.infoWindow.setTitle(title);
  map.infoWindow.setContent(content);
  map.infoWindow.show(screenPoint, map.getInfoWindowAnchor(screenPoint));

  map.centerAt(mapPoint);
}
function getType(amount) {
  if (amount >= 100) {
    return "many";
  } else if (amount > 0) {
    return "less";
  } else {
    return "zero";
  }
}
function addPointGraphic(map, type, center, graphicLayer) {
  const defer = Deferred();
  createPointGraphic(type, center).then(function(graphic) {
    graphicLayer.add(graphic);
    defer.resolve(graphic);
  });
  return defer.promise;
}
function createPointGraphic(type, center) {
  const defer = Deferred();
  var fileName = getSymbolFileName(type);
  esriLoader
    .loadModules(
      [
        "esri/geometry/Point",
        "esri/graphic",
        "esri/symbols/PictureMarkerSymbol"
      ],
      _options
    )
    .then(([Point, Graphic, PictureMarkerSymbol]) => {
      var point = new Point(center);
      var symbol = new PictureMarkerSymbol({
        url: require("../assets/images/" + fileName),
        width: 20,
        height: 25,
        xoffset: 0,
        yoffset: 0
      });
      var graphic = new Graphic(point, symbol);
      defer.resolve(graphic);
    });
  return defer.promise;
}

function getSymbolFileName(type) {
  switch (type) {
    case "current":
      return _symbolFile.currentPosition;
      break;
    case "many":
      return _symbolFile.stores.many;
      break;
    case "less":
      return _symbolFile.stores.less;
      break;
    case "zero":
      return _symbolFile.stores.zero;
      break;
  }
}
</script>

<style>
@import url("https://js.arcgis.com/3.28/esri/css/esri.css");

.map {
  width: 100%;
  height: 100%;
}
.esriPopup .esriPopupWrapper {
  background: #fff;
  height: 225px;
}
.esriPopup .sizer {
  width: 160px;
}
.map-container .esriSimpleSliderTL {
  top: initial;
  left: initial;
  bottom: 20px;
  right: 20px;
}
.map-container p.infoWindow {
  font-family: "Noto Sans TC", "微軟正黑體";
  margin: 0;
}
.map-container .infoWindow.title {
  font-size: 16px;
  font-weight: 900;
  padding-bottom: 10px;
}
.map-container .infoWindow.content {
  font-size: 10px;
  line-height: 20px;
}
.esriPopup .titlePane {
  background-color: #f7f7f7;
  color: #555;
}
.esriPopup .contentPane {
  padding: 3px 10px 20px 15px;
  box-sizing: border-box;
}
.esriPopup .titleButton {
  background-image: url("../assets/images/close.svg") !important;
}
.esriPopup .titleButton.close {
  right: 8px;
  top: 8px;
  width: 15px;
  background-position: center;
  height: 15px;
}
.esriPopup .titleButton.maximize {
  display: none;
}

.map-container .mask-container {
  position: relative;
  margin-top: 30px;
}

.map-container .mask-container .update {
  font-size: 8px;
  position: absolute;
  bottom: -20px;
  right: 0;
}
.LocateButton .zoomLocateButton {
  position: absolute;
  right: 20px;
  bottom: 95px;
  z-index: 3;
}
.map-container .map .back.icon {
  display: inline-block;
  position: absolute;
  right: 20px;
  bottom: 95px;
  z-index: 3;
  background-color: #fff;
  border-radius: 50%;
  padding: 7px;
  border: 1.5px solid #888;
}
@media screen and (min-width: 1024px) {
  /*電腦版*/
  .esriPopup .esriPopupWrapper {
    height: inherit;
  }
  .esriPopup .sizer {
    z-index: 41;
    width: 270px;
  }
  .esriPopup .contentPane {
    padding: 10px 18px 24px 18px;
  }
  .map-container .infoWindow.content {
    font-size: 14px;
    line-height: 24px;
  }
  .map-container .infoWindow.title {
    font-size: 20px;
  }
  .map-container .mask-container .update {
    font-size: 10px;
    bottom: -10px;
  }
}
</style>
