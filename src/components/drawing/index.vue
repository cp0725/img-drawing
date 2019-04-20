<template>
  <div class="drawing-wrap">
    <div class="tool-wrap">
      <div>
        <el-tooltip content="矩形">
          <el-button @click="drawingType = 'rectangle'" circle><img src="./img/jx.png"></el-button>
        </el-tooltip>
        <el-tooltip content="多边形" v-if="false">
          <el-button @click="drawingType = 'polygon'" circle v-if="false"><img src="./img/dbx.png"></el-button>
        </el-tooltip>
        <el-tooltip content="自定义路径">
          <el-button @click="drawingType = 'route'" circle><img src="./img/hb.png"></el-button>
        </el-tooltip>
        <el-tooltip content="半径画圆">
          <el-button @click="drawingType = 'radius'" circle><img src="./img/bj.png"></el-button>
        </el-tooltip>
        <el-tooltip content="直径画圆">
          <el-button @click="drawingType = 'diameter'" circle><img src="./img/zj.png"></el-button>
        </el-tooltip>
        <el-tooltip content="文字">
          <el-button @click="drawingType = 'font'" circle><img src="./img/wz.png"></el-button>
        </el-tooltip>
        <i class="border"></i>
        <el-tooltip content="清除">
          <el-button @click="clear" circle><img src="./img/qc.png"></el-button>
        </el-tooltip>
        <el-tooltip content="画笔粗细">
          <el-button v-popover:popover circle><img src="./img/cx.png"></el-button>
        </el-tooltip>
        <el-popover ref="popover" placement="bottom" width="50" trigger="click">
          <el-slider v-model="lineWidth" :min="1" :max="10" @change="lineWidthChange" style="padding: 10px 0 10px 38px" vertical height="150px"></el-slider>
        </el-popover>
        <el-tooltip content="颜色">
          <el-color-picker v-model="strokeStyle" @change="strokeStyleChange" class="color-picker" size="mini"></el-color-picker>
        </el-tooltip>
        <i class="border"></i>
        <el-tooltip content="撤销">
          <el-button @click="revokeAndRecovery(-1)" circle><img src="./img/ht.png"></el-button>
        </el-tooltip>
        <el-tooltip content="重做">
          <el-button @click="revokeAndRecovery(1)" circle><img src="./img/cz.png"></el-button>
        </el-tooltip>
      </div>
      <div class="btn">
        <span class="info">{{drawingType ? info[drawingType] : '请选择左侧工具开始标记'}}</span>
        <i class="border"></i>
        <el-button @click="drawingSure" type="success">确认</el-button>
        <el-button @click="drawingCancel">取消</el-button>
      </div>
    </div>
    <div class="canvas-wrap" ref="canvasWrap">
      <canvas class="canvas" ref="canvas" @mousedown="mousedown" @mouseup="mouseup" @mousemove="mousemove" @click="canvasClick">你的设备不支持绘制功能请使用现代浏览器</canvas>
    </div>
    <div v-show="textareaShow" class="textarea-wrap" ref="textarea">
      <el-input type="textarea" :rows="3" placeholder="请输入内容" v-model="textareaText"></el-input>
      <div class="btn-wrap">
        <el-button @click="fontSure" type="success" size="mini">确认</el-button>
        <el-button @click="fontCancel" size="mini">取消</el-button>
      </div>
    </div>
  </div>
</template>

