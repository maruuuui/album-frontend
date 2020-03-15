<template>
  <v-card>
    <v-toolbar color="gray" dark>
      <v-toolbar-title>画像詳細</v-toolbar-title>
      <v-spacer></v-spacer>

      <v-btn color="primary" fab @click="$emit('closeOverlay')">&times;</v-btn>
    </v-toolbar>

    <v-row class="d-flex justify-center">
      <v-img
        :src="overlay_item.image"
        contain
        :max-width="0.8*window_width"
        :max-height="0.8*window_height"
      ></v-img>
    </v-row>

    <!-- <v-card-text class="text--primary"> -->
    <v-list class="transparent">
      <v-list-item>
        <v-list-item-title>画像ファイル名</v-list-item-title>
        <v-list-item-subtitle>{{ overlay_item.image_name }}</v-list-item-subtitle>
      </v-list-item>
    </v-list>

    <v-list class="transparent">
      <v-list-item>
        <v-list-item-title>備考</v-list-item-title>
        <v-list-item-subtitle>{{ overlay_item.memo }}</v-list-item-subtitle>
      </v-list-item>
    </v-list>
    <v-list class="transparent">
      <v-list-item>
        <v-list-item-title>更新日時</v-list-item-title>
        <v-list-item-subtitle>{{ overlay_item.updated_at }}</v-list-item-subtitle>
      </v-list-item>
    </v-list>
    <v-list class="transparent">
      <v-list-item>
        <v-list-item-title>登録日時</v-list-item-title>
        <v-list-item-subtitle>{{ overlay_item.created_at }}</v-list-item-subtitle>
      </v-list-item>
    </v-list>
    <v-card-actions>
      <v-btn color="red" @click="$emit('deleteImage',overlay_item_url)">削除</v-btn>
    </v-card-actions>
  </v-card>
</template>

<script>
export default {
  name: "imageDetailOverlay",
  props: ["overlay_item", "overlay_item_url", "window_width", "window_height"]
};
</script>