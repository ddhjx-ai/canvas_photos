<template>
  <div id="app">
    <Clip-image @saveImage='saveImage' v-if="!isSave"></Clip-image>
    <div class="resultBox" @click="isSave = false" v-else>
      <img :src="data" alt="">
    </div>
  </div>
</template>

<script>
import './assets/css/base.css'
import ClipImage from './components/ClipImage'

export default {
  name: 'App',
  data() {
    return{
      isSave: false,
      data: ''
    }
  },
  components: {
    ClipImage
  },
  methods: {
    saveImage(payload) {
      this.isSave = true
      this.data = payload
    }
  },
}
</script>

<style lang='less'>
  .resultBox{
    padding: 0.2rem;
  }

  .resultBox img{
    display: block;
    box-sizing: border-box;
    width: 7.1rem;
    height: 7.1rem;
    border: 0.02rem solid #ddd;
  }
</style>