<script>
import errImg from './base64'
export default {
  data() {
    return {
      info: {
        rectangle: '拖拽绘制矩形',
        polygon: '多边形',
        route: '长按鼠标左键绘制自由路径',
        radius: '以半径拖拽画圆',
        diameter: '以直径拖拽画圆',
        font: '点击画布编辑文字',
        clear: '已清楚你的全部绘制',
        revoke: '撤销操作',
        recovery: '恢复撤销',
        thickness: '请调整画笔粗细',
        color: '请选择颜色'
      },
      canvas: null,
      downX: 0,
      downY: 0,
      downMs: 0,
      drawingType: '',
      drawingStatus: false,
      imgData: [],
      imgDataIndex: 0,
      textareaText: '',
      textareaShow: false,
      lineWidth: 2,
      strokeStyle: '#ff0000'
    }
  },
  mounted() {
    this.canvas = this.$refs.canvas.getContext('2d')
    this.drawingImg()
  },
  methods: {
    // push历史记录list
    pushCanvasData(){
      const W = this.$refs.canvas.width
      const H = this.$refs.canvas.height
      this.imgData = this.imgData.slice(0, this.imgDataIndex + 1)
      this.imgData.push(this.canvas.getImageData(0, 0, W, H))
      this.imgDataIndex = this.imgData.length - 1
    },
    // 设置canvas最大尺寸是img尺寸最小尺寸是100%
    setCanvasAttr(imgW, imgH){
      const canvasWrapW = this.$refs.canvasWrap.offsetWidth
      const canvasWrapH = this.$refs.canvasWrap.offsetHeight
      this.$refs.canvas.width = imgW < canvasWrapW ? canvasWrapW : imgW
      this.$refs.canvas.height = imgH < canvasWrapH ? canvasWrapH : imgH
      return {
        x: (this.$refs.canvas.width - imgW) / 2,
        y: (this.$refs.canvas.height - imgH) / 2,
        w: imgW,
        h: imgH
      }
    },
    drawingImg(){
      const image = new Image()
      image.crossOrigin = 'anonymous'
      image.src = this.url
      image.onload = eve => {
        const position = this.setCanvasAttr(image.width, image.height) 
        this.canvas.drawImage(image, position.x, position.y, position.w, position.h)
        this.setStyle()
        this.pushCanvasData()
      }
    },
    setStyle(){
      this.canvas.font = "14px Arial"
      this.canvas.fillStyle = '#ff0000'
      this.canvas.strokeStyle = '#ff0000'
      this.canvas.lineWidth = 2
    },
    lineWidthChange(num){
      this.canvas.lineWidth = num
    },
    strokeStyleChange(color){
      this.canvas.fillStyle = color
      this.canvas.strokeStyle = color
    },
    mousedown(event){
      if(this.drawingType == ''){ return }
      this.downMs = +new Date
      this.drawingStatus = true
      this.downX = event.offsetX
      this.downY = event.offsetY
      this.canvas.beginPath()
    },
    mouseup(event){
      const interval = +new Date - this.downMs > 20 && this.downMs 
      if(interval){
        this.pushCanvasData()
      }
      if(this.drawingType != ''){
        this.drawingStatus = false
      }
    },
    mousemove(event){
      if(!this.drawingType || !this.drawingStatus){return}
      this.canvas.clearRect(0, 0, this.$refs.canvas.width, this.$refs.canvas.height)
      this.canvas.putImageData(this.imgData[this.imgDataIndex], 0, 0)
      if(this.drawingType == 'rectangle'){ // 矩形
        this.rectangle(event.offsetX, event.offsetY)
      }else if(this.drawingType == 'radius'){ // 半径
        this.radius(event.offsetX, event.offsetY)
      }else if(this.drawingType == 'diameter'){ // 直径
        this.diameter(event.offsetX, event.offsetY)
      }else if(this.drawingType == 'route'){ // 自定义路径
        this.route(event.offsetX, event.offsetY)
      }else if(this.drawingType == 'polygon'){ // 多边形
        this.polygon(event.offsetX, event.offsetY)
      }
    },
    canvasClick(event){
      if(this.drawingType != 'font'){ return }
      this.textareaShow = true
      this.downX = event.offsetX
      this.downY = event.offsetY
      this.$refs.textarea.style.left = `${event.clientX}px`
      this.$refs.textarea.style.top = `${event.clientY}px` 
    },
    fontCancel(){
      this.textareaText = ''
      this.textareaShow = false
    },
    fontSure(){
      let X = 0, Y = 0
      for(let i = 0, length = this.textareaText.length; i < length; i++){
        X = (i % 11) * 15 + 12 // 15是文字水平间隔 12用来解决padding造成的文字跳动
        Y = (Math.floor(i / 11) + 1 ) * 20 + 2  // 20是文字纵向间隔 2用来解决padding造成的文字跳动
        this.canvas.fillText(this.textareaText[i], X + this.downX, Y + this.downY) 
      }
      this.fontCancel()
      this.pushCanvasData()
    },
    rectangle(X, Y){ // 矩形
      this.canvas.strokeRect(this.downX, this.downY, X - this.downX, Y - this.downY)
    },
    radius(X, Y){ // 半径
      const radius = Math.sqrt( Math.pow(X - this.downX, 2) + Math.pow(Y - this.downY, 2) )
      this.canvas.beginPath()
      this.canvas.arc(this.downX, this.downY, radius, 0, Math.PI*2, true)
      this.canvas.stroke()
      this.canvas.closePath()
    },
    diameter(X, Y){ // 直径
      const radius = Math.sqrt( Math.pow(X - this.downX, 2) + Math.pow(Y - this.downY, 2) ) / 2
      const dotX = (X - this.downX) / 2 + this.downX 
      const dotY = (Y - this.downY) / 2 + this.downY
      this.canvas.beginPath()
      this.canvas.arc(dotX, dotY, radius, 0, Math.PI*2, true)
      this.canvas.stroke()
      this.canvas.closePath()
    },
    route(X, Y){ // 自定义路径
      this.canvas.lineTo(X, Y)
      this.canvas.stroke()
    },
    polygon(X, Y){ // 多边形（直线）
      this.canvas.beginPath()
      this.canvas.moveTo(this.downX, this.downY)
      this.canvas.lineTo(X, Y)
      this.canvas.stroke()
    },
    revokeAndRecovery(n){
      this.imgDataIndex += n
      this.imgDataIndex = this.imgDataIndex < 0 ? 0 : this.imgDataIndex
      this.imgDataIndex = this.imgDataIndex == this.imgData.length ? this.imgData.length-1 : this.imgDataIndex
      this.canvas.putImageData(this.imgData[this.imgDataIndex], 0, 0)
    },
    clear(){
      this.imgData = this.imgData.slice(0, this.imgDataIndex + 1)
      this.imgData.push(this.imgData[0])
      this.imgDataIndex = this.imgData.length - 1
      this.canvas.putImageData(this.imgData[this.imgDataIndex], 0, 0)
    },
    signStatus(){
      const status = this.imgData.length == 1 || this.imgDataIndex == 0
      return {
        status,
        info: status ? '未对画布做任何修改' : '用户对画布做了修改'
      }
    },
    eleConfirm(title, type){ // await
      return this.$confirm(title, '提示', {
        type
      }).then(() => true)
        .catch(() => false)
    },
    async drawingSure(){
      const base64 = this.$refs.canvas.toDataURL()
      if(this.signStatus().status){
        const status = await this.eleConfirm('未对资源做任何标记确定要保存吗？', 'warning')
        return status && this.$emit('closeDrawing', base64)
      }
      this.$emit('closeDrawing', base64)
    },
    async drawingCancel(){
      if(this.signStatus().status){
        return this.$emit('closeDrawing')
      }
      const status = await this.eleConfirm('确定要丢弃你的标记吗？', 'warning')
      status && this.$emit('closeDrawing')
    }
  },
  props: {
    url: {
      type: String,
      default: errImg.img
    }
  }
}
</script>

