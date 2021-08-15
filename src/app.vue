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
            id="img"
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
            <div
              id="highEnergy"
              class="slider-high-energy"
              v-show="current.highEnergy"
              :style="
                'left:' +
                (realFormatFromSecond(current.highEnergy) / audio.maxTime) *
                  100 +
                '%'
              "
            ></div>
            <div id="s-hover" :style="'width:' + hoverTime + '%'"></div>
            <div class="slider-fill" :style="'width:' + sliderTime + '%'"></div>
            <div
              id="thumb"
              @touchmove="onTouchMove"
              @touchend="onTouchEnd"
              class="slider-thumb"
              :style="'left:' + thumbTime + '%'"
            ></div>
          </div>
        </div>
        <div class="player-controls3">
          <div
            @click="changeMode" 
            id='player_mode'
            class="btn mode iconfont"
          ></div>
          <div>
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
          <div
            class="btn list iconfont icon-list"
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
    dse: {
      type: Boolean,
      default: false,
    },
  },
  data: function () {
    return {
      plw: 0,

      sliderTime: 0,
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
        highEnergy: null,
      },
      touch: false,
      mode_icon: {
        "loop": "icon-xunhuan",
        "single": "icon-danquxunhuan",
        "random": "icon-suiji"
      },
      mode: "loop",
      random_list: [],
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
      this.plw = parseInt(localStorage.getItem("Player_plw"));
    }
    setInterval(() => {
      this.$emit("updateData", this.current);
    }, 1000);
    if (this.dse) {
      $("#player").addClass("dse");
    }
    var mode = ['single', 'loop', 'random']
    if (localStorage.getItem("Player_mode")) {
      if (mode.indexOf(localStorage.getItem("Player_mode")) != -1){
        $("#player_mode").addClass(this.mode_icon[localStorage.getItem("Player_mode")]);
        this.mode = localStorage.getItem("Player_mode")
      }
      else {
        localStorage.setItem("Player_mode", 'loop');
        $("#player_mode").addClass('icon-xunhuan');
      }
    } else {
      localStorage.setItem("Player_mode", 'loop');
      $("#player_mode").addClass('icon-xunhuan');
    }
  },
  watch: {
    playlist(list) {
      if (list[this.plw]) {
        new Promise((resolve) => {
          var i = 0
          while (list[i]) {
            var random = Math.floor((Math.random() * list.length))
            while(this.random_list.indexOf(random) != -1) {
              random = Math.floor((Math.random() * list.length))
            }
            this.random_list[i] = random
            i++
          }
          resolve()
        }).then(()=>{
          if (this.mode != 'random') {
            this.current = list[this.plw];
          }
          else {
            this.current = list[this.random_list[this.plw]]
          }
        })
      } else {
        this.current = {
          startTime: "00:00",
          endtime: "00:00",
          cover: "",
          name: "",
          artist: "",
          source: "",
          highEnergy: null,
        };
      }
    },
    plw(plw) {
      if (this.localplw) {
        localStorage.setItem("Player_plw", plw);
      }
    },
    dse() {
      if (this.dse) {
        $("#player").addClass("dse");
      } else {
        $("#player").removeClass("dse");
      }
    },
    mode(n, l) {
      $("#player_mode").removeClass(this.mode_icon[l]);
      $("#player_mode").addClass(this.mode_icon[n]);
      localStorage.setItem('Player_mode', this.mode)
    }
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
        $("#img").css({ top: "0", width: "100%", height: "100%" });
        this.goNormalPage();
      } else {
        $("#player").addClass("player3");
        $("#player").removeClass("player");
        if (!this.dse) {
          $("#player").css({
            background: `url('${this.current.cover}') no-repeat center center`,
            "background-size": "0",
          });
        } else {
          $("#player").css({
            background: "rgba(150, 150, 150)",
            "background-size": "100% 100%",
          });
        }
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
        $("#img").animate(
          { top: "15%", width: "70%", height: "70%" },
          500,
          "swing"
        );
        $(".player-controls").removeClass("active");
        $(".play-pause").attr("class", "btn play-pause icon-kaishi iconfont");
        if (this.playlist.length - 1 == this.plw) {
          this.plw = -1;
        }
        this.plw++;
        if (this.mode != 'random') {
          this.current = this.playlist[this.plw];
        }
        else {
          this.current = this.playlist[this.random_list[this.plw]];
        }
        this.updateData();
        this.sliderTime = 0;
        this.thumbTime = 0;
        this.audio.currentTime = "00:00";
        this.play = true;
        if (!this.dse) {
          $(".player3").css({
            background: `url('${this.current.cover}')  center center no-repeat`,
            "background-size": "0",
          });
        }
        times = setInterval(() => {
          window.clearInterval(times);
          this.$refs.audio.play();
        }, 1000);
      }
    },
    prevAudio() {
      var that = this;
      if (this.playlist.length > 1) {
        $("#img").animate(
          { top: "15%", width: "70%", height: "70%" },
          500,
          "swing"
        );
        $(".player-controls").removeClass("active");
        $(".play-pause").attr("class", "btn play-pause icon-kaishi iconfont");
        if (this.plw == 0) {
          this.plw = this.playlist.length;
        }
        this.plw--;
        if (this.mode != 'random') {
          this.current = this.playlist[this.plw];
        }
        else {
          this.current = this.playlist[this.random_list[this.plw]];
        }
        this.updateData();
        this.sliderTime = 0;
        this.thumbTime = 0;
        this.audio.currentTime = "00:00";
        this.play = true;
        if (!this.dse) {
          $(".player3").css({
            background: `url('${this.current.cover}')  center center no-repeat`,
            "background-size": "0",
          });
        }
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
      $("#img").animate(
        { top: "0", width: "100%", height: "100%" },
        800,
        "swing"
      );
      this.audio.playing = true;
      $(".player-controls").addClass("active");
      $(".play-pause").attr("class", "btn play-pause icon-zanting iconfont");
    },
    onPause() {
      $("#img").animate(
        { top: "15%", width: "70%", height: "70%" },
        800,
        "swing"
      );
      this.audio.playing = false;
      if (this.audio.currentTime >= this.audio.maxTime) {
        if (this.mode != 'single') {
          this.nextAudio();
        }
        else {
          this.$refs.audio.currentTime = 0;
          this.$refs.audio.play();
        }
      }
      $(".player-controls").removeClass("active");
      $(".play-pause").attr("class", "btn play-pause icon-kaishi iconfont");
    },
    onLoadedMetaData(res) {
      this.audio.maxTime = parseInt(res.target.duration);
    },
    onTimeUpdate(res) {
      this.audio.currentTime = res.target.currentTime;
      this.sliderTime = (res.target.currentTime / this.audio.maxTime) * 100;
      this.thumbTime = (res.target.currentTime / this.audio.maxTime) * 100;
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
    },
    hideHover() {
      this.hoverTime = 0;
    },
    handleTouchStart(e) {
      this.setValue(e);
      this.$refs.audio.play();
    },
    onTouchMove(e) {
      this.$refs.audio.pause();
      const { maxTime, minTime, step } = this.audio;
      let value =
        ((e.touches[0].clientX - $("#s-area").offset().left) /
          $("#s-area").width()) *
        (maxTime - minTime);
      value = Math.round(value / step) * step + minTime;
      value = parseFloat(value.toFixed(5));
      if (value > maxTime) {
        value = maxTime;
      } else if (value < minTime) {
        value = minTime;
      }
      this.$refs.audio.currentTime = value;
      this.touch = true;
      $("#thumb").addClass("touch");
    },
    onTouchEnd() {
      this.$refs.audio.play();
      this.touch = false;
      $("#thumb").removeClass("touch");
    },
    realFormatFromSecond(second) {
      if (second) {
        var zc = ["", "", ""];
        var n = 0;
        for (var i = 0; second[i]; i++) {
          if (second[i] == ":") {
            zc[n] = parseInt(zc[n]);
            n++;
          } else {
            zc[n] += second[i];
          }
        }
        var out = zc[0] * 60 + zc[1] * 1 + (zc[2] / 60) * 0.1;
      }
      return out;
    },
    changeMode() {
      if (this.mode == 'loop'){
        this.mode = 'single'
      }
      else if (this.mode == 'single') {
        this.mode = 'random'
        this.plw = this.random_list.indexOf(this.plw)
      }
      else if (this.mode == 'random') {
        this.mode = 'loop'
        this.plw = this.random_list[this.plw]
      }
    }
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
  line-height: normal;
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

.dse.player {
  box-shadow: unset;
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
  -webkit-backdrop-filter: blur(20px);
  backdrop-filter: blur(20px);
  transition: max-width 0.8s, height 0.8s;
  overflow: hidden;
}

.dse.player3 {
  -webkit-backdrop-filter: unset;
  backdrop-filter: unset;
}

.player3::before {
  content: "";
  left: 0;
  bottom: 0;
  max-width: inherit;
  width: 100%;
  height: inherit;
  position: fixed;
  background-image: inherit;
  background-size: 100% 100%;
  /*background-attachment: fixed; */
  filter: blur(100px);
  z-index: 2001;
  /* transition: 0.8s; */
}

.dse.player3::before {
  filter: unset;
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
    width: 100%;
    padding: 7vw;
  }
  .player3 .music-imgs {
    width: calc(84vw - 30px);
    height: calc(84vw - 30px);
    /*max-width: 80vw;*/
    /*max-height: 80vw;*/
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
  box-shadow: 0 0px 10px #5a5a5a54;
  object-fit: cover;
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
  display: flex;
  white-space: nowrap;
  text-align: center;
  margin-top: 30px;
  transition: 0.8s;
  z-index: 2002;
  align-items: center;
  justify-content: space-between;
}

.player3 .player-controls3 * {
  display: inline;
  white-space: nowrap;
  z-index: 2002;
  color: #ffffff;
  align-items: center;
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
  transition: filter 0.8s;
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

.btn.mode,.btn.list {
  font-size: 26px;
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
  background-color: rgba(255, 255, 255, 0.7);
  z-index: 2001 !important;
}

#ins-time,
#s-hover {
  background-color: #cccccc;
}

#seek-bar {
  content: "";
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  width: 0;
  background-color: rgba(250, 250, 250, 0.8);
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

.slider-fill {
  content: "";
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  width: 0;
  background-color: rgba(255, 255, 255, 1);
  transition: 0.2s ease width;
}

.slider-thumb {
  position: absolute;
  width: 9px;
  top: -2px;
  height: 9px;
  background-color: rgba(255, 255, 255);
  border-radius: 50%;
  transform: translate(-50%, 0);
  cursor: pointer;
  transition: 0.8s ease width;
}

.slider-high-energy {
  position: absolute;
  width: 6px;
  top: -0.5px;
  height: 6px;
  background-color: rgba(255, 255, 255, 0.6);
  border-radius: 50%;
  transform: translate(-50%, 0);
  cursor: pointer;
  transition: 0.8s ease width;
}

.slider-thumb.touch {
  width: 15px;
  top: -5px;
  height: 15px;
  transform: translate(-50%, 0);
}

.max2cd {
  display: none !important;
}

.player3 .max2cd {
  display: inline !important;
  position: relative;
  color: #f7f7f7;
}

.max2d {
  display: unset;
}

.player3 .max2d {
  display: none;
}

@font-face {
  font-family: "iconfont"; /* Project id 2293529 */
  src: url('//at.alicdn.com/t/font_2293529_d0yf2d8oxo.woff2?t=1629018034235') format('woff2'),
       url('//at.alicdn.com/t/font_2293529_d0yf2d8oxo.woff?t=1629018034235') format('woff'),
       url('//at.alicdn.com/t/font_2293529_d0yf2d8oxo.ttf?t=1629018034235') format('truetype');
}

.iconfont {
  font-family: "iconfont" !important;
  font-size: 16px;
  font-style: normal;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.icon-suiji:before {
  content: "\e7b8";
}

.icon-xunhuan:before {
  content: "\e68e";
}

.icon-danquxunhuan:before {
  content: "\e6a2";
}

.icon-bianji1:before {
  content: "\e89a";
}

.icon-fangda:before {
  content: "\e89b";
}

.icon-dianzan:before {
  content: "\e89c";
}

.icon-jubao:before {
  content: "\e89d";
}

.icon-lianjie:before {
  content: "\e89e";
}

.icon-shuoming:before {
  content: "\e89f";
}

.icon-shoucang:before {
  content: "\e8a0";
}

.icon-liaotian:before {
  content: "\e8a1";
}

.icon-sousuo:before {
  content: "\e8a2";
}

.icon-suoxiao1:before {
  content: "\e8a3";
}

.icon-wenjianjia:before {
  content: "\e8a4";
}

.icon-shezhi:before {
  content: "\e8a5";
}

.icon-yinle:before {
  content: "\e8a6";
}

.icon-zhuanfa:before {
  content: "\e8a7";
}

.icon-zhuye:before {
  content: "\e8a8";
}

.icon-wenjianheimingdan:before {
  content: "\e8a9";
}

.icon-wenjianyinle:before {
  content: "\e8aa";
}

.icon-wenjianshoucang:before {
  content: "\e8ab";
}

.icon-wenjian:before {
  content: "\e8ac";
}

.icon-wenjianshangchuan:before {
  content: "\e8ad";
}

.icon-yunshangchuan:before {
  content: "\e8ae";
}

.icon-yunyinle:before {
  content: "\e8af";
}

.icon-yunshuaxin:before {
  content: "\e8b0";
}

.icon-yun:before {
  content: "\e8b1";
}

.icon-list1:before {
  content: "\e615";
}

.icon-GitHub:before {
  content: "\e601";
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

.icon-log-out:before {
  content: "\e64a";
}

.icon-find:before {
  content: "\e602";
}

.icon-avatar:before {
  content: "\e6a7";
}

.icon-info:before {
  content: "\e62d";
}

</style>