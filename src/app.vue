<template>
  <div>
    <div id="player" class="player">
      <audio
        ref="audio"
        @pause="onPause"
        @play="onPlay"
        @timeupdate="onTimeUpdate"
        @loadedmetadata="onLoadedMetaData"
        preload="auto"
        :src="current.source"
        controls="controls"
        style="display: none"
        id="audio"
      ></audio>
      <div class="btn max2cd iconfont icon-suoxiao" @click="playerMax"></div>
      <div class="player-controls">
        <div class="music-imgs">
          <img
            class="img"
            v-on:click="playerMax"
            v-if="current.cover"
            :src="current.cover"
          />
          <img
            class="img"
            v-on:click="playerMax"
            v-if="!current.cover"
            :src="defaultCover"
          />
        </div>
        <div id="player-content3">
          <div
            class="btn prev iconfont icon-diyigeshipin"
            @click="prevAudio"
          ></div>
          <div
            class="btn play-pause iconfont icon-kaishi"
            @click="startPlayOrPause"
          ></div>
          <div
            class="btn next iconfont icon-zuihouyigeshipin"
            @click="nextAudio"
          ></div>
        </div>
        <div id="player-content2">
          <span @click="playerNarrow"
            ><a class="music-name" @click="goMusic">{{ current.name }}</a></span
          >
          <span class="max2d">-</span>
          <span @click="playerNarrow"
            ><a class="artist-name" @click="goArtist">{{
              current.artist
            }}</a></span
          >
          <div class="time">
            <div class="current-time">
              {{ audio.currentTime | formatSecond }}
            </div>
            <div class="total-time">{{ audio.maxTime | formatSecond }}</div>
          </div>
          <div
            id="s-area"
            @mousedown="handleTouchStart"
            @mousemove="sethover"
            @mouseout="hideHover"
          >
            <div class="slider-track"></div>
            <div class="silderNone" :style="'width:' + 0 + '%'"></div>
            <!-- <div id="s-hover" :style="'width:' + hoverTime + '%'"></div> -->
            <div class="slider-fill" :style="'width:' + sliderTime + '%'"></div>
            <div class="slider-thumb" :style="'left:' + thumbTime + '%'"></div>
          </div>
        </div>
        <div class="player-controls3">
          <div
            class="btn prev iconfont icon-diyigeshipin"
            @click="prevAudio"
          ></div>
          <div
            class="btn play-pause iconfont icon-kaishi"
            @click="startPlayOrPause"
          ></div>
          <div
            class="btn next iconfont icon-zuihouyigeshipin"
            @click="nextAudio"
          ></div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import $ from "jquery";
