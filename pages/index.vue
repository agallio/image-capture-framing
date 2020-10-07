<template>
  <v-row justify="center" align="center">
    <v-col cols="12">
      <v-dialog
        v-model="isCameraOpen"
        fullscreen
        hide-overlay
        transition="dialog-bottom-transition"
      >
        <div class="dialog-content">
          <div id="container" ref="container">
            <div id="vid_container" ref="vidContainer" class="">
              <video ref="video" id="video" autoplay playsinline></video>
              <div id="video_overlay">
                <div class="base"></div>
                <v-btn icon dark x-large @click="toggleCamera">
                  <v-icon>mdi-close</v-icon>
                </v-btn>
              </div>
            </div>
            <div id="gui_controls" ref="guiControls" class="">
              <button
                class="cam-btn"
                ref="switchCameraButton"
                id="switchCameraButton"
                name="switch Camera"
                type="button"
                aria-pressed="false"
                @click="toggleSwitchCamera"
              ></button>
              <button
                class="cam-btn"
                ref="takePhotoButton"
                id="takePhotoButton"
                name="take Photo"
                type="button"
                @click="takePhotoButton"
              ></button>
              <button
                class="cam-btn"
                ref="toggleFullScreenButton"
                id="toggleFullScreenButton"
                name="toggle FullScreen"
                type="button"
                aria-pressed="false"
                @click="toggleFullScreen"
              ></button>
            </div>
          </div>
        </div>
      </v-dialog>

      <v-dialog
        v-model="isImageCaptured"
        fullscreen
        hide-overlay
        transition="dialog-bottom-transition"
      >
        <v-card>
          <v-toolbar dark color="#000">
            <v-btn icon dark @click="toggleCamera">
              <v-icon>mdi-close</v-icon>
            </v-btn>
            <v-toolbar-title>Image Captured!</v-toolbar-title>
            <v-spacer></v-spacer>
            <v-toolbar-items>
              <v-btn dark text @click="isImageCaptured = false">Save</v-btn>
            </v-toolbar-items>
          </v-toolbar>

          <div class="d-flex align-center justify-center preview">
            <img :src="capturedImage" alt="image" class="capturedImage" />
          </div>
        </v-card>
      </v-dialog>

      <h2 class="h2 mb-4">Image Capture &amp; Compressor</h2>

      <img
        v-if="capturedImage"
        :src="capturedImage"
        alt="image"
        class="capturedImage"
      />

      <v-btn depressed block @click="toggleCamera">Buka Kamera</v-btn>
      <v-btn
        depressed
        block
        color="warning"
        class="mt-2"
        :disabled="!capturedImageData"
        @click="checkImageSize"
        >Send</v-btn
      >

      <div class="image-detail">
        <template v-if="imageDetail.width > 0">
          <p class="my-0">Size : {{ imageDetail.size }}</p>
          <p class="my-0">Width : {{ imageDetail.width }}</p>
          <p class="my-0">Height: {{ imageDetail.height }}</p>
        </template>
        <template v-else>Upload image first!</template>
      </div>

      <v-btn
        depressed
        block
        class="mt-2"
        color="primary"
        :disabled="imageDetail.width === 0"
        @click="downloadImage"
        >Download</v-btn
      >
    </v-col>
  </v-row>
</template>

<script>
import screenfull from "screenfull";

