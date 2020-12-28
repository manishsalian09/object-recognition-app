<template>
  <div>
    <VideoFeed />
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import VideoFeed from "@/components/VideoFeed.vue"; // @ is an alias to /src

export default defineComponent({
  name: "Home",
  components: {
    VideoFeed
  }
});
</script>