function realFormatSecond(second) {
  var secondType = typeof second;
  if (secondType === "number" || secondType === "string") {
    second = parseInt(second);
    var hours = Math.floor(second / 3600);
    second = second - hours * 3600;
    var mimute = Math.floor(second / 60);
    second = second - mimute * 60;
    return ("0" + mimute).slice(-2) + ":" + ("0" + second).slice(-2);
  } else {
    return "0:00:00";
  }
}
var times = null;
export default {
  name: "MamicPlayer",
  props: {
    defaultCover: {
      type: String,
    },
    playlist: {
      type: Array,
    },
    localplw: {
      type: Boolean,
      default: true,
    },
  },
  data: function () {
    return {
      plw: 0,

      sliderTime: 0,
      silderNone: 0,
      hoverTime: 0,
      thumbTime: 0,

      audio: {
        playing: false,
        currentTime: 0,
        maxTime: 0,
        minTime: 0,
        step: 0.1,
      },
      lefticon: "",
      play: false,
      current: {
        startTime: "00:00",
        endtime: "00:00",
        cover: "",
        name: "",
        artist: "",
        source: "",
      },
    };
  },
  beforeMount() {
    if (localStorage.getItem("Player_plw")) {
    } else {
      localStorage.setItem("Player_plw", 0);
    }
  },
  mounted() {
    if (this.localplw) {
      this.plw = localStorage.getItem("Player_plw");
    }
    setInterval(() => {
      this.$emit("updateData", this.current);
    }, 1000);
  },
  watch: {
    playlist(list) {
      this.current = list[this.plw];
    },
    plw(plw) {
      if (this.localplw) {
        localStorage.setItem("Player_plw", plw);
      }
    },
  },
  methods: {
    goMusic: function () {
      this.$emit("pressMusicName", this.current);
    },
    goArtist: function () {
      this.$emit("pressArtistName", this.current);
    },
    updateData: function () {
      this.$emit("updateData", this.current);
    },
    goPlayerPage: function () {
      this.$emit("goPlayerPage", this.current);
    },
    goNormalPage: function () {
      this.$emit("goNormalPage", this.current);
    },
    playerMax: function () {
      if ($("#player").hasClass("player3")) {
        $("#player").addClass("player");
        $("#player").removeClass("player3");
        $("#player").css({
          background: "rgba(255, 255, 255, 0.8)",
          "background-size": "100% 100%",
        });
        this.goNormalPage();
      } else {
        $("#player").addClass("player3");
        $("#player").removeClass("player");
        $("#player").css({
          background: `url('${this.current.cover}')  center center no-repeat`,
          "background-size": "0",
        });
        this.goPlayerPage();
      }
    },
    playerNarrow: function () {
      if ($("#player").hasClass("player3")) {
        $("#player").addClass("player");
        $("#player").removeClass("player3");
        $("#player").css({
          background: "rgba(255, 255, 255, 0.8)",
          "background-size": "100% 100%",
        });
      }
    },
    startPlayOrPause() {
      if (!this.audio.playing) {
        this.play = true;
        this.$refs.audio.play();
      } else {
        this.$refs.audio.pause();
        this.play = false;
      }
    },
    nextAudio() {
      var that = this;
      if (this.playlist.length > 1) {
        $(".player-controls").removeClass("active");
        $(".play-pause").attr("class", "btn play-pause icon-kaishi iconfont");
        if (this.playlist.length - 1 == this.plw) {
          this.plw = -1;
        }
        this.plw++;
        this.current = this.playlist[this.plw];
        this.updateData();
        this.sliderTime = 0;
        this.thumbTime = 0;
        this.audio.currentTime = "00:00";
        this.play = true;
        $(".player3").css({
          background: `url('${this.current.cover}')  center center no-repeat`,
          "background-size": "0",
        });
        that.silderNone = 0;
        times = setInterval(() => {
          window.clearInterval(times);
          this.$refs.audio.play();
        }, 1000);
      }
    },
    prevAudio() {
      var that = this;
      if (this.playlist.length > 1) {
        $(".player-controls").removeClass("active");
        $(".play-pause").attr("class", "btn play-pause icon-kaishi iconfont");
        if (this.plw == 0) {
          this.plw = this.playlist.length;
        }
        this.plw--;
        this.current = this.playlist[this.plw];
        this.updateData();
        this.sliderTime = 0;
        this.thumbTime = 0;
        this.audio.currentTime = "00:00";
        this.play = true;
        $(".player3").css({
          background: `url('${this.current.cover}')  center center no-repeat`,
          "background-size": "0",
        });
        that.silderNone = 0;
        times = setInterval(() => {
          window.clearInterval(times);
          this.$refs.audio.play();
        }, 1000);
        this.$refs.audio.play();
      }
    },
    playaudio() {
      this.$refs.audio.play();
      this.play = true;
    },
    pause() {
      this.$refs.audio.pause();
    },
    onPlay() {
      this.audio.playing = true;
      $(".player-controls").addClass("active");
      $(".play-pause").attr("class", "btn play-pause icon-zanting iconfont");
    },
    onPause() {
      this.audio.playing = false;
      if (this.audio.currentTime >= this.audio.maxTime) {
        this.nextAudio();
      }
      $(".player-controls").removeClass("active");
      $(".play-pause").attr("class", "btn play-pause icon-kaishi iconfont");
    },
    onLoadedMetaData(res) {
      this.audio.maxTime = parseInt(res.target.duration);
    },
    onTimeUpdate(res) {
      if (this.hoverTime == 0) {
        this.audio.currentTime = res.target.currentTime;
      }
      if (this.hoverTime == 0) {
        this.sliderTime = (res.target.currentTime / this.audio.maxTime) * 100;
        this.thumbTime = (res.target.currentTime / this.audio.maxTime) * 100;
      }
    },
    setValue(e) {
      const { maxTime, minTime, step } = this.audio;
      let value =
        ((e.clientX - $("#s-area").offset().left) / $("#s-area").width()) *
        (maxTime - minTime);
      value = Math.round(value / step) * step + minTime;
      value = parseFloat(value.toFixed(5));
      if (value > maxTime) {
        value = maxTime;
      } else if (value < minTime) {
        value = minTime;
      }
      this.$refs.audio.currentTime = value;
    },
    sethover(e) {
      const { maxTime, minTime, step } = this.audio;
      let value =
        ((e.clientX - $("#s-area").offset().left) / $("#s-area").width()) *
        (maxTime - minTime);
      value = Math.round(value / step) * step + minTime;
      value = parseFloat(value.toFixed(5));
      this.hoverTime =
        ((e.clientX - $("#s-area").offset().left) / $("#s-area").width()) * 100;
      this.audio.currentTime = value;
      this.thumbTime = this.hoverTime;
      this.sliderTime = this.hoverTime;
    },
    hideHover() {
      this.hoverTime = 0;
      this.audio.currentTime = this.$refs.audio.currentTime;
      this.onTimeUpdate({ target: { currentTime: this.audio.currentTime } });
    },
    handleTouchStart(e) {
      this.setValue(e);
      this.$refs.audio.play();
    },
  },
  filters: {
    formatSecond(second = 0) {
      return realFormatSecond(second);
    },
  },
};
</script>
<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
    "Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
    sans-serif;
}