export default {
  data() {
    return {
      isCameraOpen: false,
      isImageCaptured: false,
      imageReady: false,
      imageDetail: {
        width: 0,
        height: 0,
        size: "",
      },

      // Camera
      video: null,
      capturedImage: null,
      capturedImageData: null,
      amountOfCameras: 0,
      currentFacingMode: "environment",
    };
  },
  methods: {
    detectIfiOS() {
      return (
        [
          "iPad Simulator",
          "iPhone Simulator",
          "iPod Simulator",
          "iPad",
          "iPhone",
          "iPod",
        ].includes(navigator.platform) ||
        // iPad on iOS 13 detection
        (navigator.userAgent.includes("Mac") && "ontouchend" in document)
      );
    },
    toggleCamera() {
      this.resetImage();

      if (this.isCameraOpen) {
        this.capturedImage = null;
        this.isCameraOpen = false;
        this.isImageCaptured = false;
        if (!this.detectIfiOS()) {
          this.toggleFullScreen();
        }
        this.stopCameraStream();
      } else {
        this.capturedImage = null;
        this.isCameraOpen = true;
        this.isImageCaptured = false;
        if (!this.detectIfiOS()) {
          this.toggleFullScreen();
        }
        this.createCameraElement();
      }
    },
    createCameraElement() {
      if (
        navigator.mediaDevices &&
        navigator.mediaDevices.getUserMedia &&
        navigator.mediaDevices.enumerateDevices
      ) {
        navigator.mediaDevices
          .getUserMedia({ audio: false, video: true })
          .then((stream) => {
            stream.getTracks().forEach((track) => track.stop());

            this.deviceCount().then((deviceCount) => {
              this.amountOfCameras = deviceCount;

              // Init the UI and the camera stream
              this.initCameraUI();
              this.initCameraStream();
            });
          })
          .catch((err) => {
            if (err === "PermissionDeniedError") {
              alert("Permission denied. Please refresh and give permission.");
            }

            console.error(`getUserMedia() error: ${err}`);
          });
      } else {
        alert(
          "Mobile camera is not supported by browser, or there is no camera detected/connected"
        );
        this.isCameraOpen = false;
      }
    },
    stopCameraStream() {
      let tracks = this.$refs.video.srcObject.getTracks();

      tracks.forEach((track) => {
        track.stop();
      });
    },

    deviceCount() {
      return new Promise((resolve) => {
        let videoInCount = 0;

        navigator.mediaDevices
          .enumerateDevices()
          .then((devices) => {
            devices.forEach((device) => {
              if (device.kind === "video") {
                device.kind = "videoinput";
              }

              if (device.kind === "videoinput") {
                videoInCount++;
                console.log(`Videocam : ${device.label}`);
              }
            });

            resolve(videoInCount);
          })
          .catch((err) => {
            console.log(`${err.name} : ${err.message}`);
            resolve(0);
          });
      });
    },
    initCameraUI() {
      const takePhotoButton = this.$refs.takePhotoButton;
      const toggleFullScreenButton = this.$refs.toggleFullScreenButton;
      const switchCameraButton = this.$refs.switchCameraButton;

      // const fullScreenChange = () => {
      //   if (screenfull.isFullscreen) {
      //     toggleFullScreenButton.setAttribute("aria-pressed", true);
      //   } else {
      //     toggleFullScreenButton.setAttribute("aria-pressed", false);
      //   }
      // };

      // if (screenfull.isEnabled) {
      //   screenfull.on("change", fullScreenChange);

      //   toggleFullScreenButton.style.display = "block";

      //   fullScreenChange();
      // } else {
      //   console.log("iOS doesn't support fullscreen (yet)");
      // }

      if (this.amountOfCameras > 1) {
        switchCameraButton.style.display = "block";
      }

      let angle;
      window.addEventListener(
        "orientationchange",
        () => {
          if (screen.orientation) {
            angle = screen.orientation.angle;
          } else {
            angle = window.orientation;
          }

          const guiControls = this.$refs.guiControls.classList;
          const vidContainer = this.$refs.vidContainer.classList;

          if (angle === 270 || angle === -90) {
            guiControls.add("left");
            vidContainer.add("left");
          } else {
            if (guiControls.contains("left")) guiControls.remove("left");
            if (vidContainer.contains("left")) vidContainer.remove("left");
          }
        },
        false
      );
    },
    initCameraStream() {
      let video = document.getElementById("video");
      const switchCameraButton = this.$refs.switchCameraButton;

      if (window.stream) {
        window.stream.getTracks().forEach((track) => {
          console.log(track);
          track.stop();
        });
      }

      const size = 1280;
      const constraints = {
        audio: false,
        video: {
          width: { ideal: size },
          height: { ideal: size },
          facingMode: this.currentFacingMode,
        },
      };

      navigator.mediaDevices
        .getUserMedia(constraints)
        .then((stream) => {
          window.stream = stream;
          video.srcObject = stream;

          if (constraints.video.facingMode) {
            if (constraints.video.facingMode === "environment") {
              switchCameraButton.setAttribute("aria-pressed", true);
            } else {
              switchCameraButton.setAttribute("aria-pressed", false);
            }
          }

          const track = window.stream.getVideoTracks()[0];
          const settings = track.getSettings();
          console.log(`Settings : ${JSON.stringify(settings, null, 4)}`);
        })
        .catch((err) => console.error(`getUserMedia() error : ${err}`));
    },
    takePhotoButton() {
      this.takeSnapshot();
    },
    toggleFullScreen() {
      screenfull.toggle(this.$refs.container).then(() => {
        console.log(
          `Fullscreen mode : ${
            screenfull.isFullscreen ? "enabled" : "disabled"
          }`
        );
      });
    },
    toggleSwitchCamera() {
      if (currentFacingMode === "environment") {
        this.currentFacingMode = "user";
      } else {
        this.currentFacingMode = "environment";
      }

      this.initCameraStream();
    },
    takeSnapshot() {
      let canvas = document.createElement("canvas");
      let video = document.getElementById("video");

      let width = video.videoWidth;
      let height = video.videoHeight;

      canvas.width = width;
      canvas.height = height;

      context = canvas.getContext("2d");
      context.drawImage(video, 0, 0, width, height);

      const getCanvasBlob = (canvas) => {
        return new Promise((resolve) =>
          canvas.toBlob((blob) => resolve(blob), "image/jpeg")
        );
      };

      let imageFile;
      getCanvasBlob(canvas).then((blob) => {
        imageFile = new File([blob], "image.jpg", { type: "image/jpeg" });
        this.toggleCamera();
        this.capturedImage =
          imageFile && imageFile instanceof File
            ? URL.createObjectURL(imageFile)
            : null;
        this.capturedImageData = imageFile;
        this.isImageCaptured = true;
      });
    },

    resetImage() {
      this.imageDetail = {
        width: 0,
        height: 0,
        size: "",
      };
      this.imageReady = false;
    },
    checkImageSize() {
      let it = this;
      var fr = new FileReader();
      fr.onload = function () {
        var img = new Image();

        img.onload = function () {
          it.imageDetail = {
            width: img.width,
            height: img.height,
            size: Math.floor(it.capturedImageData.size / 1000) + "KB",
          };
          it.imageReady = true;
        };

        img.src = fr.result;
      };
      fr.readAsDataURL(this.capturedImageData);
    },
    downloadImage() {
      let a = document.createElement("a");
      document.body.appendChild(a);
      a.style = "display: none";
      let blob = new Blob([this.capturedImageData], { type: "octet/stream" }),
        url = window.URL.createObjectURL(blob);
      a.href = url;
      a.download = "image.jpg";
      a.click();
      window.URL.revokeObjectURL(url);
    },
  },
};
</script>

