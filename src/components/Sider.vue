<template>
  <div class="sider" v-show="shouldRender">
    <div class="header">
      <div class="close" @click="closeSider">✖</div>
      <div class="description">
        今天
        <span class="title">星期{{dayList[week]}}</span>，
        <span v-if="week===0">不限身份證字號購買</span>
        <span v-else>
          為
          <span class="title">{{buyDay}}</span>
          購買日
        </span>
      </div>
      <div class="search">
        <input type="text" v-model="searchValue" placeholder="請輸入坐標" />
        <span class="search-icon">
          <font-awesome-icon
            class="color_gray icon"
            icon="search"
            @click="clickSearchAdrress"
            v-on:keyup.enter="clickSearchAdrress"
          ></font-awesome-icon>
        </span>
      </div>
      <div class="count">
        <span>顯示成人口罩</span>
        <select v-model="countValue" @change="changeCount">
          <option value="0">0個以上</option>
          <option value="30">30個以上</option>
          <option value="50">50個以上</option>
          <option value="100">100個以上</option>
        </select>
        <span>的店家</span>
      </div>
    </div>

    <div class="item-container" v-if="stores">
      <div class="item" :key="store.properties.id" v-for="store in stores">
        <v-Item v-bind:store="store" @clickStoreNameAfter="handleItemCallback"></v-Item>
      </div>
      <div v-if="stores.length==0" class="no-item color_gray">查無店家</div>
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
      searchValue: "",
      addressList: [],
      dayList: ["日", "一", "二", "三", "四", "五", "六"]
    };
  },
  computed: {
    week: function() {
      var week = new Date().getDay();
      return week;
    },
    buyDay: function() {
      switch (this.week) {
        case 1:
        case 3:
        case 5:
          return "奇數";
          break;
        case 2:
        case 4:
        case 6:
          return "偶數";
          break;
      }
    }
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
      var value = this.searchValue;
      if (isChina(value)) {
        // var url ="https://restapi.amap.com/v3/geocode/geo";
        // var data = {params:{
        // address:value,
        //  key:'AIzaSyAlvlSW8Q8HCY4lGvzqeAmdNn1HeiYjkeI'
        // }};
        // this.$http.get(url,data)
        //   .then(function(responese) {
        //     var data = responese.data.result;
        //     if(data.length>0){

        //     }
        //   })
        //   .catch(function(error) {
        //     console.log(error);
        //   });
        // var geocoder = new google.maps.Geocoder();
        // geocoder.geocode({ address: value }, function(results, status) {
        //   debugger
        //   if (status == google.maps.GeocoderStatus.OK) {
        //     map.setCenter(results[0].geometry.location);

        //     document.getElementById(
        //       "lat"
        //     ).value = results[0].geometry.location.lat();
        //     document.getElementById(
        //       "lng"
        //     ).value = results[0].geometry.location.lng();
        //     var marker = new google.maps.Marker({
        //       map: map,
        //       position: results[0].geometry.location
        //     });

        //   } else {
        //     alert("失敗, 原因: " + status);
        //   }
        // });
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
.sider .description {
  color: #fff;
  padding: 0 0 15px 10px;
}
.sider .description .title {
  font-size: 24px;
  font-weight: 900;
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
  height: calc(var(--vh, 1vh) * 100 - 265px);
}
.item-container .item {
  position: relative;
  padding: 12px;
}
.no-item {
  text-align: center;
  margin-top: 20px;
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
  .sider .description .title {
  font-size: 28px;
}
}
</style>