a {
  display: inline;
}
.player2 {
  z-index: 2000;
  background: rgba(255, 255, 255, 0.8);
  border-radius: 20px;
  box-shadow: 0 30px 80px #656565;
  bottom: 10px;
  padding: 20px 15px;
  height: 60vh;
  white-space: nowrap;
  transition: 0.8s;
}

.player {
  z-index: 2000;
  position: fixed;
  background: rgba(255, 255, 255, 0.8);
  border-radius: 20px;
  box-shadow: 0 30px 80px #656565;
  bottom: 10px;
  max-width: 165px;
  height: 55px;
  padding: 20px 15px;
  white-space: nowrap;
  left: 10px;
  right: 10px;
  transition: max-width 0.8s, height 0.8s;
  overflow: hidden;
}

.player.max {
  max-width: 100%;
  background: rgba(255, 255, 255, 0.9);
}

.player3 {
  z-index: 2000;
  position: fixed;
  border-radius: 20px;
  padding: 20px 15px;
  white-space: nowrap;
  left: 0;
  right: 0;
  bottom: 0;
  border-radius: 0;
  height: 100%;
  max-width: 100%;
  /* background: rgba(66, 66, 66, 0.9); */
  -webkit-backdrop-filter: blur(20px);
  backdrop-filter: blur(20px);
  transition: max-width 0.8s, height 0.8s;
  overflow: hidden;
}

.player3::before {
  content: "";
  top: -25%;
  left: -25%;
  right: 0;
  bottom: 0;
  width: 150%;
  height: 150%;
  position: fixed;
  background-image: inherit;
  /* background: url("/img/background-dark.png") no-repeat center center; */
  background-size: 100% 100%;
  /* background-attachment: fixed; */
  filter: blur(100px);
  z-index: 2001;
  transition: 0.8s;
  /* -webkit-animation: rotateAlbumArt 50s linear 0s infinite forwards;
    animation: rotateAlbumArt 50s linear 0s infinite forwards; */
}

.player div {
  display: inline;
  color: #000;
  white-space: nowrap;
}

.player2 div {
  display: inline;
  color: #000;
  white-space: nowrap;
}

