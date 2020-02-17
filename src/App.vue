<template>
  <div id="app">
    <router-view />
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
 @import "assets/css/vendors/reset.css";
#app {
  font-family: "Noto Sans TC", "微軟正黑體";
  font-style: normal;
}
.color_gray {
  color: #777;
}
.color_blue {
  color: #4b6de4;
}
.color_green {
  color: #78d5d7;
}
  button,input,select,textarea{
    -webkit-appearance:none;
  }
</style>
