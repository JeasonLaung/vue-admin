<style lang="scss">
.panel{
  display: flex;
  .poster-container{
    background-repeat: no-repeat;
    background-position: center;
    background-size: contain;
  }
  .lf{
  }
  .rt{
    .item{
      display: flex;
      flex-direction: column;
      cursor: pointer;
      border: 1px dashed #eee;
      font-size: 13px;
      margin-bottom: 10px;
      height: 60px;
      justify-content: center;
      align-items: center;
      &.active{
        border-style: solid;
        border-color: orangered;
        .i {
          color: orangered;
        }
      }
      img {
        width: 30px;
        height: 30px;
      }
    }
  }
}
</style>
<template>
  <div class="app-container">
    <el-form ref="form" :model="form" label-width="120px">
      <el-form-item label="活动名称">
        <el-input v-model="form.title" />
      </el-form-item>
      <el-form-item label="活动链接">
        <el-input v-model="form.url" />
      </el-form-item>
    </el-form>
    <div class="panel">
      <div class="lf">
        <div
        class="poster-container"
        :style="{
          backgroundImage: 'url(' + src + ')',
          width: width + 'px',
          height: height + 'px'
        }">
          <ResizeBox
          :isActive="true"
          :w="200"
          :aspectRatio="true"
          :h="30"
          v-on:resizing="resize"
          v-on:dragging="resize"
          >
            用户昵称
          </ResizeBox>
        </div>
      </div>
      <div class="rt">
        <div
        v-for="(item, index) in settingList"
        :key="index"
        @click="addDecoration(item.type)"
        :data-type="item.type"
        class="item"
        :class="[item.type == currentControll ? 'active' : '']">
          <i v-if="item.icon" :class="[item.icon]" style="font-size: 35px"></i>
          <img v-else-if="item.image" :src="item.image" />
          <div>{{item.title}}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// import VueQr from 'vue-qr'
