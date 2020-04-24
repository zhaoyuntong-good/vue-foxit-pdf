<template>
  <div id="app">
    <foxit-pdf :pdfSrc="pdfSrc" :fdfData="fdfData" @submitPdf="submitPdf"></foxit-pdf>
  </div>
</template>

<script>
  
  import foxitPdf from './components/plugins/foxit-pdf'
  export default {
    name: 'App',
    components: {
      foxitPdf
    },
    data() {
      return {
        pdfSrc: '/FoxitPDFSDKforWeb_DemoGuide.pdf',
        fdfData: null,
      };
    },
    created(){
      setTimeout(() => {
        fetch('/comments (1).fdf').then( res => res.arrayBuffer().then( buffer => this.fdfData = buffer))
      },5000)
    },
    methods: {
      submitPdf (val) {
        console.log(val)
      }
    }
  }
</script>

<style>
* {
  margin: 0;
  padding: 0;
}
#app {
  width: 100vw;
  height: 100vh;
}
.back{
    position: fixed;
    top: 20px;
    right: 20px;
    cursor: pointer;
  }
</style>
