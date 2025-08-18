---
title: uniapp打包app时使用html2canvas画图并预览
date: 2025-04-07 11:38:20
tags: [uniapp, vue2, app, html2canvas, renderjs, 打包]
---

hbuilderx 3.1.4
vue2 项目
原生 App-本地打包 生成本地打包 App 资源

```json
<!-- package.json -->

"html2canvas": "^1.2.2"
```

```vue
<!-- src/pages/demo.vue -->

<template>
  <view v-for="(item, index) in 3">
    <view :id="'id-' + index" @click.stop="imgClick('#id-' + index)">
      <!-- img标签不可用 -->
      <!-- <img /> -->

      <!-- image标签图片不清晰，rich-text比较清晰 -->
      <!-- rich-text图片不会自动打包，这里v-show="0"让程序把图片加入打包，或者打包后手动再加入图片 -->
      <image v-show="0" src="../../static/image.jpg" mode="widthFix"></image>

      <!-- rich-text可用本地图片或网络图片 -->
      <rich-text
        :nodes="`<img class='rich-text-class' style='width: 100%; height: 100%;' src='src/static/image.jpg' />`"
      ></rich-text>
      <!-- <rich-text
        :nodes="`<img class='' style='' src='${imageUrl}' />`"
      ></rich-text> -->

      <view class="absolute">文字文字文字文字文</view>
      <view class="absolute">元素要被id包裹在内</view>
    </view>
  </view>

  <PreviewImage :images="genPic" ref="previewImage" />
  <html2canvas
    ref="html2canvas"
    :domId="domId"
    @renderFinish="renderFinish"
  ></html2canvas>
  <!-- <viewer :images="genPic">
    <image v-for="src in genPic" :key="src" :src="src" />
  </viewer> -->
</template>

<script>
// import "viewerjs/dist/viewer.css";
// import { api as viewerApi } from "v-viewer";
// import html2canvas from "html2canvas";
import PreviewImage from "../../components/previewImg.vue";
import html2canvas from "../../components/html2canvas.vue";
export default {
  components: {
    html2canvas,
    PreviewImage,
  },
  data() {
    return {
      domId: "",
      genPic: [],
    };
  },
  methods: {
    imgClick(id) {
      this.$nextTick(() => {
        this.domId = "";
        setTimeout(() => {
          this.domId = id;
        }, 100);
      });

      return;
      // 一些踩坑

      const view = uni.createSelectorQuery().select(id);
      const view2 = document.querySelector(id);

      html2canvas(document.querySelector(id)).then((canvas) => {
        // document.body.appendChild(canvas)
        canvas.toBlob((blob) => {
          const url = URL.createObjectURL(blob);
          uni.previewImage({
            urls: [url],
          });
        });
      });
    },
    renderFinish(filePath) {
      this.genPic = [filePath];
      this.$refs.previewImage.open(0);

      return;
      // 一些踩坑

      // v-viewer用不了
      const $viewer = viewerApi({
        options: {
          transition: false,
          tooltip: false,
          button: false,
          navbar: false,
          title: false,
          toolbar: false,
        },
        images: this.genPic,
      });
    },
  },
};
</script>
```

```vue
<!-- src/components/html2canvas.vue -->
<!-- 不懂renderjs，能用 -->

<template>
  <view>
    <view class="html2canvas" :prop="domId" :change:prop="html2canvas.create">
      <slot></slot>
    </view>
  </view>
</template>

<script>
export default {
  name: "html2canvas",
  props: {
    domId: {
      type: String,
      required: true,
    },
  },
  methods: {
    async renderFinish(url) {
      try {
        this.$emit("renderFinish", url);
      } catch (e) {
        //TODO handle the exception
        console.log("html2canvas error", e);
      }
    },
    showLoading() {
      uni.showToast({
        title: "正在生成",
        icon: "none",
        mask: true,
        duration: 100000,
      });
    },
    hideLoading() {
      uni.hideToast();
    },
  },
};
</script>

<script module="html2canvas" lang="renderjs">
import html2canvas from "html2canvas";
export default {
  methods: {
    async create(domId) {
      try {
        if (domId == "") {
          return;
        }
        // this.$ownerInstance.callMethod('showLoading', true);
        const timeout = setTimeout(async () => {
          const shareContent = document.querySelector(domId);
          const canvas = await html2canvas(shareContent, {
            // windowWidth: Window.innerWidth,
            // windowHeight: Window.innerHeight,
            width: shareContent.offsetWidth, //设置canvas尺寸与所截图尺寸相同，防止白边
            height: shareContent.offsetHeight, //防止白边
            logging: true,
            useCORS: true,
            allowTaint: true,
            // scale: Window.devicePixelRatio,
          });
          canvas.toBlob((blob) => {
            const url = URL.createObjectURL(blob);
            this.$ownerInstance.callMethod("renderFinish", url);
          });
          // const base64 = canvas.toDataURL('image/png');
          // this.$ownerInstance.callMethod('renderFinish', base64);
          // this.$ownerInstance.callMethod("hideLoading", true);
          clearTimeout(timeout);
        }, 100);
      } catch (error) {
        console.log(error);
      }
    },
  },
};
</script>
```

