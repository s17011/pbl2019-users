<template>
  <div id="app">
    <div id="form-page">
      <div class="form-container">
        <!-- <h1>投稿</h1> -->
        <div class="input">
          <div class="input-container">
            <div class="input-boxes">
              <div class="input-box">
                <div class="title">
                  <p>画像</p>
                  <p class="require">*必須</p>
                </div>

                <div class="input-area">
                  <div class="input-button">
                    <label
                      @click="pickFile"
                      v-model="imageName"
                      style="display:inline;"
                    >
                      画像を選択
                    </label>
                    <input
                      type="file"
                      ref="image"
                      accept="image/*"
                      @change="onFilePicked"
                      required
                    />
                  </div>
                </div>

                <div class="preview">
                  <figure>
                    <img :src="imageURL" />
                  </figure>
                </div>
              </div>

              <div class="input-box">
                <div class="title">
                  <p>問題の詳細</p>
                  <p class="require">*必須</p>
                </div>

                <div class="input-area">
                  <textarea
                    v-model="incidentDetail"
                    placeholder="問題の詳細を記入してください"
                  ></textarea>
                </div>
              </div>

              <div class="input-box">
                <div class="title">
                  <p>詳しい場所</p>
                  <p class="require">*必須</p>
                </div>

                <div class="input-area">
                  <textarea
                    v-model="locationDetail"
                    placeholder="場所の詳細を記入してください"
                  ></textarea>
                </div>
              </div>

              <div class="input-box">
                <div class="title">
                  <p>位置情報</p>
                  <p class="require">*必須</p>
                </div>

                <div class="input-area">
                  <div class="input-button">
                    <label @click="getPosition">
                      端末から取得
                    </label>

                    <label>
                      画像から取得
                    </label>
                  </div>
                </div>

                <div id="map" class="preview">
                  <iframe
                    style="height:100%; width:100%;"
                    :src="iframeURL"
                  ></iframe>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="confirm">
          <label class="confirm-button" @click="upload">
            送信する
          </label>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { db, cs } from "./main";

export default {
  name: "App",
  data() {
    return {
      incidentDetail: "",
      locationDetail: "",
      imageName: "",
      imageURL: "",
      imageFile: "",
      latitude: 0,
      longitude: 0,
      iframeURL: null
    };
  },
  created() {
    //do something after creating vue instance
    this.flattenData();
  },
  methods: {
    // 画像取得処理
    pickFile() {
      this.$refs.image.click();
      // this.getPosition();
    },
    onFilePicked(e) {
      let files = e.target.files;
      if (files[0] !== undefined) {
        this.imageName = files[0].name;
        if (this.imageName.lastIndexOf(".") <= 0) {
          return;
        }
        const fr = new FileReader();
        fr.readAsDataURL(files[0]);
        fr.addEventListener("load", () => {
          this.imageURL = fr.result;
          this.imageFile = files[0];
        });
      } else {
        this.imageName = this.imageFile = this.imageURL = null;
      }
    },
    flattenData() {
      this.incidentDetail = this.locationDetail = this.imageName = this.imageURL = this.imageFile = this.latitude = this.longitude = this.iframeURL = null;
    },
    isIncludeNull() {
      return [
        this.incidentDetail,
        this.locationDetail,
        this.latitude,
        this.longitude,
        this.imageFile
      ].includes(null);
    },
    // 画像アップロード処理
    upload() {
      if (this.isIncludeNull()) {
        alert("必須項目が入力されていません");
        return;
      }
      // Storageオブジェクト作成
      let storageRef = cs.ref();
      // ファイルパス設定
      let incidentsRef = storageRef.child("images/" + this.imageName);
      // timestamp用のDate
      let date = new Date();
      let now =
        date.toLocaleString().replace(/\//g, "") + ":" + date.getMilliseconds();
      console.log(now);
      // ファイルを適用してアップロード
      incidentsRef.put(this.imageFile).then(snapshot => {
        snapshot.ref.getDownloadURL().then(downloadURL => {
          // ↓Debug↓
          let stat = {
            timestamp: now,
            incidentDetail: this.incidentDetail,
            locationDetail: this.locationDetail,
            latitude: this.latitude,
            longitude: this.longitude,
            downloadURL
          };
          console.log(stat);
          // ↑Debug↑
          db.collection("images")
            .doc(`${now}`)
            .set(stat);
        }).then(()=>{
          alert("ご報告ありがとうございました。")
          this.flattenData()
        });
      });
    },
    getPosition() {
      navigator.geolocation.getCurrentPosition(pos => {
        this.latitude = pos.coords.latitude;
        this.longitude = pos.coords.longitude;

        console.log(this.latitude, this.longitude);
        this.setIframeURL();
      });
    },
    // setMap() {
    //   let map = L.map('map');
    //   let position = [this.latitude, this.longitude];
    //   map.setView(position, 15);
    //   L.tileLayer( 'http://{s}.tile.osm.org/{z}/{x}/{y}.png', {
    // 		maxZoom: 18,
    // 		attribution: 'Map data &copy; <a href="http://openstreetmap.org">OpenStreetMap</a> contributors, '
    // 	}).addTo(map);
    //   let mark = L.marker(position);
    //   mark.addTo(map);
    // },
    setIframeURL() {
      this.iframeURL = `https://maps.google.co.jp/maps?output=embed&q=${this.latitude},${this.longitude}`;
    }
  },
  components: {}
};
</script>

<style>
/* #app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
} */

#form-page p {
  margin: 0;
}