import ResizeBox from 'vue-drag-resize'
const qrString = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAMgAAADICAYAAACtWK6eAAAI1klEQVR4Xu2d0ZUTRxBFm4wcgsnIIUAETsGOBEJwRnAGjM8x1lL1Zh/PXeqrHw6oVLW6r696RlpGb9Zanxa3BIG3a62PpkFkZgJZtHn7BkEypNdaCBJDbRuEIDaUdSMEqRntVoEgwUQQJAjbNApBTCA7bRCkQ2mvGgQJ5oEgQdimUQhiAtlpgyAdSnvVIEgwDwQJwjaNQhATyE4bBOlQ2qsGQYJ5IEgQtmkUgphAdtogSIfSXjUIEswDQYKwTaMQxASy0wZBOpT2qkGQYB4IEoRtGoUgJpCdNgjSobRXDYIE80CQIGzTKAQxgey0QZAOpb1qECSYB4IEYZtGIYgJZKcNgnQo7VWDIME8ECQI2zQKQUwgO20QpENprxoECeaBIEHYplEIYgLZaYMgHUp71SBIMA8ECcI2jUIQE8hOGwTpUNqrBkGCeSBIELZpFIKYQHbaIEiH0l41CBLMA0GCsE2j4oJclzrd6fZhrfVr6AeaKsjRmaWvzXs07KEXrz46MwRhB6k2UASpCBnvPxo2O4hlJUUPi9lB2EGqVXv0ixqCIAiCvEyAd7F4F6vyY7GDlIh8BUfD5hzEspA4B7Fg7DWJwkaQXihFVTQzzkE4B6lW7dG7PoIgCIJwkv4igeh2zSFW5WLr/mhm7CDsINWq5BCrImS8/2jY7CCWlcQOYsHYaxKFjSC9UHgXy8LJ0gRBaoxH7/qcg3AOUimCIBUh4/1Hw+YQy7KSors+Owg7SLVqj35RQxAEQRA+KOSDwsqCH9zPDvIKeOpDj4bNOYi6XB7Wcw5iwdhrEoWNIL1Q+BzEwsnSBEFqjEfv+pykc5JeKYIgFSHj/UfD5hDLspKiuz47CDtItWqPflFDEARBED4H4XOQygI+B3lIgMv+cNmfUh0OsUpEvoKjYXOSbllInKRbMPaaRGEjSC8UPii0cLI0QZAa49G7Pu9i8S5WpQiCVISM9x8Nm0Msy0qK7vrsIOwg1ao9+kUNQRAEQTb6oPB9lYZw/6e1vlya//rzun17pfv290f/9n3N9QWefInnj6EfnVl6BxHW/9OVTv2W26cLQnhC8U/ShZ/t6UoRZF6kCBLMDEGCsE2jEMQEstMGQTqU9qpBkGAeCBKEbRqFICaQnTYI0qG0Vw2CBPNAkCBs0ygEMYHstEGQDqW9ahAkmAeCBGGbRiGICWSnDYJ0KO1VgyDBPBAkCNs0CkFMIDttEKRDaa8aBAnmgSBB2KZRCGIC2WmDIB1Ke9UgSDAPBAnCNo1CEBPIThsE6VDaqwZBgnkgSBC2aRSCmEB22iBIh9JeNQgSzANBgrBNoxDEBLLTBkE6lPaqQZBgHggShG0ahSAmkJ02CNKhtFcNggTzQJAgbNMoBDGB7LRBkA6lvWoQJJgHggRhm0YhiAlkpw2CdCjtVYMgwTwQJAjbNOqLIO9MzWjzYwIfjV9/QGaZ1fZxt0vbZ542UyDQJIAgTVCUnUkAQc7MnWfdJIAgTVCUnUkAQc7MnWfdJIAgTVCUnUkAQc7MnWfdJIAgTVCUnUkAQc7MnWfdJIAgTVCUnUkAQc7MnWfdJHAJkvqe8OaP9NRl1+9jOW67Zvb98/vtlU/2r1c+/tvDf2n2+c88vie9Sc5Q9uy/zfvo+V2C/G5g97+1QJAcegTJsbZNQhAbyrIRgpSI9itAkFwmCJJjbZuEIDaUZSMEKRHtV4AguUwQJMfaNglBbCjLRghSItqvAEFymSBIjrVtEoLYUJaNEKREtF8BguQyQZAca9skBLGhLBshSIlovwIEyWWCIDnWtkkIYkNZNkKQEtF+BQiSywRBcqxtkxDEhrJshCAlov0KECSXCYLkWNsmIYgNZdkIQUpE+xUgSC4TBMmxtk1CEBvKshGClIj2K0CQXCYIkmNtm4QgNpRlIwQpEe1XgCC5TBAkx9o2CUFsKMtGCFIi2q8AQXKZIEiOtW0SgthQlo0QpES0XwGC5DJBkBxr2yQEsaEsGyFIiWi/grQgrmvTOkmmrnOLIM7UQr3Sgux2NfkPwYt3I0hoUTvHIEju6vYI4ly5oV4IgiCupcbV3Q0kOcQyQFxrffK0sXZxC/LHWutP0094fQVD9ztC/jWSHYQdxLQG1yNBrjdArvO8O7f3a613dx744DG3zzURBEFMaxBBHCA5xHJQnHOIxQ4i5o0gIrAXyqecgyCImDeCiMAQxAKMc5CbGG+DuzHvxM9B2EHEhcIOIgJjB7EAu/1CyLtYvItlWYFr8S6WAyQ7iIOi7/MBz0/ztcv1i6jf/zIqh1giYQQRgQ0vRxAxQAQRgQ0vRxAxQAQRgQ0vRxAxQAQRgQ0vRxAxQAQRgQ0vRxAxQAQRgQ0vRxAxQAQRgQ0vRxAxQAQRgQ0vRxAxQAQRgQ0vRxAxQAQRgQ0vRxAxQAQRgQ0vRxAxQAQRgQ0vRxAxQAQRgQ0vRxAxQAQRgQ0vRxAxQAQRgb1QfvdSOp7pj7tcl+nh191fSRhBXgnw74dz0QaNI/+jUOP1T/VtcDfm8X/SNWhcOE7j9VOqEcSHlSsrGlhyiGWAOOjavJyki3kjiAjshXLOQTSOt48UuKoJVzXRltrL1RxiGUiygxggcoglQ2QHkZF9fcBtcDfm8S6WBo13sTReP6UaQXxYOcQysORbbg0QOcSSId5+IUyfpMvP7IkewCGWFuaRh1gaoueqRhAtTwTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtQgTReI2vRhAtwm0Eub4FiFuGgOvaxJO+QOc1ZF28bq/xz3iF6JmdzVfJAAAAAElFTkSuQmCC'
// 限制高度
const limitSize = {
  height: 750,
  width: 750
}
export default {
  components: {
    // VueQr,
    ResizeBox
  },
  data() {
    return {
      limitSize,
      width: 0,
      height: 0,
      scale: 0,
      src: 'https://img.redocn.com/sheji/20171207/shangyedichanshangpuchangweixinhaibao_9044177.jpg',
      form: {
        name: ''
      },

      settingList: [
        {
          title: '用户头像',
          icon: 'el-icon-user-solid',
          type: 'avatar'
        },
        {
          title: '用户昵称',
          type: 'nickName'
        },
        {
          title: '链接二维码',
          type: 'urlQr',
          image: qrString
        }
      ],
      // 当前设置
      currentControll: '',
      config: {
      }
    }
  },

  mounted() {
    const img = new Image()
    img.src = this.src
    img.onload = (e) => {
      const { height, width } = img
      // 谁大谁恶谁正确
      if (width <= height) {
        this.scale = limitSize.height / height
        this.height = limitSize.height
        this.width = this.scale * width
      } else {
        this.scale = limitSize.width / width
        this.width = limitSize.width
        this.height = this.scale * height
      }
    }
  },

  methods: {
    addDecoration (type) {
      this.currentControll = type
    },
    resize (e) {
      console.log(e)
    }
  }
}
</script>
