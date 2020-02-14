<template>
  <div>
    <div class="color_blue title" @click="clickTitle(store.properties.id)">
      <font-awesome-icon class="icon" icon="map-marker"></font-awesome-icon>
      {{store.properties.name}}
    </div>
    <div class="content">
      <a target="_blank" v-bind:href="url">
        <p class="content infoWindow">{{store.properties.address}}</p>
      </a>
      <a v-bind:href="phobeNumber">
        <p class="content infoWindow">{{store.properties.phone}}</p>
      </a>
    </div>
    <div class="update color_gray">更新時間：{{store.properties.updated}}</div>
    <div class="mask-container">
      <div class="title color_blue">成人口罩 {{store.properties.mask_adult}}</div>
      <div class="title color_green">兒童口罩 {{store.properties.mask_child}}</div>
    </div>
  </div>
</template>
		<script>
export default {
  props: ["store"],
  components: {},
  methods: {
    clickTitle: function(storeId) {
      this.$emit("clickStoreNameAfter", storeId);
    }
  },
  computed: {
    url: function() {
      return (
        "https://www.google.com/maps/dir/?api=1&amp;destination=" +
        this.store.geometry.coordinates[1] +
        "," +
        this.store.geometry.coordinates[0]
      );
    },
    phobeNumber: function() {
      var phoneString = this.store.properties.phone.split("-");
      return (
        "tel:+886-" + phoneString[0].replace("0", "") + "-" + phoneString[1]
      );
    }
  }
};
</script>
<style scoped>
.item .title {
  font-size: 20px;
  font-weight: 900;
  padding-bottom: 8px;
  cursor: pointer;
}
.item .content {
  font-size: 14px;
  line-height: 24px;
}
.item .mask-container {
  position: relative;
  margin: 10px 0 10px;
  box-shadow: 0px 0px 8px #bbb;
  display: inline-block;
}
.item .mask-container:after {
  content: "";
  display: block;
  clear: both;
}
.item .mask-container div {
  float: left;
  width: 130px;
  text-align: center;
  line-height: 38px;
  border-radius: 20px;
}
.item .update {
  padding-top: 30px;
  font-size: 10px;
}
.item .icon {
  font-size: 17px;
  vertical-align: baseline;
  padding: 0 5px;
}
</style>