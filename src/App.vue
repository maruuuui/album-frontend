<template>
  <v-app>
    <v-app-bar app color="blue lighten-4" short>
      <div class="d-flex align-center">
        <v-btn text>
          <span>album</span>
        </v-btn>
      </div>
      <v-spacer></v-spacer>
      <div class="d-flex align-center">
        <v-btn text>
          <span class="mr-2">upload Image</span>
        </v-btn>
      </div>
    </v-app-bar>

    <v-content>
      <template>
        <v-container>
          <form
            action="/api/v1/image/"
            method="POST"
            enctype="multipart/form-data"
            class="form-horizontal"
            novalidate
          >
            <v-file-input label="アップロード画像" filled name="image" id="post_image_id"></v-file-input>
            <v-text-field
              v-model="post_memo"
              prepend-icon="mdi-pencil"
              label="メモ"
              filled
              name="memo"
            ></v-text-field>
            <v-btn @click="create(api_url)">登録</v-btn>
          </form>
        </v-container>
      </template>
      <v-container fluid grid-list-md>
        <v-row align-content="start">
          <v-col sm="4" md="3" lg="2" v-for="image_item in image_items" v-bind:key="image_item.id">
            <!-- 各画像のサムネイル -->
            <v-card class="flexcard" height="100%" @click="showOverlay(image_item)">
              <v-img class="align-end" :src="image_item.image" height="200">
                <div class="blue lighten-4">
                  <span class="white--text">{{ image_item.image_name }}</span>
                </div>
              </v-img>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-content>
    <!-- カードクリック時に開くオーバーレイのコンテンツ -->
    <v-dialog :value="overlay" :dark="false">
      <v-card>
        <v-toolbar color="gray" dark>
          <v-toolbar-title>画像詳細</v-toolbar-title>
          <v-spacer></v-spacer>

          <v-btn color="primary" fab @click="overlay = false">&times;</v-btn>
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
          <v-btn color="red" @click="deleteImage(overlay_item_url)">削除</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-app>
</template>
<script>
import axios from "axios";
import Cookies from "js-cookie";

export default {
  name: "App",

  data: () => ({
    image_items: [], //APIで取得してくる画像の入れ物
    uploadWindowIsShown: false,
    overlay: false, //オーバーレイの表示状態
    overlay_item: {}, //オーバーレイに表示する対象
    overlay_item_url: "", //オーバーレイに表示する対象の編集に使うURL
    api_url: "http://127.0.0.1:8000/api/v1/image/",
    post_file: null, //アップロードする画像
    post_memo: "" //新規アップロード画像に対するメモ
  }),
  created: function() {
    //RestAPIからDB上の画像を持ってくる
    this.getImage(this.api_url);

    //DjangoにPOSTなどするときに必要なトークンを取得
    axios.get(this.api_url).then(function(response) {
      Cookies.set("csrftoken", response.data.token);
    });
  },
  methods: {
    create(overlay_item_url) {
      let obj1 = document.getElementById("post_image_id");
      this.post_file = obj1.files[0];

      //送信データ
      const params = new FormData();
      params.append("memo", this.post_memo);
      params.append("image", this.post_file);

      this.postImage(overlay_item_url, params);
    },
    //オーバーレイに表示する対象(レコード)の情報を入れ込んで表示
    showOverlay(image_item) {
      this.overlay = !this.overlay;
      this.overlay_item = image_item;
      this.overlay_item_url = this.api_url + image_item.id + "/";
    },
    getImage(url) {
      this.image_items = [];
      var self = this; //スコープ的に必要っぽい
      axios.get(url).then(function(response) {
        for (var i = 0; i < response.data.results.length; i++) {
          self.image_items.push(response.data.results[i]);
        }
      });
    },
    postImage(url, request) {
      var self = this; //スコープ的に必要っぽい
      let requestHeader = {
        headers: {
          "X-CSRFToken": Cookies.get("csrftoken"),
          "content-type": "multipart/form-data"
        }
      };
      axios
        .post(url, request, requestHeader)
        .then(function(response) {
          alert(response);
          self.getImage(url);
        })
        .catch(function(error) {
          alert(error);
          self.getImage(url);
        });
    },
    patchImage(url, request) {
      let requestHeader = {
        headers: { "X-CSRFToken": Cookies.get("csrftoken") }
      };
      axios
        .patch(url, request, requestHeader)
        .then(function(response) {
          alert(response);
        })
        .catch(function(error) {
          alert(error);
        });
    },
    deleteImage(url) {
      var self = this; //スコープ的に必要っぽい
      let requestHeaderWithNullData = {
        headers: { "X-CSRFToken": Cookies.get("csrftoken") },
        data: {}
      };
      axios
        .delete(url, requestHeaderWithNullData)
        .then(function() {
          alert("削除しました");
          self.overlay = false;
          self.getImage();
        })
        .catch(function(error) {
          alert(error);
        });
    }
  },
  computed: {
    window_width: function() {
      return window.innerWidth; //画面幅
    },
    window_height: function() {
      return window.innerHeight; //画面幅
    }
  }
};
</script>

<style>
.flexcard {
  display: flex;
  flex-direction: column;
}
</style>