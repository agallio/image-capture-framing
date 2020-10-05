<template>
  <v-row justify="center" align="center">
    <v-col cols="12">
      <v-dialog
        v-model="isCameraOpen"
        fullscreen
        hide-overlay
        transition="dialog-bottom-transition"
      >
        <v-toolbar dark color="primary">
          <v-btn icon dark @click="isCameraOpen = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
          <v-toolbar-title>Settings</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-toolbar-items>
            <v-btn dark text @click="isCameraOpen = false"> Save </v-btn>
          </v-toolbar-items>
        </v-toolbar>

        <div class="camera-box">
          <video id="vid" ref="camera" width="100%" autoplay></video>
          <div class="frame-box">
            <div class="frame-radius"></div>
          </div>
        </div>
      </v-dialog>

      <v-btn depressed @click="toggleCamera">Buka Kamera</v-btn>
    </v-col>
  </v-row>
</template>

<script>
export default {
  data() {
    return {
      isCameraOpen: false
    };
  },
  methods: {
    toggleCamera() {
      if (this.isCameraOpen) {
        this.isCameraOpen = false;
        this.stopCameraStream();
      } else {
        this.isCameraOpen = true;
        this.createCameraElement();
      }
    },
    createCameraElement() {
      const constraints = (window.constraints = {
        audio: false,
        video: {
          width: 768,
          height: 1024
        }
      });

      navigator.mediaDevices
        .getUserMedia(constraints)
        .then(stream => {
          this.$refs.camera.srcObject = stream;
        })
        .catch(error => {
          console.log(error);
          alert("May the browser didn't support or there is some errors.");
        });
    },
    stopCameraStream() {
      let tracks = this.$refs.camera.srcObject.getTracks();

      tracks.forEach(track => {
        track.stop();
      });
    }
  }
};
</script>

<style lang="scss">
.camera-box {
  position: absolute;
  height: calc(100% - 56px);
  background: black;
}
#vid {
  height: 100%;
}
.frame {
  &-box {
    width: 100%;
    height: 100%;
    position: absolute;
    top: 0px;
    border-right: 50px solid black;
    border-left: 50px solid black;
    border-top: 100px solid black;
    border-bottom: 100px solid black;
    opacity: 0.5;
  }

  &-radius {
    width: 107%;
    height: 106%;
    background: transparent;
    border-radius: 23px;
    border: 13px solid black;
    position: absolute;
    top: -7px;
    left: -7px;
  }
}
</style>