<style scoped lang="scss">
.drawing-wrap{
  width: 100%;
  height: 100%;
  background: #f1f1f1;
  display: flex;
  flex-direction: column;
}
.tool-wrap{
  height: 50px;
  padding: 0 20px;
  display: flex;
  justify-content: space-between;
  flex-shrink: 0;
  & > div{
    display: flex;
    align-items: center;
  }
  img{
    width: 14px;
  }
  button{
    padding: 6px;
    margin: 0 10px 0 0;
    height: 28px;
  }
  .btn{
    button{
      padding: 6px 20px;
    }    
  }
  .border{
    width: 2px;
    height: 16px;
    background: #ccc;
    border-right: 1px solid #e5e5e5;
    margin: 0 16px 0 6px;
  }
  .info{
    margin-right: 10px;
    color: #888;
  }
  .color-picker{
    width: 28px;
    height: 28px;
    border: 1px solid #dcdfe6;
    background: #fff;
    border-radius: 50%;
    margin-right: 10px;
    /deep/ .el-color-picker__trigger{
      border: none;
    }
    /deep/ .el-color-picker__trigger{
      transform: translate(-1px, -1px) scale(.7, .7);
    }
  }
}
/deep/ .el-slider{
  padding-left: 30px;
}
.canvas-wrap{
  flex: 1;
  background: url("./img/back.png");
  background-size: 30px;
  overflow: auto;
  .canvas{
    display: block;
    // width: 100%;
    // height: 100%;
  }
}
.btn-wrap{
  height: 50px;
}
.textarea-wrap{
  position: absolute;
  width: 180px;
  /deep/ .el-textarea__inner{
    padding: 6px 10px;
    background: rgba(255,255,255,0);
    border: 1px solid red;
  }
  .btn-wrap{
    text-align: right;
    height: 28px;
    .el-button--mini{
      padding: 3px;
      margin-top: 3px;
    }
  }
}
</style>
