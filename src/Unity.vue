<template>
  <div class="webgl-content">
    <div :id="containerId" v-bind:style="{ width: width + 'px', height: height + 'px' }"></div>
    <div class="logo Dark" v-if="loaded == false"></div>
    <div class="progress Dark" v-if="loaded == false">
      <div class="empty" style="width: 0%;"></div>
      <div class="full" v-bind:style="{ width: progress * 100 + '%'}"></div>
    </div>
    <div class="footer" v-if="hideFooter !== true">
      <div class="webgl-logo"></div>
      <div class="fullscreen" @click.prevent="fullscreen"></div>
    </div>
  </div>
</template>

<script>
  import Vue from 'vue'

  export default {
    props: ['src', 'module', 'width', 'height', 'externalProgress', 'unityLoader', 'hideFooter'],
    name: 'UnityWebGL',
    data () {
      return {
        containerId: 'unity-container-' + Number(Math.random().toString().substr(3, length) + Date.now()).toString(36),
        gameInstance: null,
        loaded: false,
        progress: 0,
        error: null
      }
    },
    methods: {
      fullscreen () {
        this.gameInstance.SetFullscreen(1)
      },
      message(gameObject, method, param) {
        if (param === null) {
          param = ''
        }
        if (this.gameInstance !== null){
          this.gameInstance.SendMessage(gameObject, method, param)
        } else {
          console.warn('vue-unity-webgl: you\'ve sent a message to the Unity content, but it wasn\t instantiated yet.')
        }
      }
    },
    beforeMount() {

      if (!this.eventBus) {
        this.eventBus = new Vue({
          data: {
            ready: false,
            load: false
          }
        })
      }

      if (typeof UnityLoader === 'undefined' && this.unityLoader && !this.eventBus.load) {
        const script = document.createElement('SCRIPT')
        script.setAttribute('src', this.unityLoader)
        script.setAttribute('async', '')
        script.setAttribute('defer', '')
        document.body.appendChild(script)
        this.eventBus.load = true
        script.onload = () => {
          this.eventBus.ready = true
          this.eventBus.$emit('onload')
        }
      } else {
        this.eventBus.ready = true
        this.eventBus.load = true
      }
    },
    beforeDestroy: function () {
      console.log("navigate off!");
      if (this.gameInstance !== null){
        this.gameInstance.Quit(function() {
             console.log("quit webgl Unity instance!");
        });
        this.gameInstance = null;
        
        //unity 2020
      //  this.gameInstance
      //   .Quit()
      //   .then(() => {
      //       // Do whatever your app needs here
      //       console.log("quit webgl Unity instance was a success!");
      //       this.gameInstance = null;
      //       //this.playState$.next(null);
      //   })
      //   .catch(() => {
      //     console.log("Quitting Unity wasn't a success 😖");
      //   });
      
      }
    },
    mounted () {
      const instantiate = () => {
        if (typeof UnityLoader === 'undefined') {
          let error = 'The UnityLoader was not defined, please add the script tag ' +
            'to the base html and embed the UnityLoader.js file Unity exported or use "unityLoader" attribute for path to UnityLoader.js.'
          console.error(error)
          this.error = error
          return
        }
        if (this.src === null) {
          let error = 'Please provice a path to a valid JSON in the "src" attribute.'
          console.error(error)
          this.error = error
          return
        }
        let params = {}
        if (this.externalProgress) {
          params.onProgress = this.externalProgress
        } else {
          params.onProgress = ((gameInstance, progress) => {
            this.loaded = (progress === 1)
            this.progress = progress
          })
        }
        if (this.module) {
          params.Module = this.module
        }
        this.gameInstance = UnityLoader.instantiate(this.containerId, this.src, params)
      }

      if (this.eventBus.ready) {
        instantiate()
      } else {
        this.eventBus.$on('onload', () => {
          instantiate()
        })
      }
    }
  }

</script>
