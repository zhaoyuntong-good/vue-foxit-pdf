<template>
  <div id="pdf-ui"></div>
</template>

<script>
  import './sdk/UIExtension.css';
  import './sdk/selUIExtension.css'
  require('./sdk/UIExtension.full.js');
  export default {
    name: 'foxit-pdf',
    props: {
      pdfSrc: {
        type: String,
        required: true
      },
      fdfData: {
        type: Object | ArrayBuffer,
        default: null
      }
    },
    data() {
      return {
        pdfui: null,
        licenseSN: 'MntXF3mTnba/COzFyzMBGfcd01CDB9hvh4k6qwYmGlWDbLkiCLFfjg==',
        licenseKey: 'PjwcmTSwMVyob7urGyG0AlYv0yI33y77QY63gymiuz3p5Bbwk4AkQXmpZkZMxSvihZTniB7qFeJvbP/hM276q8Pqh9JPJ0FTKf1kzg/T+Roe+YH9DmkG+375jOeyTPY53817HiTkog15Mq0r8DqFFhBeYXIGoI024n61ls96/CrZvx6XOjJZHORK89Aq3kRk7frKCiu5w0hN90QtfE6YfZpDrwXHhWPrGNUvfvTSLqGOVkfDXlwabhAMx/Iy9NBqvMLjNDyLARQNGB8NRsyzW2m4B7fyYm2l8obQpaZvASlgV0UbSkFU2uQjqQfIgFs5R+th5sH+2mAVDsw208KoXy4RgwakodpaLyyM1qREIDgQogNYRLDxZDBMwg5sn4oidFcuilO7bexk2HoBDdHKSCHRw4pEXVU0surbJsZ+dHB3StZmSnpUeq0a6kvj5cd9FDxPU4mUvExUi21ok+AXYil6rW9GnpphiEzu1CVgPyjG3jzNkR9hzDQfh4qS2geFUuLZH1DNWJ65+6bghVurvUirh1UkMqpOT3JHO8K/pNMHHg5J48Er54lWYtuPqpKv0wIxF2jcXdUJeXnthIncilhSmefpIs1u1fWEc0AuOUlOC+FsP6tjiy9lVi35iBg5TGVi2XNkmPUqCLfBQrcRiC//G54QeMzctknUSdT8PqX/tykOftLkRpDOii81+9u7rsW6YrOajj5uLQGAng6bA2AQK/sjyzXijf46g8Q48xJY4UdhoEuhSDqXSiVaHg5yQDl/epUw49AI2gmE1T3Um2b/opAdVKTJjl9e02PhbIZ80iogGcSYyb4Q+U0ROQsV7bClva8PAk38xTs46T2V02tW3ty4VIKd5bqJQbnM5cOtggcU7ZcA715BvHEwyGuxPWl96hza+O+zXocHS1lj/ddYE6JaR4kicuoKnNuPbIKeiOwMJXyxaGb8sGlu+ij3EmfxusOvKDKR8TGlhT39qvLWkjhn+9kqTb7PZ99NtP5/+qpddhRfY3rvhGNTaEgMq6yUD51WmB5oCiXIQfPces1pHwDdqBGe7PYRbi+3o1I7+nIVeNbYpePIXicD9SfixV97J+zEquXLDOhrBXLRyIhbdF/fXLcKpL4VNc27cKqI3WzWGNrnOSft//5EIHzVrbrOUMJ31Zs1Kci2N+5o1j0/WFeP6rpBPUmK4MTH+AXbDf1bFL4clz5r7t1CdBznU+3Xb08VPLo0W2f1dg6l6plMm97DCVX5pr0=',
        pdfLoading: null,
        Events: null,
      };
    },
    watch: {
      pdfSrc: function(){
        this.pdfLoading = this.showPdfLoading();
        this.getPdf(this.pdfSrc);
        this.closePdfLoading(this.pdfLoading);
      },
      fdfData: function(){
        this.getFdf();
      }
    },
    mounted(){
      this.Events = UIExtension.PDFViewCtrl.Events;
      this.initPdfUi();
      this.pdfLoading = this.showPdfLoading();
      this.pdfui.addViewerEventListener(this.Events.openFileSuccess, () => this.closePdfLoading(this.pdfLoading));
      this.pdfui.addViewerEventListener(this.Events.openFileFailed, () => this.closePdfLoading(this.pdfLoading));
      this.pdfui.addViewerEventListener(this.Events.addAnnotSuccess, () => this.getPdfDoc());
      this.getPdf(this.pdfSrc);
    },
    methods: {
      // 初始化pdfui
      initPdfUi(){
        this.pdfui = new UIExtension.PDFUI({
          il8n: { 
            absolutePath: '/locals',        
            lng: 'zh-CN'     
          }, 
          viewerOptions: {
            libPath: '/foxitPdf',
            jr: {
              licenseSN: this.licenseSN,
              licenseKey: this.licenseKey,
            }
          },
          renderTo: '#pdf-ui',
          template: [
            '<webpdf>',
            '  <toolbar>',
            '    <zoom-dropdown></zoom-dropdown>',
            '    <goto-prev-page-button></goto-prev-page-button>',
            '    <goto-next-page-button></goto-next-page-button>',
            '    <goto-page-input></goto-page-input>',
            '    <create-pencil-button></create-pencil-button>',
            '  </toolbar>', 
            '  <viewer></viewer>',
            '</webpdf>'
          ].join('')
        });
      },
      // loading
      showPdfLoading(){
        return this.pdfui.loading()
      },
      // 关闭loading
      closePdfLoading (pdfLoading) {
        pdfLoading.then( res => res.close())
      },
      // 加载pdf
      getPdf (pdfSrc) {
        fetch(pdfSrc).then( res => res.arrayBuffer().then( buffer => this.pdfui.openPDFByFile(buffer)))
      },
      // 获取pdfDoc
      getPdfDoc(){
        this.pdfui.getCurrentPDFDoc().then( res => this.exportFdf(res))
      },
      // 导出批注后的fdf
      exportFdf (res) {
        res.exportAnnotsToFDF().then( fdf => this.$emit('submitPdf', fdf))
      },
      // 获取传进来的fdf
      getFdf () {
        this.pdfui.getCurrentPDFDoc().then( res => res.importAnnotsFromFDF(this.fdfData))
      }
    },
    beforeDestroy(){
      // 销毁pdfui实例
      if (this.pdfui) {
        this.pdfui.removeViewerEventListener(this.Events.openFileSuccess, () => this.closePdfLoading(this.pdfLoading));
        this.pdfui.removeViewerEventListener(this.Events.openFileFailed, () => this.closePdfLoading(this.pdfLoading));
        this.pdfui.removeViewerEventListener(this.Events.addAnnotSuccess, () => this.getPdfDoc());
        this.pdfui.destroy();
        this.pdfui = null;
      }
    }
  };
</script>

<style scoped>
  #pdf-ui {
    width: 100%;
    height: 100%;
    background-color: #DDDEDE;
    overflow: auto;
  }
</style>
