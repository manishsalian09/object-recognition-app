<template>
  <div>
    <button @click="start">Start</button>
    <div id="camera">
      <video ref="videoRef"></video>
      <canvas ref="output"></canvas>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted, toRefs } from "vue";
import * as cocoSsd from '@tensorflow-models/coco-ssd';
import '@tensorflow/tfjs-backend-webgl';
import '@tensorflow/tfjs-backend-webgl';

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
  },
  methods: {
    start() {
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
      const predictions = await model.detect(this.videoRef);
      console.log('Predictions: ');
      console.log(predictions);
      this.draw(predictions);
    },
    draw(predictions) {
      const ctx = this.canvasRef.getContext("2d");
      ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height);
      const font = "24px helvetica";
      ctx.font = font;
      ctx.textBaseline = "top";

      predictions.forEach(prediction => {
        const x = prediction.bbox[0];
        const y = prediction.bbox[1];
        const width = prediction.bbox[2];
        const height = prediction.bbox[3];
        // Draw the bounding box.
        ctx.strokeStyle = "#2fff00";
        ctx.lineWidth = 1;
        ctx.strokeRect(x, y, width, height);
        // Draw the label background.
        ctx.fillStyle = "#2fff00";
        const textWidth = ctx.measureText(prediction.class).width;
        const textHeight = parseInt(font, 10);
        // draw top left rectangle
        ctx.fillRect(x, y, textWidth + 10, textHeight + 10);
        // draw bottom left rectangle
        ctx.fillRect(x, y + height - textHeight, textWidth + 15, textHeight + 10);

        // Draw the text last to ensure it's on top.
        ctx.fillStyle = "#000000";
        ctx.fillText(prediction.class, x, y);
        ctx.fillText(prediction.score.toFixed(2), x, y + height - textHeight);
      });
    }
  }
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
</style>
