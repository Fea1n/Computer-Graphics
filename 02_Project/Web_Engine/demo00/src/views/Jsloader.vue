<template>
  <div>
    <div class="mapBox">
      <!-- // 要放大的容器 -->
      <div id="mapCodeID"><img src="https://img2.baidu.com/it/u=3117014787,2706459536&fm=253&fmt=auto&app=138&f=JPEG?w=231&h=500" alt="">
      </div>
      <!-- // 小图标 -->
      <div class="iconBox" @click="enlarge"><img src="https://s4.ax1x.com/2022/01/27/7jEmTO.png" title="点击放大"></div>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      FullScreen: false, // 是否全屏
    };
  },
  mounted() {},
  methods: {
    //点击全屏事件
    enlarge() {
      // let element = document.documentElement;//全屏放大
      let element = document.getElementById("mapCodeID"); //需要全屏容器的id
      // 浏览器兼容
      if (this.FullScreen) {
        if (document.exitFullscreen) {
          document.exitFullscreen();
        } else if (document.webkitCancelFullScreen) {
          document.webkitCancelFullScreen();
        } else if (document.mozCancelFullScreen) {
          document.mozCancelFullScreen();
        } else if (document.msExitFullscreen) {
          document.msExitFullscreen();
        }
      } else {
        if (element.requestFullscreen) {
          element.requestFullscreen();
        } else if (element.webkitRequestFullScreen) {
          element.webkitRequestFullScreen();
        } else if (element.mozRequestFullScreen) {
          element.mozRequestFullScreen();
        } else if (element.msRequestFullscreen) {
          element.msRequestFullscreen();
        }
      }
      this.FullScreen = !this.FullScreen;
    },
  },
};
</script>

<style scoped>
.mapBox {
  position: relative;
}
#mapCodeID {
  width: 100%;
  height: 500px;
}
#mapCodeID img {
  width: 100%;
  height: 100%;
}
.iconBox {
  cursor: pointer;
  position: absolute;
  top: 200px;
  right: 20px;
}
</style>

