<template>
  <div class="main-wrap">
    <h3>感知哈希算法-均值哈希 canvas实现demo</h3>
    <input type="file" id="myFile" @change="handleChange" />
    <template v-if="isShowOriginCanvas">
      <p>原图：</p>
      <canvas id="origin">抱歉，您的浏览器还不支持canvas。</canvas>
      <div>
        <button @click="handleHashEncode($event, 8)">获取8*8hash编码</button>
        <button @click="handleHashEncode($event, 32)">获取32*32hash编码</button>
      </div>
    </template>
    <template v-if="isShowGrayCanvas">
      <p>灰度处理：</p>
      <canvas id="hash">抱歉，您的浏览器还不支持canvas。</canvas>
      <p>hash编码：{{hash}}</p>
    </template>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        isShowOriginCanvas: false,
        isShowGrayCanvas: false,
        hash: '',
      }
    },
    methods: {
      // 将图片渲染在canvas上
      handleChange(e) {
        this.isShowOriginCanvas = true
        const file = e.target.files[0]
        const reader = new FileReader()
        reader.readAsDataURL(file)
        reader.onload = (ev) => {
          const img = new Image()
          img.src = event.target.result
          img.onload = () => {
            const myCanvas = document.getElementById('origin')
            const context = myCanvas.getContext('2d')
            myCanvas.width = img.width
            myCanvas.height = img.height
            context.drawImage(img, 0, 0)
          }
        }
      },

      // 获取图片的hash编码
      handleHashEncode(e, size = 32) {
        this.isShowGrayCanvas = true
        this.$nextTick(() => {
          const originCanvas = document.getElementById('origin')
          const hashCanvas = document.getElementById('hash')
          hashCanvas.width = size
          hashCanvas.height = size
          const context = hashCanvas.getContext('2d')
          context.drawImage(originCanvas, 0, 0, size, size)
          const imgData = context.getImageData(0, 0, size, size)
          // 获取灰度图像
          const grayArray = this.toGrayImage(imgData.data)
          let grayImgData = context.createImageData(imgData)
          grayImgData.data.forEach((item, index) => {
            grayImgData.data[index] = grayArray[index]
          })
          context.putImageData(grayImgData, 0, 0)
          // 获取hash编码
          this.hash = this.encodeHash(grayArray)
        })
      },

      // 彩色图像通过加权均值方式转灰度图像
      toGrayImage(data) {
        let newData = []
        for (let i = 0; i < data.length; i += 4) {
          const gray = Math.round(0.3 * data[i] + 0.59 * data[i + 1] + 0.11 * data[i + 2])
          newData = [...newData, gray, gray, gray, data[i + 3]]
        }
        return newData
      },

      // 计算图片均值hash编码
      encodeHash(data) {
        // 获取灰度数组
        let grayArray = []
        for (let i = 0; i < data.length; i += 4) {
          grayArray.push(data[i])
        }
        // 计算均值
        const total = grayArray.reduce((pre, cur) => {
          return pre + cur
        })
        const avg = total / grayArray.length
        // 依次对比
        const hashArray = grayArray.map((item) => {
          return item > avg ? 1 : 0
        })
        return hashArray.join('')
      }
    }
  }
</script>

<style scoped>
  .main-wrap{
    font-size: 12px;
  }
  canvas{
    display: block;
  }
  p{
    word-break: break-all;
  }
</style>