.player3 div {
  color: #000;
  white-space: nowrap;
}

.player-controls {
  display: block;
  position: relative;
  width: 100%;
  height: auto;
  border-radius: 20px;
}

.player3 .player-controls {
  width: 64vh;
  padding: 7vh;
}

.player .music-imgs {
  position: absolute;
  top: -9px;
  width: 33px;
  height: 33px;
  -webkit-transform: rotateZ(0);
  transform: rotateZ(0);
  transition: 0.3s ease all;
  border-radius: 50%;
  overflow: hidden;
  box-shadow: 0 0px 5px #656565;
  z-index: 2002;
}

.player2 .music-imgs {
  position: absolute;
  top: -9px;
  width: 45px;
  height: 45px;
  -webkit-transform: rotateZ(0);
  transform: rotateZ(0);
  transition: 0.3s ease all;
  border-radius: 50%;
  overflow: hidden;
  box-shadow: 0 0px 5px #656565;
  z-index: 2002;
}

.player3 .music-imgs {
  position: relative;
  top: -9px;
  width: 50vh;
  height: 50vh;
  -webkit-transform: rotateZ(0);
  transform: rotateZ(0);
  transition: 0.8s all;
  border-radius: 10%;
  /* overflow: hidden; */
  text-align: center;
  z-index: 2002;
}

@media screen and (orientation: portrait) {
  .player3 .player-controls {
    width: 94vw;
    padding: 7vw;
  }
  .player3 .music-imgs {
    width: 80vw;
    height: 80vw;
  }
}

.player .music-imgs:hover {
  -webkit-filter: brightness(80%);
  filter: brightness(80%);
}

.player2 .music-imgs:hover {
  -webkit-filter: brightness(80%);
  filter: brightness(80%);
}

.player .music-imgs .img {
  display: inline;
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1000;
  -webkit-animation: rotateAlbumArt 10s linear 0s infinite forwards;
  animation: rotateAlbumArt 10s linear 0s infinite forwards;
  animation-play-state: paused;
  object-fit: cover;
}

.player2 .music-imgs .img {
  display: inline;
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1000;
  -webkit-animation: rotateAlbumArt 10s linear 0s infinite forwards;
  animation: rotateAlbumArt 10s linear 0s infinite forwards;
  animation-play-state: paused;
}

.player3 .player-controls .music-imgs .img {
  display: inline-block;
  position: relative;
  border-radius: 10%;
  top: 15%;
  left: 0;
  width: 70%;
  height: 70%;
  z-index: 1000;
  transition: 0.5s;
  transition: 0.8s;
  box-shadow: 0 0px 10px #5a5a5a54;
  object-fit: cover;
}

.player3 .player-controls.active .music-imgs .img {
  display: inline-block;
  position: relative;
  border-radius: 10%;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 2002;
}

.player-controls.active .music-imgs .img {
  z-index: 1;
  animation-play-state: running;
}

.player-controls.active .music-imgs {
  box-shadow: 0 0px 10px #656565;
}

.player3 .player-controls.active .music-imgs {
  box-shadow: none;
}

@-webkit-keyframes rotateAlbumArt {
  0% {
    -webkit-transform: rotateZ(0);
    transform: rotateZ(0);
  }
  100% {
    -webkit-transform: rotateZ(360deg);
    transform: rotateZ(360deg);
  }
}

@keyframes rotateAlbumArt {
  0% {
    -webkit-transform: rotateZ(0);
    transform: rotateZ(0);
  }
  100% {
    -webkit-transform: rotateZ(360deg);
    transform: rotateZ(360deg);
  }
}

#player-content3 {
  position: relative;
  margin-left: 50px;
}

.player .player-controls3 {
  display: none;
}

.player2 .player-controls3 {
  display: none;
}

.player3 #player-content3 {
  display: none !important;
}

.player3 .player-controls3 {
  position: relative;
  margin-left: 0;
  display: block;
  white-space: nowrap;
  text-align: center;
  margin-top: 30px;
  transition: 0.8s;
  z-index: 2002;
}

