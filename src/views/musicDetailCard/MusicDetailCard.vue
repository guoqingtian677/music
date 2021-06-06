<template>
  <div class="musicDetailCard" :class="isMusicDetailCardShow ? '' : 'hide'">
    <div class="closeCard" @click="closeCard">
      <i class="iconfont icon-arrow-down-bold"></i>
    </div>
    <div class="musicDetailCardContainer" v-if="!cleanCard && musicInfo.name">
      <div class="top">
        <div class="left">
          <div class="discContainer">
            <div
              class="needle"
              :class="$store.state.isPlay ? 'needleRotate' : ''"
              ref="needle"
            >
              <img src="~assets/img/MusicDetailCard/needle.png" alt="" />
            </div>
            <!-- 通过音乐的加载时差删除discAnimation类名再添加,达到重置动画的效果 -->
            <div
              class="disc"
              :class="[
                $store.state.isPlay ? '' : 'pause',
                $store.state.isMusicLoad ? '' : 'discAnimation',
              ]"
              ref="disc"
            >
              <img src="~assets/img/MusicDetailCard/disc.png" alt="" />
              <img
                src="~assets/img/test.jpg"
                alt=""
                class="musicAvatar"
                v-if="!musicInfo.al"
              />
              <img
                :src="musicInfo.al.picUrl"
                alt=""
                class="musicAvatar"
                v-else
              />
            </div>
          </div>
        </div>
        <div class="right">
          <div class="title">
            <div class="musicName">{{ musicInfo.name }}</div>
            <div class="album">{{ musicInfo.al.name }}</div>
            <div class="singer">
              {{ musicInfo.ar[0].name }}
            </div>
          </div>
          <lyrics-scroll :lyric="lyric"></lyrics-scroll>
        </div>
      </div>
      <div class="bottom">
        <comment :comments="comment.comments"
          ><div slot="title">全部评论({{ comment.totalCount }})</div></comment
        >
        <!-- 评论分页 -->
        <div class="page" v-show="comment.comments && comment.comments[0]">
          <el-pagination
            background
            layout="prev, pager, next"
            :total="comment.totalCount"
            small
            :page-size="20"
            :current-page="currentCommentPage"
            @current-change="commentPageChange"
          >
          </el-pagination>
        </div>
      </div>
    </div>
    <div v-else class="tip">暂无播放音乐</div>
  </div>
</template>

<script>
import Comment from "components/comment/Comment.vue";
import LyricsScroll from "components/lyricsScroll/LyricsScroll.vue";

// 定时器名称
let timer;

