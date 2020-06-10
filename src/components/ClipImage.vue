<template>
  <div class="clipImageBox">
    <div class="canvasBox" @touchstart="startFunc" @touchmove="moveFunc">
      <canvas :width="CW" :height="CH" ref="canvas"></canvas>
      <div class="mark" v-show="ISMARK" :style="{width:MW+'px',height:MH+'px',
      left:ML+'px',top:MT+'px'}"></div>
    </div>

    <div class="buttonBox">
      <input type="file" accept="image/*" class="file" ref="file" 
      @change="changeFunc"/>
      <button @click="clickFunc">选择图片</button>
      <button @click="scaleFunc(1)">放大</button>
      <button @click="scaleFunc(0)">缩小</button>
      <button @click="saveFunc">保存图片</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    let winW = document.documentElement.clientWidth,
      font = parseFloat(document.documentElement.style.fontSize),
      canvasW = winW - 0.4 * font,
      markW = canvasW * 0.7;
    return {
      // canvas画布的大小
      CW: canvasW,
      CH: canvasW,
      // mark 遮罩层的大小
      MW: markW,
      MH: markW,
      // mark  遮罩层的位置
      ML: (canvasW - markW) / 2,
      MT: (canvasW - markW) / 2,
      // 传入的图片的大小和位置
      IW: 0,
      IH: 0,
      IL: 0,
      TT: 0,
      // 是否显示mark
      ISMARK: false
    }
  },
  methods: {
    clickFunc(){
      // 触发file选中文件的操作
      this.$refs.file.click()
    },
    changeFunc(){
      // 选中新的图片
      this.ISMARK = true
      let file = this.$refs.file.files[0]
      if(!file) return
      // 先基于FildReader进行文件的读取
      let fileExample = new FileReader()
      // 将图片文件转换为 base64 编码
      fileExample.readAsDataURL(file)
      // 转换完成时调用的函数
      fileExample.onload = ev => {
        // 创建新图片
        this.IMAGE = new Image()
        this.IMAGE.src = ev.target.result
        // 新图片创建完成后调用的函数
        this.IMAGE.onload = () => {
          this.IW = this.IMAGE.width
          this.IH = this.IMAGE.height
          // 重新按照比例计算宽高
          let n = 1
          if(this.IW > this.IH) {
            n = this.IW / this.CW
            this.IW = this.CW
            this.IH = this.IH / n
          } else {
            n = this.IH / this.CH
            this.IH = this.CH
            this.IW = this.IW / n
          }
          this.IL = (this.CW - this.IW) / 2
          this.IT = (this.CH - this.IH) / 2
          // 绘制图片
          this.drawImage()
        }
      }
    },
    // 绘制图片的方法
    drawImage() {
      // 创建2D渲染画布
      this.CTX = this.$refs.canvas.getContext('2d')
      // 清空画布
      this.CTX.clearRect(0,0,this.CW,this.CH)
      // 绘制图片
      this.CTX.drawImage(this.IMAGE,this.IL,this.IT,this.IW,this.IH)
    },
    // 放大缩小
    scaleFunc(flag) {
      if(!this.IMAGE) return
      let n = this.IW / this.IH
      if(flag) {
        this.IW += 10
        this.IH += 10/n
      } else {
        this.IW -= 10
        this.IH -= 10/n
      }
      this.drawImage()
    },
    // 拖拽图片
    startFunc(ev) {
      if(!this.IMAGE) return
      // 按下获取手指的坐标
      let point = ev.changedTouches[0]
      this.startX = point.clientX
      this.startY = point.clientY
    },
    moveFunc(ev) {
      if(!this.IMAGE) return
      let point = ev.changedTouches[0]
      this.changeX = point.clientX - this.startX
      this.changeY = point.clientY - this.startY
      if(Math.abs(this.changeX) > 10 || Math.abs(this.changeY) > 10) {
        this.IL += this.changeX
        this.IT += this.changeY
        this.drawImage()
        this.startX = point.clientX
        this.startY = point.clientY
      }
    },
    // 保存图片
    saveFunc() {
      if(!this.IMAGE) return
      // 获取指定区域的像素信息
      let imageData = this.CTX.getImageData(this.ML,this.MT,this.MW,this.MH)
      // 创建一个新的 canvas 画布，用来保存图片
      let canvas2 = document.createElement('canvas'),
          canvas2CTX = canvas2.getContext('2d');
      canvas2.width = this.MW
      canvas2.height = this.MH
      // 把像素信息放置到创建的画布中
      canvas2CTX.putImageData(imageData,0,0,0,0,this.MW,this.MH)
      // 把画布中的内容生成图片的 base64
      this.$emit('saveImage',canvas2.toDataURL('image/png'))
    }
  },
}
/* 
  设计思路:
    1.准备数据：画布大小、mark大小和位置、上传图片的大小和位置、是否显示mark层
    2.上传图片：选中图片，将其绘制到画布中
    3.实现图片的拖拽：重新绘制图片所在位置
    4.实现图片的放大和缩小：重新绘制图片的大小
    5.保存图片：把遮罩层选中的部分(像素)生成一张新的图片
*/
</script>

<style lang="less" scoped>
  .clipImageBox {
    padding: 0.2rem;
  }
  .clipImageBox .canvasBox{
    position: relative;
    overflow: hidden;
  }
  .clipImageBox .canvasBox canvas{
    display: block;
    box-sizing: border-box;
    margin: 0 auto;
    width: 7.1rem;
    height: 7.1rem;
    border: 0.02rem solid #ddd;
  }
  .clipImageBox .canvasBox .mark{
    box-sizing: border-box;
    position: absolute;
    z-index: 20;
    width: 70%;
    height: 70%;
    border: 0.02rem dashed lightcoral;
    background: rgba(0, 0, 0, 0.2);
  }
  .clipImageBox .buttonBox .file{
    display: none;
  }
  .clipImageBox .buttonBox {
    // margin-top: 0.3rem;
  }
  .clipImageBox .buttonBox button{
    margin-right: 0.2rem;
    padding: 0.2rem;
    border: none;
    font-size: 0.28rem;
    background: lightblue;
    cursor: pointer;
  }
</style>