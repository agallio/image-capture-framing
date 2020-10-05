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
          <video ref="camera" width="100%" autoplay></video>
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
      isCameraOpen: false,
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
          height: 1024,
        },
      });

      navigator.mediaDevices
        .getUserMedia(constraints)
        .then((stream) => {
          this.$refs.camera.srcObject = stream;
        })
        .catch((error) => {
          console.log(error);
          alert("May the browser didn't support or there is some errors.");
        });
    },
    stopCameraStream() {
      let tracks = this.$refs.camera.srcObject.getTracks();

      tracks.forEach((track) => {
        track.stop();
      });
    },
  },
};
</script>

<style>
.camera-box {
  position: absolute;
  /* top: 20px; */
  /* right: 20px; */
  /* bottom: 20px; */
  /* left: 20px; */
  /* box-shadow: 0px 0px 20px 56px rgba(0, 0, 0, 0.6); */
  border: 1px solid #ffffff;
  border-radius: 10px;
}
</style>