<style lang="scss">
.frame-overlay {
  position: absolute;
  top: 0;
  left: 0;
}

.dialog-content {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  background: #000;
}

#vid_container {
  position: fixed;
  top: 0;
  left: 0;
}

#video {
  width: 100%;
  height: 100%;
  object-fit: cover;
  z-index: 0;
}

#gui_controls {
  position: fixed;
  background-color: #111; /*rgba(255, 0, 0, 0.5);*/
  z-index: 2;
  bottom: 0;
}

#video_overlay {
  position: fixed;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;

  z-index: 10;
  background-color: transparent;
}

.base {
  position: absolute;
  width: 204px;
  height: 323px;
  box-shadow: 0px 0px 20px 200px rgba(0, 0, 0, 0.6);
  border: 1px solid #ffffff;
  border-radius: 10px;
  transform: translate(-50%, -70%);
  top: 50%;
  left: 50%;
}

.preview {
  background: #000;
  height: calc(100vh - 56px);
}

.cam-btn {
  outline: none;
  position: absolute;
  color: white;
  display: block;
  opacity: 1;
  background: transparent;
  border: solid 2px #fff;
  padding: 0;
  text-shadow: 0px 0px 4px black;
  background-position: center center;
  background-repeat: no-repeat;
  pointer-events: auto;
  z-index: 2;
}

#takePhotoButton {
  left: calc(50% - 40px);
  top: calc(50% - 40px);
  width: 80px;
  height: 80px;
  background-image: url("/ic_photo_camera_white_48px.svg");
  border-radius: 50%;
  background-color: rgba(0, 0, 0, 0.5);
}

#takePhotoButton:active {
  background-color: #fff;
}

#toggleFullScreenButton {
  display: none;
  width: 64px;
  height: 64px;
  background-image: url("/ic_fullscreen_white_48px.svg");
  border-radius: 50%;
  background-color: rgba(0, 0, 0, 0.5);
}

#toggleFullScreenButton[aria-pressed="true"] {
  background-image: url("/ic_fullscreen_exit_white_48px.svg");
}

#switchCameraButton {
  display: none;
  width: 64px;
  height: 64px;
  background-image: url("/ic_camera_rear_white_36px.svg");
  border-radius: 50%;
  background-color: rgba(0, 0, 0, 0.5);
}

#switchCameraButton[aria-pressed="true"] {
  background-image: url("/ic_camera_front_white_36px.svg");
}

.capturedImage {
  width: 100%;
}

.image-detail {
  margin-top: 10px;
  padding: 10px 20px;
  text-align: left;
  background: #f0f0f0;
}

@media screen and (orientation: portrait) {
  /* portrait-specific styles */

  /* video_container (video) doesn't respect height... 
       so we will fill it in completely in portrait mode
    */
  #vid_container {
    width: 100%;
    height: 80%;
  }

  #gui_controls {
    width: 100%;
    height: 20%;
    left: 0;
  }

  #switchCameraButton {
    left: calc(20% - 32px);
    top: calc(50% - 32px);
  }

  #toggleFullScreenButton {
    left: calc(80% - 32px);
    top: calc(50% - 32px);
  }
}

@media screen and (orientation: landscape) {
  .base {
    height: 204px;
    width: 323px;
    transform: translate(-80%, -50%);
    top: 50%;
    left: 50%;
  }

  #vid_container {
    width: 80%;
    height: 100%;
  }

  #vid_container.left {
    left: 20%;
  }

  /* we default to right */
  #gui_controls {
    width: 20%;
    height: 100%;
    right: 0;
  }

  /* for the lefties */
  #gui_controls.left {
    left: 0;
  }

  #switchCameraButton {
    left: calc(50% - 32px);
    top: calc(18% - 32px);
  }

  #toggleFullScreenButton {
    left: calc(50% - 32px);
    top: calc(82% - 32px);
  }
}
</style>
