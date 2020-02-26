<template>
  <v-card class="flexcard" height="100%" @click="$emit('showOverlay', image_item)">
    <v-img class="align-end" :src="image_item.image" height="200">
      <div class="blue lighten-4">
        <span class="white--text">{{ image_item.image_name }}</span>
      </div>
    </v-img>
  </v-card>
</template>

<script>
export default {
  name: "imageCard",
  props: ["image_item"]
};
</script>