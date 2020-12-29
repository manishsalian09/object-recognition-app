<template>
  <div id="camera">
    <video ref="videoRef" width="640" height="480"></video>
    <canvas ref="output"></canvas>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted, toRefs } from "vue";
import * as cocoSsd from '@tensorflow-models/coco-ssd';
import '@tensorflow/tfjs-backend-webgl';
import '@tensorflow/tfjs-backend-cpu';

export default defineComponent({
  name: "VideoFeed",
  data() {
    return {
      videoRef: Object,
      canvasRef: Object
    }
  },
  mounted() {
    this.videoRef = this.$refs.videoRef;
    this.canvasRef = this.$refs.output;
    this.startVideoFeed();
  },
  methods: {
    startVideoFeed() {
      navigator.mediaDevices.getUserMedia({ video: true, audio: false })
      .then(stream => {
        this.videoRef.srcObject = stream;
        this.videoRef.play();
        this.predictObjects();
      })
      .catch(function(err) {
          console.log("An error occurred: " + err);
      });
    },
    async predictObjects() {
      const model = await cocoSsd.load();
      setInterval(async () => {
        const predictions = await model.detect(this.videoRef);
        console.log(predictions);
        this.draw(predictions);
      }, 1000);
    },
    draw(predictions) {
      const ctx = this.canvasRef.getContext("2d");
      ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height);
      this.canvasRef.height = this.videoRef.height;
      this.canvasRef.width = this.videoRef.width;
      predictions.forEach(prediction => {
        const [x, y, width, height] = prediction['bbox']; 
        const text = prediction['class']; 
        ctx.strokeStyle = 'green'
        ctx.font = '18px Arial';
        ctx.fillStyle = 'green';
        ctx.beginPath();   
        ctx.fillText(text, x, y);
        ctx.rect(x, y, width, height); 
        ctx.stroke();
      });
    }
  }
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
video {
  left: 0;
  right: 0;
  position: absolute;
}

canvas {
  left: 0;
  right: 0;
  position: absolute;
}
</style>
