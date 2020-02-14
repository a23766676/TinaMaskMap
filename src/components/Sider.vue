<template>
  <div class="sider" v-show="shouldRender">
    <div class="header">
      <div class="close" @click="closeSider">✖</div>
      <div class="search">
        <input type="text" v-model="searchValue" placeholder="請輸入坐標或地址" />
        <span class="search-icon">
          <!-- <i class="fa fa-search color_gray"></i> -->
          <font-awesome-icon
            class="color_gray icon"
            icon="search"
            @click="clickSearchAdrress"
            v-on:keyup.enter="clickSearchAdrress"
          ></font-awesome-icon>
        </span>
      </div>
      <div class="count">
        <span>最多顯示</span>
        <select v-model="countValue" @change="changeCount">
          <option value="10">10</option>
          <option value="30">30</option>
          <option value="50">50</option>
          <option value="100">100</option>
        </select>
        <span>個</span>
      </div>
    </div>

    <div class="item-container">
      <div class="item" :key="store.properties.id" v-for="store in stores">
        <v-Item v-bind:store="store" @clickStoreNameAfter="handleItemCallback"></v-Item>
      </div>
    </div>
    <div class="note color_gray">
      <p>
        <font-awesome-icon class="icon" icon="exclamation-triangle"></font-awesome-icon>部分藥局因採發放號碼牌方式，方便民眾購買口罩，系統目前無法顯示已發送號碼牌數量。
      </p>
      <p>
        <font-awesome-icon class="icon" icon="exclamation-triangle"></font-awesome-icon>口罩數量以藥局實際存量為主，線上查詢之數量僅供參考。
      </p>
    </div>
  </div>
</template>
		<script>
import vItem from "./../components/Slider/Item.vue";
export default {
  name: "Sider",
  props: ["shouldRender", "stores", "count"],
  data: function() {
    return {
      countValue: this.count,
      searchValue: ""
    };
  },
  watch: {},
  components: {
    vItem
  },
  methods: {
    closeSider: function() {
      this.$emit("callback", false);
    },
    handleItemCallback: function(storeId) {
      this.$emit("clickStoreNameAfter", storeId);
    },
    changeCount: function() {
      this.$emit("changeCountAfter", this.countValue);
    },
    clickSearchAdrress: function() {
      debugger;
      var value = this.searchValue;
      if (isChina(value)) {
        var url =
          "https://www.tgos.tw/TGOS/Web/Address/TGOS_AddressBatchQuery.aspx/QueryAddr?";
        var config = {
          data: {
            oAPPId: "QP8t8akpQxtPYZLixwpt96npqVLO46UG/w7P3SrgJwGCkUEdDkPbdw==",
            oAPIKey:
              "cGEErDNy5yNTw4lb4fzGu/McHxMCS5EctGpg7uDXUFGjmz3K4zulLpOYjnzZgP5b+vnxIpmurxrdDUSVbG5EW6eeHh07xNgNiHEfuZtKjWih08UKZzq+Zf/fssG1IQJHdH9jVgi0lsXW3aDM86QjMPwddvCUTstFOiIPebP2+1Kx0SXmAXXmiAruuhN7xb9wf2IWJ/5iy9+E5ZK2so7BX4+hRv/LU7esyHGOkA35CKAI+ehHQEe0hWRhRj5QQ46a573EyyY8r1GTH2WGBqEV8v2gDw7Spwu0wKKZA60bAy0eETpkGxA1ZZg2eb+E3ToErTuEVo3VXPKl6gJswzOj9IDsteSS80Bj",
              oAddress:value,
              oSRS:'EPSG:4326',
              oFuzzyType:0,
              oResultDataType:'JSON',
              oReturnMaxCount:5


          
          },

        };
        this.$http.post(url, config).then(function(responese) {debugger});
      } else {
        var point = [];
        var latAndLon = value.split(",");
        point.push(Number(latAndLon[1]));
        point.push(Number(latAndLon[0]));
        this.$emit("clickSearchAfter", point);
      }
    }
  }
};

function isChina(s) {
  var patrn = /[\u4E00-\u9FA5]|[\uFE30-\uFFA0]/gi;
  if (!patrn.exec(s)) {
    return false;
  } else {
    return true;
  }
}
</script>
<style scoped>
.sider {
  background-color: #fff;
  box-shadow: rgba(248, 190, 190, 0.5) 2px 0px 11px;
  z-index: 41;
  position: absolute;
  padding-bottom: 15px;
  box-sizing: border-box;
  width: 100%;
  height: 100%;
}
.sider.active {
  z-index: 31;
  display: block;
}
.sider .close {
  position: absolute;
  top: 20px;
  color: #fff;
  z-index: 1;
  right: 20px;
}
.sider .header {
  padding: 15px 0;
  background: linear-gradient(
    123.72deg,
    #4065e3 0%,
    rgba(120, 213, 215, 0.81) 105.41%
  );
}
.sider .search {
  padding: 0 10px;
  position: relative;
}
.sider .search input {
  width: 80%;
  border-radius: 20px;
  border: 1px solid #668afe;
  line-height: 30px;
  padding-left: 15px;
}
.sider .count select {
  border-radius: 20px;
  padding: 3px 0 3px 15px;
}
/* .sider .search input::placeholder {
  color:#668afe;
} */
.sider .search input:focus {
  outline: none;
}
.sider .search .search-icon {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  right: 21%;
  vertical-align: top;
  cursor: pointer;
}
.sider .search .search-icon svg {
  color: #668afe;
}
.sider .count {
  padding: 10px 0 5px 15px;
  color: #fff;
}
.sider .note {
  position: absolute;
  bottom: 10px;
  font-size: 12px;
  padding: 0 10px;
}

.sider .note p {
  margin: 5px 0;
  line-height: 16px;
}

.sider .note .icon {
  color: red;
}
.sider .item-container {
  overflow-y: auto;
  margin: 10px 0px 20px 10px;
  height: calc(100vh - 230px);
}
.item-container .item {
  position: relative;
  padding: 12px;
}
.item-container .item:not(:last-child)::after {
  content: "";
  display: inline-block;
  width: 100%;
  bottom: 0px;
  height: 2px;
  position: absolute;
  background-color: #ddd;
  left: -10px;
  z-index: -1;
}
@media screen and (min-width: 1024px) {
  .sider .close {
    display: none;
  }
  .sider {
    position: relative;
    width: 26.5%;
    display: block;
  }
  .sider .search input {
    width: 92%;
  }
  .sider .search .search-icon {
    transform: translateY(-50%);
    right: 10%;
  }
}
</style>