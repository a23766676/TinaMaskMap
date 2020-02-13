<template>
  <div class="map" id="map">
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
var _graphicLayers = null;

var _graphicLayerSetting = {
  storesGraphicLayer: { layer: null, index: 1 },
  currentPositionGraphicLayer: {
    layer: null,
    index: 2
  }
};
var _map=null;
export default {
  props: ["stores", "storeId"],
  watch: {
    storeId: function(id) {
      var store = this.stores.find(function(store){
        return store.properties.id===id;
      })
      debugger
       openInfoWindow(_map, graphic, evt.screenPoint, store.geometry);
    },
    stores: function(result) {
      result.forEach(function(store) {
        var type = getType(parseInt(store.properties.mask_adult));
        addPositioningPattern(_map, type, store.geometry.coordinates).then(
          function(graphic) {
            graphic.attributes = store;
          }
        );
      });
    }
  },
  methods: {
    getType: getType,
    addPositioningPattern: addPositioningPattern
  },
  mounted() {
    // preload the ArcGIS API
    var vue = this;

    var getCenter = async function() {
      const defer = Deferred();
      var center = [];
      if ("geolocation" in navigator) {
        navigator.geolocation.getCurrentPosition(
          function(position) {
            center = [position.coords.longitude, position.coords.latitude];
            defer.resolve(center);
          },
          function(e) {
            debugger;
          },
          {
            enableHighAcuracy: true
          }
        );
      } else {
        center = [121.525, 25.0392];
        defer.resolve(center);
      }
      return defer.promise;
    };

    getCenter().then(function(center) {
      esriLoader
        .loadModules(
          [
            "esri/map",
            "esri/layers/WebTiledLayer",
            "esri/layers/GraphicsLayer",
            "esri/geometry/screenUtils"
          ],
          _options
        )
        .then(
          ([Map, WebTiledLayer, GraphicsLayer, screenUtils]) => {
            _map = new Map("map", {
              center: center,
              basemap: "gray",
              logo: false,
              fadeOnZoom: true
            });
            initBasemap(_map);
            _graphicLayers = slimGraphicLayers(_graphicLayerSetting);
            // initGraphicLayers(_graphicLayerSetting);
            addGraphicLayersToMap(_map, _graphicLayerSetting);
            addGraphicLayersEvent(_map, _graphicLayers);
            addPositioningPattern(_map, "current", center);
            addEventListener(vue, _map);
            function addEventListener(vue, map) {
              map.on("extent-change", function(data) {
                vue.$emit("callback", data.extent);
              });
              _graphicLayers.storesGraphicLayer.on("click", function(evt) {
                var graphic = evt.graphic;
                var mapPoint = screenUtils.toMapPoint(
                  map.extent,
                  map.width,
                  map.height,
                  evt.screenPoint
                );
                openInfoWindow(map, graphic, evt.screenPoint, mapPoint);
              });
            }

            function initBasemap(map) {
              map.setLevel(19);
              var layer = new WebTiledLayer(
                "https://wmts.nlsc.gov.tw/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=EMAP&STYLE=_null&TILEMATRIXSET=EPSG:3857&TILEMATRIX=EPSG:3857:{level}&TILEROW={row}&TILECOL={col}&FORMAT=image/png"
              );
              map.addLayer(layer);
            }

            function addGraphicLayersToMap(map, graphicLayerSetting) {
              var layerSort = orderByGraphicLayersByIndex(graphicLayerSetting);
              layerSort.forEach(function(item) {
                var graphicLayer = graphicLayerSetting[item.name];
                map.addLayer(graphicLayer.layer, graphicLayer.index);
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
            function slimGraphicLayers(graphicLayerSetting) {
              var result = {};
              for (var prop in graphicLayerSetting) {
                graphicLayerSetting[prop].layer = new GraphicsLayer();
                result[prop] = graphicLayerSetting[prop].layer;
              }

              return result;
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
          },
          reason => {
            console.log(reason);
          }
        );
    });
  }
};
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
function addPositioningPattern(map, type, center) {
  const defer = Deferred();
  createPointGraphic(type, center).then(function(graphic) {
    var graphicsLayer = getGraphicsLayer(type);
    graphicsLayer.add(graphic);
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
function getGraphicsLayer(type) {
  switch (type) {
    case "current":
      return _graphicLayers.currentPositionGraphicLayer;
      break;
    default:
      return _graphicLayers.storesGraphicLayer;
      break;
  }
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
  font-size: 20px;
  font-weight: 900;
  padding-bottom: 10px;
}
.map-container .infoWindow.content {
  font-size: 14px;
  line-height: 24px;
}
.esriPopup .titlePane {
  background-color: #f7f7f7;
  color: #555;
}
.esriPopup .contentPane {
  padding: 10px 18px 24px 18px;
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
  font-size: 10px;
  position: absolute;
  bottom: -10px;
  right: 0;
}
@media screen and (min-width: 768px) {
  /*電腦版*/
  .map-container .map {
    width: 73.5%;
  }
}
</style>

function newFunction() {
  const defer=Deferred();
}
