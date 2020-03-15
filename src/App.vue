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
        <v-btn text @click="showUploadForm = !showUploadForm">
          <span class="mr-2">upload Image</span>
        </v-btn>
      </div>
    </v-app-bar>

    <v-content>
      <template v-if="showUploadForm">
        <v-container>
          <form
            action="/api/v1/image/"
            method="POST"
            enctype="multipart/form-data"
            class="form-horizontal"
            novalidate
          >
            <v-file-input
              label="アップロード画像"
              filled
              name="image"
              id="post_image_id"
            ></v-file-input>
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
          <v-col
            sm="4"
            md="3"
            lg="2"
            v-for="image_item in image_items"
            v-bind:key="image_item.id"
          >
            <image-card :image_item="image_item" @showOverlay="showOverlay" />
          </v-col>
        </v-row>
      </v-container>
    </v-content>
    <!-- カードクリック時に開くオーバーレイのコンテンツ -->
    <v-dialog :value="showImageDetailOverlay" :dark="false">
      <image-detail-overlay
        :window_width="window_width"
        :window_height="window_height"
        :overlay_item="overlay_item"
        :overlay_item_url="overlay_item_url"
        @deleteImage="deleteImage"
        @closeOverlay="closeOverlay"
      />
    </v-dialog>
  </v-app>
</template>
<script>
/* eslint-disable no-console */
import axios from "axios";
import Cookies from "js-cookie";
import imageCard from "./components/imageCard";
import imageDetailOverlay from "./components/imageDetailOverlay";

export default {
  name: "App",

  components: {
    "image-card": imageCard,
    "image-detail-overlay": imageDetailOverlay
  },

  data: () => ({
    image_items: [], //APIで取得してくる画像の入れ物
    showUploadForm: false,
    showImageDetailOverlay: false, //オーバーレイの表示状態
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
      this.showImageDetailOverlay = !this.showImageDetailOverlay;
      this.overlay_item = image_item;
      this.overlay_item_url = this.api_url + image_item.id + "/";
    },
    //オーバーレイを閉じる
    closeOverlay() {
      this.showImageDetailOverlay = false;
    },
    getImage(url) {
      console.log("getImage", url); // レスポンスデータ

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
          console.log(response.data); // レスポンスデータ
          console.log(response.status); // ステータスコード
          console.log(response.statusText); // ステータステキスト
          console.log(response.headers); // レスポンスヘッダ
          console.log(response.config); // コンフィグ
          alert("画像を追加しました。");
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

        .then(function(response) {
          console.log(response.data); // レスポンスデータ
          console.log(response.status); // ステータスコード
          console.log(response.statusText); // ステータステキスト
          console.log(response.headers); // レスポンスヘッダ
          console.log(response.config); // コンフィグ
          alert("削除しました。");
          self.showImageDetailOverlay = false;
          self.getImage(self.api_url);
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
