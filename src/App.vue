<template>
  <div id="app" @resize="isApp">
    <loader :isshow="getShowLoading" loaderbackground="rgba(0,0,0,0.3)"></loader>
    <div class="maincontent">
      <fixed-bg v-if="imageInfo.url && imageSetting" :imagepath="imagePrevPath +'' + imageInfo.url" :maskcolor="getGlobalStyle.contentInfo.bgcolor" :maskopacity="getGlobalStyle.contentInfo.opacity"></fixed-bg>
    </div>
    <v-content></v-content>
    <audio :src="getCurrentMusic.url" ref="myAudio"></audio>
  </div>
</template>
<script>
// import axios from 'axios'
import store from './store'
import fecth from './utils/fecth.js'
import home from './components/home.vue'
import fixedbg from './components/common/fixedbg/fixedbg.vue'
import content from './components/common/content/content.vue'
import pic from './components/pic/pic.vue'
import loader from './components/common/loader/loader.vue'
// import $ from 'jquery'

export default {
  data () {
    return {
      isgetimagebybing: store.getters.getShowBingImage
    }
  },
  name: 'app',
  components: {
    home,
    'v-content': content,
    'fixed-bg': fixedbg,
    pic,
    loader
  },
  methods: {
    fetchData () {
      // 获取壁纸信息
      const isShowBingImage = store.getters.getGlobalInfo.showBingImage
      let getbingApi = isShowBingImage ? 'http://www.daiwei.org/vue/server/home.php?inAjax=1&do=getImageByBingJson' : 'http://www.daiwei.org/vue/server/home.php?inAjax=1&do=getHomeImage'
      const hasFixedImageBg = localStorage.getItem('fixedImageBg')

      if (isShowBingImage) {
        fecth.get(getbingApi).then((res) => {
          let imageInfo = {}
          imageInfo.url = res.data.url
          imageInfo.title = res.data.title
          imageInfo.disc = res.data.disc
          store.dispatch({
            type: 'set_FixedImageInfo',
            data: imageInfo
          })
        }, (err) => {
          alert(err)
        })
      } else {
        if ((hasFixedImageBg === null || '')) {
           fecth.get(getbingApi).then((res) => {
            let imageInfo = {}
            imageInfo.url = res.data.url
            imageInfo.title = res.data.title
            imageInfo.disc = res.data.disc
            store.dispatch({
              type: 'set_FixedImageInfo',
              data: imageInfo
            })
          }, (err) => {
            alert(err)
          })
        } else {
          // 背景图片地址设置本地存储
          const getFixedImageBg = JSON.parse(localStorage.getItem('fixedImageBg'))
          if (!(getFixedImageBg === null || '')) {
            store.dispatch({
              type: 'set_FixedImageInfo',
              data: getFixedImageBg
            })
          }
        }
      }
    },
    getPlace () {
      fecth.get('http://www.daiwei.org/vue/server/home.php?inAjax=1&do=getAdress').then((res) => {
        store.dispatch({
          type: 'set_Place',
          data: res.data.data
        })
        this.getWeather(res.data.data.city)
      }, (err) => {
        alert(err)
      })
    },
    // 获取天气信息
    getWeather (city) {
      fecth.post(`http://www.daiwei.org/vue/server/home.php?inAjax=1&do=getWeather`, {place: city}).then((res) => {
        // store.dispatch({
        //   type: 'set_Place',
        //   data: res.data.data
        // })
        // this.getWeather(res.data.data.city)
        // console.log(JSON.stringify(res.data))
        store.dispatch({
          type: 'set_Weather',
          data: res.data.data.forecast
        })
      }, (err) => {
        alert(err)
      })
    },
    isApp () {
      let isTrue = false
      if (document.body.clientWidth < 768) {
        isTrue = false
      } else {
        isTrue = true
      }
      store.commit({
        type: 'setIsHigher768',
        data: isTrue
      })
      // console.log(isTrue)
    },
    shouldLoadingBg () {
      if (window.localStorage) {
        // 全局设置
        const getAllStorage = JSON.parse(localStorage.getItem('globalInfo'))
        if (!(getAllStorage === null || '')) {
          store.dispatch({
            type: 'set_GlobalInfo',
            data: getAllStorage
          })
        }
        // 背景图片地址设置本地存储
        const getFixedImageBg = JSON.parse(localStorage.getItem('fixedImageBg'))
        if (!(getFixedImageBg === null || '')) {
          store.dispatch({
            type: 'set_FixedImageInfo',
            data: getFixedImageBg
          })
        }
        return true
      } else {
        return false
      }
    },
    setAudioRef () {
      store.commit({
        type: 'setAudioEle',
        data: this.$refs.myAudio
      })
    }
  },
  watch: {
    getIsBingBg (nowval, oldval) {
      this.fetchData()
      localStorage.setItem('globalInfo', JSON.stringify(store.getters.getGlobalInfo))
    }
  },
  computed: {
    imagePrevPath () {
      return store.getters.getGlobalInfo.showBingImage ? 'http://www.bing.com' : ''
    },
    getGlobalStyle () {
      return store.getters.getGlobalInfo
    },
    imageInfo () {
      return store.getters.getFixedImageInfo
    },
    imageSetting () {
      return store.getters.getFixedBgInfo
    },
    getIsBingBg () {
      return store.getters.getGlobalInfo.showBingImage
    },
    getShowLoading () {
      return store.getters.getShowLoading
    },
    getCurrentMusic () {
      return store.getters.getCurrentAudio
    }
  },
  mounted () {
    // this.$toast()
      // 设置audio 的refs
      this.setAudioRef()
    // this.$nextTick(() => {
      // 是否在线加载壁纸
      this.shouldLoadingBg()
      // 获取用户地址
      this.getPlace()
      // 是否小于768
      this.isApp()
      // 加载数据
      this.fetchData()
      // 挂载 onresize事件
      window.onresize = () => {
        this.isApp()
      }
      localStorage.setItem('globalInfo', JSON.stringify(store.getters.getGlobalInfo))
    // })
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
@import 'common/stylus/border-1px/index.styl'
@import '/static/font-icon/style.css'
  audio
    display:none
  body,html
    margin:0
    padding:0
    box-sizing:border-box
    background:#f0f0f0f0
    user-select:none
    -webkit-tap-highlight-color: transparent
  #app 
    position: fixed
    top: 0
    left: 0
    bottom: 0
    right:0
    background:transparent
</style>