```vue
<!-- src/components/previewImg.vue -->
<!-- ai写的一个图片预览，效果不够理想 -->
<!-- uni.previewImage用不了 -->
<!-- v-viewer用不了 -->

<template>
  <div v-if="isVisible" class="preview-container">
    <div class="mask" @click="close"></div>
    <div class="preview-content">
      <img
        :src="images[currentIndex]"
        @click="rotateImage"
        @touchstart="handleTouchStart"
        @touchmove="handleTouchMove"
        @touchend="handleTouchEnd"
        @mousedown="handleMouseDown"
        @mousemove="handleMouseMove"
        @mouseup="handleMouseUp"
        :style="{
          transform: `translate(${position.x}px, ${position.y}px) scale(${scale}) rotate(${rotate}deg)`,
          width: '100vw',
          height: 'auto',
          objectFit: 'contain',
          transition: isDragging
            ? 'transform 0.05s ease'
            : 'transform 0.05s ease',
        }"
      />
      <!-- <viewer
        :options="{
          transition: false,
          tooltip: false,
          button: false,
          navbar: false,
          title: false,
          toolbar: false,
        }"
        :images="images"
        @inited="inited"
        class="viewer"
        ref="viewer"
      >
        <template #default="scope">
          <image v-for="src in scope.images" :src="src" :key="src" />
          {{ scope.options }}
        </template>
      </viewer> -->
      <!-- <div class="toolbar">
        <button @click="zoomIn">放大</button>
        <button @click="zoomOut">缩小</button>
        <button @click="rotateImage">旋转</button>
      </div> -->
    </div>
  </div>
</template>

<script>
// import 'viewerjs/dist/viewer.css';
// import { component as Viewer } from 'v-viewer';
export default {
  props: {
    images: {
      type: Array,
      required: true,
    },
    initialIndex: {
      type: Number,
      default: 0,
    },
  },
  //   components: {
  //     Viewer,
  //   },
  data() {
    return {
      isVisible: false,
      currentIndex: this.initialIndex,
      scale: 1,
      rotate: 0,
      position: { x: 0, y: 0 },
      lastPosition: { x: 0, y: 0 },
      isDragging: false,
      touches: [],
      initialDistance: 0,
      minScale: 1, // 最小缩放比例
      maxScale: 3, // 最大缩放比例
    };
  },
  methods: {
    // inited(viewer) {
    //   this.$viewer = viewer;
    // },
    open(index) {
      this.currentIndex = index;
      this.isVisible = true;
      this.resetImage();
      document.body.style.overflow = "hidden";
    },
    close() {
      this.isVisible = false;
      document.body.style.overflow = "auto";
    },
    zoomIn() {
      const newScale = Math.min(this.maxScale, this.scale + 0.1);
      this.scale = newScale;
    },
    zoomOut() {
      const newScale = Math.max(this.minScale, this.scale - 0.1);
      this.scale = newScale;
    },
    rotateImage() {
      return;
      this.rotate += 90;
    },
    handleTouchStart(event) {
      this.touches = event.touches;

      if (this.touches.length === 2) {
        // Pinch gesture started
        this.initialDistance = this.getDistance(
          this.touches[0],
          this.touches[1]
        );
      } else if (this.touches.length === 1) {
        // Drag gesture started
        this.isDragging = true;
        this.lastPosition = {
          x: this.touches[0].clientX - this.position.x,
          y: this.touches[0].clientY - this.position.y,
        };
      }
    },
    handleTouchMove(event) {
      event.preventDefault();
      this.touches = event.touches;

      if (this.touches.length === 2) {
        // Handle pinch zoom with increased sensitivity
        const currentDistance = this.getDistance(
          this.touches[0],
          this.touches[1]
        );
        const scaleDiff = (currentDistance - this.initialDistance) * 0.015; // 增加灵敏度
        const newScale = Math.max(
          this.minScale,
          Math.min(this.maxScale, this.scale + scaleDiff)
        );
        this.scale = newScale;
        this.initialDistance = currentDistance;
      } else if (this.touches.length === 1 && this.isDragging) {
        // Handle drag with increased responsiveness
        this.position = {
          x: this.touches[0].clientX - this.lastPosition.x,
          y: this.touches[0].clientY - this.lastPosition.y,
        };
      }
    },
    handleTouchEnd() {
      this.isDragging = false;
      this.touches = [];
    },
    handleMouseDown(event) {
      this.isDragging = true;
      this.lastPosition = {
        x: event.clientX - this.position.x,
        y: event.clientY - this.position.y,
      };
    },
    handleMouseMove(event) {
      if (this.isDragging) {
        this.position = {
          x: event.clientX - this.lastPosition.x,
          y: event.clientY - this.lastPosition.y,
        };
      }
    },
    handleMouseUp() {
      this.isDragging = false;
    },
    getDistance(touch1, touch2) {
      const dx = touch1.clientX - touch2.clientX;
      const dy = touch1.clientY - touch2.clientY;
      return Math.sqrt(dx * dx + dy * dy);
    },
    resetImage() {
      this.scale = 1;
      this.rotate = 0;
      this.position = { x: 0, y: 0 };
      this.lastPosition = { x: 0, y: 0 };
      this.isDragging = false;
      this.touches = [];
      this.initialDistance = 0;
    },
  },
};
</script>

<style scoped>
.preview-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.mask {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 1);
}

.preview-content {
  position: relative;
  z-index: 1001;
}

img {
  max-width: 100vw;
  width: 100vw;
  height: auto;
  object-fit: contain;
  cursor: pointer;
  user-select: none;
  touch-action: none;
}

.toolbar {
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 10px;
}

button {
  padding: 5px 10px;
  background-color: #fff;
  border: none;
  cursor: pointer;
}
</style>
```
