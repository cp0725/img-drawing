<!--
 * @Author: 常培
 * @Date: 2019-04-21 01:11:16
 * @LastEditTime: 2019-04-21 11:27:05
 * @Description: App.vue
 -->
<template>
  <div>
    <ul class="img-list">
      <li class="img-item">
        <span class="title" @click="openDrawing('assets/images/img_01.jpeg')">编辑图片img_01（800 × 500）</span>
        <img src="assets/images/img_01.jpeg" alt="img_01">
      </li>
      <li class="img-item">
        <span class="title" @click="openDrawing('assets/images/img_02.jpeg')">编辑图片img_02（720 × 926）</span>
        <img src="assets/images/img_02.jpeg" alt="img_02">
      </li>
      <li class="img-item">
        <span class="title" @click="openDrawing('assets/images/img_03.jpeg')">编辑图片img_03（1600 × 1200）</span>
        <img src="assets/images/img_03.jpeg" alt="img_03">
      </li>
    </ul>
    <div>
      <img class="imgBase64" :src="imgBase64" v-show="imgBase64" alt="imgBase64">
    </div>
    <div class="drawing-wrap" v-if="drawingStatus">
      <img-drawing :url="drawingUrl" @closeDrawing="closeDrawing"></img-drawing>
    </div>
  </div>
</template>

<script>
import imgDrawing from '@/components/drawing'
export default {
  data() {
    return {
      drawingStatus: false,
      drawingUrl: '',
      imgBase64: ''
    }
  },
  methods: {
    openDrawing(url){
      this.drawingUrl = url
      this.drawingStatus = true
    },
    closeDrawing(imgBase64){
      this.imgBase64 = imgBase64
      this.drawingStatus = false      
    }
  },
  components: {
    imgDrawing
  }
}
</script>

<style scoped lang="scss">
.img-list{
  padding: 50px;
  .img-item{
    display: flex;
    align-items: center;
    padding: 6px;
    border: 1px solid #ccc;
    &:hover{
      background: #fafafa;
    }
    .title{
      width: 250px;
      cursor: pointer;
      &:hover{
        color: royalblue;
      }
    }
    img{
      max-height: 80px;
    }
  }
}
.imgBase64{
  max-width: 800px;
}
.drawing-wrap{
  position: fixed;
  left: 0;
  right: 0;
  top: 0; 
  bottom: 0; 
  z-index: 2000;
  /* z-index: 2000是个合适的值可以刚好不覆盖elementUi的组件 */
}
</style>