export default {
  components: { LyricsScroll, Comment },
  name: "MusicDetailCard",
  data() {
    return {
      // 是否显示歌曲详情卡片
      isMusicDetailCardShow: false,
      //   歌词
      lyric: [[0, "正在加载歌词"]],
      //当前评论页数
      currentCommentPage: 1,
      // 评论数据
      comment: {},
      //   当前歌曲信息
      musicInfo: {},
      // 是否删除卡片渲染的内容
      cleanCard: false,
    };
  },
  watch: {
    "$store.state.isMusicDetailCardShow"(isMusicDetailCardShow) {
      this.isMusicDetailCardShow = isMusicDetailCardShow;
      //   性能优化
      //如果卡片没有展示就不发送请求和渲染里面的内容
      //   删除定时器 避免用户在关闭卡片的1s内又打开卡片 导致展示内容被删除
      clearTimeout(timer);
      this.cleanCard = false;
      if (
        isMusicDetailCardShow &&
        !this.comment.comments &&
        this.$store.state.musicId != ""
      ) {
        console.log("请求歌词和评论");
        this.getLyric(this.$store.state.musicId);
        this.getMusicComment(this.$store.state.musicId);
      }
      //   当卡片关闭时 延迟 3s再删除里面渲染的内容 提升体验
      else if (isMusicDetailCardShow == false) {
        timer = setTimeout(() => {
          this.cleanCard = true;
        }, 3000);
      }
    },
    // 当vuex中的歌曲id发生变化时,需要重新获取评论和歌词
    "$store.state.musicId"(musicId) {
      // 清空歌词
      this.lyric = [[0, "正在加载歌词"]];
      // 重置评论页数
      this.currentCommentPage = 1;
      // 更新当前歌曲信息
      this.musicInfo = this.$store.state.musicList[
        this.$store.state.currentIndex
      ];
      this.comment = {};
      // 优化性能,仅在卡片展示时才发送请求
      if (this.isMusicDetailCardShow) {
        this.getLyric(musicId);
        this.getMusicComment(musicId);
      }
      //   console.log(this.musicInfo);
      //   console.log(this.$refs.disc);
    },
  },
  methods: {
    // 请求
    //请求并处理歌词数据
    async getLyric(id) {
      let res = await this.$request("/lyric", { id });
      //   console.log(res);
      let lyrics = res.data.lrc.lyric;
      // 对获取到的歌词进行处理
      let arr = lyrics.split("\n");
      let array = [];
      // let obj = {};
      let time = "";
      let value = "";
      let result = [];

      //去除空行
      arr.forEach((item) => {
        if (item != "") {
          array.push(item);
        }
      });
      arr = array;
      arr.forEach((item) => {
        time = item.split("]")[0];
        value = item.split("]")[1];
        //去掉时间里的中括号得到xx:xx.xx
        var t = time.slice(1).split(":");
        //将结果压入最终数组
        result.push([parseInt(t[0], 10) * 60 + parseFloat(t[1]), value]);
      });

      this.lyric = result;
      // console.log(this.lyric);
    },
    // 请求评论数据
    async getMusicComment(id) {
      let res = await this.$request("/comment/new", {
        id,
        pageNo: this.currentCommentPage,
        sortType: 2,
        type: 0,
      });
      //   console.log(res);
      this.comment = res.data.data;
    },
    //点击分页按钮的回调
    commentPageChange(page) {
      this.comment.comments = [];
      this.currentCommentPage = page;
      this.getMusicComment(this.$store.state.musicId);
    },
    closeCard() {
      this.$store.commit("changeMusicDetailCardState");
    },
  },
  created() {},
};
</script>

<style scoped>
.musicDetailCard {
  position: absolute;
  width: 100%;
  height: calc(100vh - 55px);
  background-color: white;
  top: 0;
  left: 0;
  z-index: 1000;
  transition: all 0.5s;
  background-image: linear-gradient(to bottom, #e3e2e3, white);
}

.closeCard .iconfont {
  position: absolute;
  top: 15px;
  left: 35px;
  font-size: 21px !important;
}

.musicDetailCardContainer {
  height: 100%;
  overflow-y: scroll;
}

.hide {
  transform: translateY(calc(100vh - 55px));
}

.top {
  display: flex;
  justify-content: center;
}

.left {
  width: 220px;
  margin: 0 30px 0 -40px;
}

.discContainer {
  margin-top: 60px;
  width: 220px;
  position: relative;
}

.needle {
  position: relative;
  left: 50%;
  width: 88px;
  height: 72px;
  z-index: 10000;
  transition: all 1s;
  transform-origin: 5.3px 5.3px;
}

.needle img {
  width: 100%;
}

.needleRotate {
  transform-origin: 5.3px 5.3px;
  transform: rotate(22deg);
}

.disc {
  width: 220px;
  height: 220px;
  position: relative;
  top: -12px;
}

.disc img:nth-child(1) {
  width: 100%;
}

.disc .musicAvatar {
  position: absolute;
  top: 35px;
  left: 35px;
  width: 150px;
  z-index: -1;
}

/* 碟子设置旋转动画 */
.discAnimation {
  /* infinite动画无限循环 */
  animation: disc 25s linear infinite;
  /* 动画延迟一秒 */
  animation-delay: 0.8s;
}

@keyframes disc {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.pause {
  animation-play-state: paused;
  -webkit-animation-play-state: paused;
}

.right {
  width: 350px;
}

.title {
  width: 100%;
  text-align: center;
  font-size: 12px;
  margin: 30px 0 15px;
  color: rgb(145, 145, 145);
}

.title div {
  margin: 7px 0;
}

.musicName {
  font-size: 23px;
  color: rgb(22, 22, 22);
}

.bottom {
  margin: 40px auto;
  width: 55vw;
}

.page {
  width: 100%;
  text-align: center;
  padding-bottom: 20px;
}

.tip {
  font-size: 20px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}
</style>
