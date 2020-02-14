	<template>
  <div class="map-container clearfix">
    <div class="sider-icon" @click="clickSiderIcon">
      <font-awesome-icon class="color_blue icon" icon="search"></font-awesome-icon>
    </div>
    <v-Sider
      :should-render="showSider"
      :count="count"
      @callback="handleSiderCallback"
      @clickStoreNameAfter="storeId=$event"
      @changeCountAfter="changeCountAfter"
      @clickSearchAfter="clickSearchAfter"
      :stores="stores"
    ></v-Sider>
    <!-- <v-Sider v-model="showSider"></v-Sider> -->
    <v-Map
      @callback="handleMapCallback"
      :stores="stores"
      :storeId="storeId"
      :searchPoint="searchPoint"
    ></v-Map>
  </div>
</template>
	<script>
import vMap from "./../components/Map.vue";
import vSider from "./../components/Sider.vue";
import proj4 from "proj4";

const _searchApiUrl =
  "https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json";

export default {
  name: "Home",
  components: {
    vMap,
    vSider
  },
  data: function() {
    return {
      showSider: getShowSiderDefault(),
      stores: null,
      count: 30,
      storeId: null,
      extent: null,
      searchPoint:null
    };
  },
  created() {},
  watch: {
    count: function(value) {
      var vue = this;
      this.$http
        .get(_searchApiUrl)
        .then(function(response) {
          vue.stores = filterStores(response.data.features, vue.extent)
            .sort(compare("properties", "mask_adult"))
            .slice(0, value + 1);
        })
        .catch(function(error) {
          // 请求失败处理
          console.log(error);
        });
    }
  },
  methods: {
    clickSiderIcon: function() {
      this.showSider = true;
    },
    handleSiderCallback: function(value) {
      this.showSider = value;
    },
    handleMapCallback: function(value) {
      this.extent = value;
      var vue = this;
      this.$http
        .get(_searchApiUrl)
        .then(function(response) {
          vue.stores = filterStores(response.data.features, value)
            .sort(compare("properties", "mask_adult"))
            .slice(0, vue.count + 1);
        })
        .catch(function(error) {
          // 请求失败处理
          console.log(error);
        });
    },
    changeCountAfter: function(value) {
      this.count = Number(value);
    },
    clickSearchAfter: function(value) {
      this.searchPoint = value;
    }
  }
};
function filterStores(stores, extent) {
  var xmin = extent.xmin;
  var xmax = extent.xmax;
  var ymin = extent.ymin;
  var ymax = extent.ymax;
  var pointMin = proj4(proj4("EPSG:3857"), proj4("EPSG:4326"), [xmin, ymin]);
  var pointMax = proj4(proj4("EPSG:3857"), proj4("EPSG:4326"), [xmax, ymax]);
  var result = stores.filter(function(store) {
    var y = store.geometry.coordinates[0];
    var x = store.geometry.coordinates[1];
    var isPass = true;
    if (pointMin[0] > y || pointMax[0] < y) {
      isPass = false;
    }
    if (pointMin[1] > x || pointMax[1] < x) {
      isPass = false;
    }
    return isPass;
  });
  return result;
}
function getShowSiderDefault() {
  if (screen.width > 769) {
    return true;
  } else {
    return false;
  }
}
function compare(propertyName1, propertyName2) {
  return function(object1, object2) {
    var value1 = object1[propertyName1][propertyName2];
    var value2 = object2[propertyName1][propertyName2];
    if (value2 > value1) {
      return 1;
    } else if (value2 < value1) {
      return -1;
    } else {
      return 0;
    }
  };
}
</script>
<style scoped>
.map-container:after {
  content: "";
  display: block;
  clear: both;
}
.map-container {
  width: 100vw;
  height: 100vh;
}
.map-container .sider,
.map-container .map {
  float: left;
}
.map-container .sider-icon {
  border: 2px solid #668afe;
  width: 35px;
  line-height: 35px;
  border-radius: 50%;
  background-color: #fff;
  text-align: center;
  z-index: 3;
  position: absolute;
  top: 25px;
  left: 25px;
  font-size: 25px;
  color: #668afe;
}
.map-container a {
  text-decoration: none;
}
@media screen and (min-width: 1024px) {
  /*電腦版*/
  .map-container .sider-icon {
    display: none;
  }
  .map-container .sider {
    max-width: 400px;
  }
  .map-container .map {
    width: 73.5%;
    min-width: callback(100% - 400px);
  }
}
</style>