.player3 .player-controls3 * {
  display: inline;
  white-space: nowrap;
  z-index: 2002;
  color: #ffffff;
}

.player .btn {
  padding-left: 5px;
  padding-right: 5px;
  transition: 0.5s;
}

.player2 .btn {
  z-index: 100000;
  padding-left: 5px;
  padding-right: 5px;
  transition: 0.5s;
  -webkit-filter: invert(0);
  filter: invert(0);
}

.player3 .btn {
  z-index: 100000;
  padding-left: 20px;
  padding-right: 20px;
  transition: 0.2s;
  font-size: 4vh;
  z-index: 2002;
}

.player .btn:hover {
  -webkit-filter: invert(0.5);
  filter: invert(0.5);
}

.player2 .btn:hover {
  -webkit-filter: invert(0.5);
  filter: invert(0.5);
}

.player3 .btn:hover {
  -webkit-filter: invert(0.1);
  filter: invert(0.1);
}

#player-content2 {
  display: none;
  opacity: 0;
}

.player.max #player-content2 {
  display: inline-block;
  opacity: 1;
}

.player2.max #player-content2 {
  display: inline-block;
  opacity: 1;
}

.player3 #player-content2 {
  justify-content: space-between;
  flex-basis: 0;
  display: flex;
  flex-direction: column;
  opacity: 1;
  color: #f7f7f7;
  margin-top: 16px;
  z-index: 2002;
}

.player3 #player-content2 * {
  color: #f7f7f7;
  z-index: 2002;
}

.player .time {
  margin-left: 10px;
  margin-right: 10px;
  display: inline-block;
  width: auto;
  right: 0;
  left: 60vw;
  position: absolute;
  font-size: 10px;
}

.player2 .time {
  margin-left: 10px;
  margin-right: 10px;
  display: inline-block;
  width: auto;
  right: 0;
  left: 60vw;
  position: absolute;
  font-size: 10px;
}

.player3 .time {
  margin-left: 10px;
  margin-right: 10px;
  display: block;
  width: auto;
  right: 0;
  position: relative;
  font-size: 1vw;
  white-space: nowrap;
  margin-top: 10px;
  margin-bottom: 5px;
}

@media screen and (orientation: portrait) {
  .player3 .time {
    font-size: 1.5vh;
  }
}

.player .time .total-time {
  float: right;
}

.player2 .time .total-time {
  float: right;
}

.player3 .time .current-time {
  display: inline;
}

.player3 .time .total-time {
  float: right;
}

.player .music-name {
  display: inline-block;
  max-width: calc(100vw - 217px - 40vw);
  overflow: hidden;
}

.player2 .music-name {
  display: inline;
  max-width: calc(100vw - 217px - 40vw);
  overflow: hidden;
}

.player3 .music-name {
  display: inline;
  font-size: 5vh;
  font-weight: 500;
}

.player3 .artist-name {
  display: inline;
  font-size: 4vh;
  font-weight: 500;
  color: rgba(255, 255, 255, 0.575) !important;
}

#s-area,
#seek-bar {
  position: relative;
  height: 5px;
  border-radius: 4px;
}

#s-area {
  background-color: #fdfdfd6e;
  cursor: pointer;
  width: auto;
  display: block;
  position: absolute;
  right: 0;
  left: 60vw;
}

.player3 #s-area {
  position: relative;
  left: 0;
}

#ins-time {
  position: absolute;
  top: -29px;
  color: #fff;
  font-size: 12px;
  white-space: pre;
  padding: 5px 6px;
  border-radius: 4px;
  display: none;
}

#s-hover {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  opacity: 0.2;
  z-index: 2100 !important;
}

#ins-time,
#s-hover {
  background-color: #9b9b9b;
}

#seek-bar {
  content: "";
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  width: 0;
  background-color: rgba(250, 250, 250, 0.801);
  transition: 0.2s ease width;
}

.slider {
  background-color: #f3f3f3ea;
  cursor: pointer;
  width: auto;
  display: block;
  position: absolute;
  right: 0;
  left: 60vw;
}

