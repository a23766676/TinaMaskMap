<template>
  <div class="sider" v-show="shouldRender">
    <div class="close" @click="closeSider">✖</div>
    <div class="search">
      <input type="text" name="address" placeholder="請輸入坐標或地址" />
      <span class="search-icon">
        <!-- <i class="fa fa-search color_gray"></i> -->
        <font-awesome-icon class="color_gray icon" icon="search"></font-awesome-icon>
      </span>
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
  props: ["shouldRender", "stores"],
  components: {
    vItem
  },
  methods: {
    closeSider: function() {
      this.$emit("callback", false);
    },
    handleItemCallback:function(storeId){
      this.$emit("clickStoreNameAfter", storeId);
    }
  }
};
</script>
<style scoped>
.sider {
  background-color: #fff;
  box-shadow: rgba(248, 190, 190, 0.5) 2px 0px 11px;
  z-index: 3;
  color: #555;
  position: absolute;
  padding: 15px 0;
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
  z-index: 1;
  right: 20px;
}
.sider .search {
  padding: 0 10px;
  position: relative;
}
.sider .search input {
  width: 80%;
  border-radius: 20px;
  border: 1px solid #999;
  line-height: 30px;
  padding-left: 15px;
}
.sider .search input:focus {
  border-radius: 20px;
}
.sider .search .search-icon {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  right: 10%;
  vertical-align: top;
  cursor: pointer;
}
.sider .note {
  position: absolute;
  bottom: 10px;
  font-size: 12px;
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
  margin: 20px 0px 20px 10px;
  height: calc(100vh - 190px);
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
@media screen and (min-width: 768px) {
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
}
</style>