## drawing组件文档

```
<div class="drawing-wrap" v-if="drawingStatus">
  <img-drawing :url="drawingUrl" @closeDrawing="closeDrawing"></img-drawing>
</div>

<script>
export default {
  data(){
    return {
      drawingStatus: true,
      drawingUrl: 'http://images/img/test.png'
    }
  },
  methods: {
    closeDrawing(base64){
      base64 ? alert('用户确认了编辑') : alert('用户取消了编辑') 
    }
  }
}
</script>

<style>
.drawing-wrap{
  position: fixed;
  left: 0;
  right: 0;
  top: 0; 
  bottom: 0; 
  z-index: 2000;
  /* z-index: 2000是个合适的值可以刚好不覆盖elementUi的组件 */
}
<style>
```
关于图片的读写已经设置 `image.crossOrigin = 'anonymous'` 但是由于服务端的限制依然可能存在跨域的问题，出现跨域请联系服务端开放相应限制。