.silderNone {
  background-color: rgb(231, 231, 231);
  content: "";
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  width: 0;
  transition: 0.2s ease width;
}

.slider-track {
  position: absolute;
  top: -29px;
  color: #fff;
  font-size: 12px;
  white-space: pre;
  padding: 5px 6px;
  border-radius: 4px;
  display: none;
}

.slider-fill {
  content: "";
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  width: 0;
  background-color: rgba(255, 255, 255, 0.733);
  transition: 0.2s ease width;
}

.slider-thumb {
  position: absolute;
  width: 0.5rem;
  top: 0.15rem;
  /* top: 2.18px; */
  height: 0.5rem;
  background-color: rgba(255, 255, 255);
  border-radius: 50%;
  transform: translate(-50%, -50%);
  cursor: pointer;
  transition: 0.2s ease width;
}

.max2cd {
  display: none !important;
}

.player3 .max2cd {
  display: inline !important;
  position: absolute;
  color: #f7f7f7;
}

.max2d {
  display: unset;
}

.player3 .max2d {
  display: none;
}
@font-face {
  font-family: "iconfont";
  src: url("//at.alicdn.com/t/font_2293529_hsibd11944.eot?t=1616507953830"); /* IE9 */
  src: url("//at.alicdn.com/t/font_2293529_hsibd11944.eot?t=1616507953830#iefix")
      format("embedded-opentype"),
    /* IE6-IE8 */
      url("data:application/x-font-woff2;charset=utf-8;base64,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")
      format("woff2"),
    url("//at.alicdn.com/t/font_2293529_hsibd11944.woff?t=1616507953830")
      format("woff"),
    url("//at.alicdn.com/t/font_2293529_hsibd11944.ttf?t=1616507953830")
      format("truetype"),
    /* chrome, firefox, opera, Safari, Android, iOS 4.2+ */
      url("//at.alicdn.com/t/font_2293529_hsibd11944.svg?t=1616507953830#iconfont")
      format("svg"); /* iOS 4.1- */
}

.iconfont {
  font-family: "iconfont" !important;
  font-size: 16px;
  font-style: normal;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.icon-spotify:before {
  content: "\e807";
}

.icon-applemusic:before {
  content: "\e689";
}

.icon-Youtube:before {
  content: "\e62f";
}

.icon-qqmusic-fill-round:before {
  content: "\e68d";
}

.icon-bilibili-fill:before {
  content: "\e93e";
}

.icon-netease-cloud-music-fill:before {
  content: "\e7d6";
}

.icon-maximise:before {
  content: "\e65d";
}

.icon-fangdazhanshi:before {
  content: "\e60c";
}

.icon-shanchu:before {
  content: "\e60b";
}

.icon-bendisucai:before {
  content: "\e614";
}

.icon-diyigeshipin:before {
  content: "\e60e";
}

.icon-kaishi:before {
  content: "\e610";
}

.icon-zuihouyigeshipin:before {
  content: "\e611";
}

.icon-zanting:before {
  content: "\e612";
}

.icon-bianji:before {
  content: "\e61b";
}

.icon-wangye-loading:before {
  content: "\e62e";
}

.icon-suoxiao:before {
  content: "\e635";
}

.icon-list:before {
  content: "\e613";
}

.icon-drop-down:before {
  content: "\e604";
}

.icon-language:before {
  content: "\e6e3";
}

.icon-setting:before {
  content: "\e605";
}

.icon-good:before {
  content: "\e62a";
}

.icon-good1:before {
  content: "\e6f7";
}

.icon-log-out:before {
  content: "\e64a";
}

.icon-star:before {
  content: "\e60f";
}

.icon-find:before {
  content: "\e602";
}

.icon-write:before {
  content: "\e600";
}

.icon-avatar:before {
  content: "\e6a7";
}

.icon-share:before {
  content: "\e67a";
}

.icon-info:before {
  content: "\e62d";
}

.icon-upload:before {
  content: "\e84c";
}
</style>