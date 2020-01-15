<template>
  <div>
    <div :style="{ backgroundImage: 'url(/pm25in.jpg)' }" class="showImgLarge border">
      <div class="lightsContainer" @drop.self.prevent="drop()"></div>
    </div>
    <div class="rightShow">
      <img src="/warningLights.png" alt="" draggable="">
    </div>
    <div :style="{ backgroundImage: 'url(/pm25in.jpg)' }" class="showImgMedium border">
      <div class="lightsContainer" @drop.self.prevent="drop()"></div>
    </div>
    <div :style="{ backgroundImage: 'url(/pm25in.jpg)' }" class="showImgSmall border">
      <div class="lightsContainer" @drop.self.prevent="drop()"></div>
    </div>
  </div>
</template>
<script>
export default {
  data () {
    return {
      imgRadioArr: [] // 每个图片容器的缩放比例
    }
  },
  created () {
    this.initData()
  },
  methods: {
    drop () {
    },
    loadingImg (url) {
      return new Promise((resolve, reject) => {
        const img = new Image()
        img.src = url
        if (url === '') {
          reject(new Error('图片不存在'))
        }
        img.onload = function () {
          resolve(this)
        }
      })
    },
    async initData () {
      this.imgRadioArr = []
      const img = await this.loadingImg('/pm25in.jpg')
      const imgContainerArr = document.getElementsByClassName('border')
      const lightsContainerArr = document.getElementsByClassName('lightsContainer')
      for (let index = 0; index < imgContainerArr.length; index++) {
        const heightRadio = imgContainerArr[index].offsetHeight / img.height
        const autoWidth = img.width * heightRadio
        if (autoWidth > imgContainerArr[index].offsetWidth) {
          this.imgRadioArr[index] = imgContainerArr[index].offsetWidth / img.width
        } else {
          this.imgRadioArr[index] = heightRadio
        }
        const radioWidth = img.width * this.imgRadioArr[index]
        const radioHeight = img.height * this.imgRadioArr[index]
        lightsContainerArr[index].style.width = radioWidth + 'px'
        lightsContainerArr[index].style.height = radioHeight + 'px'
        lightsContainerArr[index].style.top = (imgContainerArr[index].offsetHeight - radioHeight) / 2 + 'px'
        lightsContainerArr[index].style.left = (imgContainerArr[index].offsetWidth - radioWidth) / 2 + 'px'
      }
    }
  }
}
</script>

<style lang="scss" scoped>
  .showImg{
    &Large{
      width:1280px;
      height:1024px;
    }
    &Medium{
      width:1024px;
      height:768px;
    }
    &Small{
      width:800px;
      height:600px;
    }
  }
  .border{
    margin:30px 10px;
    border: 1px dashed rgb(156, 81, 255);
    border-radius: 10px;
    background-position: center;
    background-size: contain;
    background-repeat: no-repeat;
    position: relative;
  }
  .lightsContainer{
    position: absolute;
  }
  .rightShow{
    float: right;
    margin-top: 200px;
  }
</style>