#form-page .form-container {
  max-width: 800px;
  margin: 0 auto;
}

#form-page .input-boxes {
  display: flex;
  flex-flow: column;
}

#form-page .input-box {
  display: flex;
  flex-flow: row;
  margin-bottom: 3px;
}

/* input-box内の要素 */
#form-page .title,
#form-page .input-area,
#form-page .preview {
  padding: 10px;
  border: 1px solid #ccc;
}

#form-page .title {
  display: flex;
  flex: 0 0 150px;
  justify-content: center;
  align-items: center;
  background-color: #eee;
  border-right: none;
}

#form-page .input-area {
  display: flex;
  flex: 1 1 auto;
  justify-content: center;
  align-items: center;
}

#form-page .preview {
  flex: 0 0 400px;
  height: 200px;
  border-left: none;
}

#form-page .input-area input,
#form-page .input-area textarea {
  width: 100%;
  height: 100%;
  box-sizing: border-box;
}

#form-page .input-area textarea {
  resize: none;
  height: 80px;
  padding: 5px;
}

/* titleの要素 */
#form-page .title .require {
  font-size: 8px;
  color: red;
}

/* 選択ボタン */
#form-page .input-button {
  display: flex;
  flex-flow: column;
}

#form-page .input-button label {
  display: inline-block;
  background-color: #666ab8;
  text-align: center;
  color: #fff;
  margin: 5px;
  padding: 10px;
  border-radius: 3px;
}

#form-page .input-button label:hover {
  background-color: #888fff;
}

#form-page .input-button input,
#form-page .input-button button {
  display: none;
}

/* 画像のプレビュー */
#form-page .preview figure {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  margin: 0;
}

#form-page .input-container img {
  max-width: 100%;
  max-height: 100%;
}

/* 確認画面ボタン */
#form-page .confirm {
  margin-top: 10px;
  padding: 0 30px;
  text-align: center;
}

#form-page .confirm-button {
  display: inline-block;
  background-color: #666ab8;
  width: 100%;
  max-width: 200px;
  padding: 10px;
  border-radius: 3px;
  color: #fff;
  text-align: center;
  box-sizing: border-box;
}

#form-page .confirm-button:hover {
  background-color: #888fff;
}

#form-page .confirm button {
  display: none;
}

/* 768px以下 */
@media only screen and (max-width: 768px) {
  #form-page .input-container {
    width: 100%;
  }

  #form-page .title {
    flex: 0 0 30px;
    border-right: 1px solid #ccc;
    border-bottom: none;
  }

  #form-page .input-box {
    flex-flow: column;
  }

  #form-page .preview {
    flex: 0 0 auto;
    border-left: 1px solid #ccc;
    border-top: none;
  }
}
</style>
