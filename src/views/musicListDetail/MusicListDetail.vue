<template>
  <div class="musicListDetail" v-if="musicListDetail">
    <!-- 歌单信息 -->
    <div class="listInfo">
      <!-- 歌单封面 -->
      <div class="listAvatar">
        <img :src="musicListDetail.coverImgUrl" alt="" />
      </div>
      <div class="right">
        <!-- 标题 -->
        <div class="title">
          <div class="titleTag">歌单</div>
          <div class="titleContent">{{ musicListDetail.name }}</div>
        </div>
        <!-- 用户信息 -->
        <div class="user">
          <div class="userAvatar">
            <img :src="musicListDetail.creator.avatarUrl" alt="" />
          </div>
          <div class="userName">{{ musicListDetail.creator.nickname }}</div>
          <div class="createTime">
            {{ musicListDetail.createTime | showDate }}创建
          </div>
        </div>
        <!-- 操作按钮 -->
        <div class="buttons">
          <div class="buttonItem playAll" @click="playAll">
            <i class="iconfont icon-bofang playAll"></i>
            <span>播放全部</span>
          </div>
          <div class="buttonItem" v-if="!isCreated" @click="collectList">
            <i class="iconfont icon-xihuan" :class="isSub ? 'red' : ''"></i>
            <span>{{ isSub ? "已收藏" : "收藏" }}</span>
          </div>
          <div class="buttonItem">
            <i class="iconfont icon-zhuanfa"></i>
            <span>分享</span>
          </div>
        </div>
        <!-- 标签 -->
        <div class="tags">
          标签：
          <div
            class="tagItem"
            v-for="(item, index) in musicListDetail.tags"
            :key="index"
          >
            {{ item }}
          </div>
        </div>
        <!-- 歌曲列表的歌曲数量和播放量 -->
        <div class="otherInfo">
          <div class="musicNum">
            歌曲 : {{ musicListDetail.trackCount | handleNum }}
          </div>
          <div class="playCount">
            播放 : {{ musicListDetail.playCount | handleNum }}
          </div>
        </div>
        <div class="desc">简介 : {{ musicListDetail.description }}</div>
      </div>
    </div>
    <!-- 歌曲列表 -->
    <div class="musicList">
      <el-tabs value="first" @tab-click="clickTab">
        <el-tab-pane label="歌曲列表" name="first">
          <!-- 表格 -->
          <el-table
            :data="musicListDetail.tracks"
            size="mini"
            style="width: 100%"
            @row-dblclick="clickRow"
            highlight-current-row
            stripe
            lazy
          >
            <el-table-column
              label=""
              width="30"
              type="index"
              :index="handleIndex"
            >
            </el-table-column>
            <el-table-column label="" width="55">
              <i class="iconfont icon-xihuan"></i>
              <i class="iconfont icon-download"></i>
            </el-table-column>
            <el-table-column prop="name" label="音乐标题" min-width="350">
            </el-table-column>
            <el-table-column prop="ar[0].name" label="歌手" min-width="120">
            </el-table-column>
            <el-table-column prop="al.name" label="专辑" min-width="170">
            </el-table-column>
            <el-table-column prop="dt" label="时长" min-width="100">
            </el-table-column>
            <!-- <el-table-column prop="id"></el-table-column> -->
          </el-table>
          <div class="loadMore" v-if="isMore" @click="loadMore">
            点击加载所有音乐
          </div>
          <div class="placeholder" v-else></div>
        </el-tab-pane>
        <el-tab-pane label="评论" name="second">
          <div class="commentList" v-if="comments.comments">
            <!-- 精彩评论 -->
            <comment
              :comments="comments.hotComments"
              v-if="comments.hotComments"
              ><div slot="title">精彩评论</div></comment
            >
            <!-- 最新评论 -->
            <comment :comments="comments.comments"
              ><div slot="title">热门评论</div></comment
            >
          </div>
          <!-- 分页 -->
          <div
            class="page"
            v-show="comments.comments && comments.comments.length != 0"
          >
            <el-pagination
              background
              layout="prev, pager, next"
              :total="comments.total"
              small
              :page-size="50"
              :current-page="currentCommentPage"
              @current-change="commentPageChange"
            >
            </el-pagination>
          </div>
        </el-tab-pane>
        <el-tab-pane label="收藏者" name="third">不想做了</el-tab-pane>
      </el-tabs>
    </div>
  </div>
</template>

<script>
import { formatDate, handleNum, handleMusicTime } from "plugins/utils";
import Comment from "components/comment/Comment";

export default {
  name: "MusicListDetail",
  data() {
    return {
      musicListDetail: null,
      comments: {},
      // 当前评论页数
      currentCommentPage: 1,
      // 是否还有更多音乐
      isMore: false,
      // 用户是否收藏了当前歌单
      isSub: false,
      // 是否是用户创建的歌单
      isCreated: false,
    };
  },
  components: {
    Comment,
  },
  methods: {
    // 请求
    // 根据传来的 id 查询歌单
    async getMusicListDetail() {
      var timestamp = Date.parse(new Date());
      // console.log(this.$route.params.id);
      let result = await this.$request("/playlist/detail", {
        id: this.$route.params.id,
        timestamp,
      });
      // console.log(result);
      this.musicListDetail = result.data.playlist;
      // console.log(this.musicListDetail);
      // 判断是否还有更多音乐
      if (
        this.musicListDetail.tracks.length !=
        this.musicListDetail.trackIds.length
      ) {
        this.isMore = true;
      }
      // 处理播放时间
      this.musicListDetail.tracks.forEach((item, index) => {
        this.musicListDetail.tracks[index].dt = handleMusicTime(item.dt);
      });
      // 判断用户是否喜欢该音乐
      // 直接两个循环性能损耗太厉害了 没什么思路暂时不做先
      // let likeMusicList = this.$store.state.likeMusicList;
    },
    // 获取歌单评论
    async getMusicListComment() {
      // 每次请求前先清空comment
      this.comments.comments = [];
      let res = await this.$request("/comment/playlist", {
        id: this.$route.params.id,
        offset: (this.currentCommentPage - 1) * 50,
        limit: 50,
      });
      // console.log(res);
      this.comments = res.data;
    },
    // 获取歌曲详情
    async getMusicDetail(ids) {
      let res = await this.$request("/song/detail", { ids });
      // 处理时间
      res.data.songs.forEach((item, index) => {
        res.data.songs[index].dt = handleMusicTime(item.dt);
      });
      this.musicListDetail.tracks.push(...res.data.songs);
    },
    // 请求用户歌单
    async getUserMusicList() {
      let timestamp = Date.parse(new Date());
      // 先从localStorage里面读取用户信息  如果登录成功是有存的
      this.userInfo =
        window.localStorage.getItem("userInfo") &&
        JSON.parse(window.localStorage.getItem("userInfo"));
      let res = await this.$request("/user/playlist", {
        uid: this.userInfo.userId,
        timestamp,
      });
      res = res.data.playlist;
      let index = res.findIndex((item) => item.subscribed == true);
      this.collectedMusicList = res.slice(index);
      // 将收藏的歌单上传至vuex
      this.$store.commit("updateCollectMusicList", this.collectedMusicList);
    },

    // 事件函数
    handleIndex(index) {
      // console.log(index);
      index += 1;
      if (index < 10) {
        return "0" + index;
      } else {
        return index;
      }
    },
    // 双击table的row的回调
    async clickRow(row) {
      // console.log(row);
      // 将musicId提交到vuex中 供bottomControl查询歌曲url和其它操作
      this.$store.commit("updateMusicId", row.id);
      // 如果歌单发生变化,则提交歌单到vuex
      if (this.musicListDetail.id != this.$store.state.musicListId) {
        // 将歌单传到vuex
        this.$store.commit("updateMusicList", {
          musicList: this.musicListDetail.tracks,
          musicListId: this.musicListDetail.id,
        });
      }

      // let result = await this.$request("/song/url", { id: row.id, br: 320000 });
      // console.log(result.data.data[0].url);
      // this.$store.commit("updateMusicUrl", result.data.data[0].url);
    },
    // 点击播放全部按钮的回调
    playAll() {
      this.$store.commit("updateMusicId", this.musicListDetail.tracks[0].id);
      this.$store.commit("updateMusicList", {
        musicList: this.musicListDetail.tracks,
        musicListId: this.musicListDetail.id,
      });
    },
    // 评论点击翻页的回调
    commentPageChange(page) {
      this.currentCommentPage = page;
      this.getMusicListComment();
    },
    // handleTableDOM(currentIndex, lastIndex) {
    //   // 将列表渲染到视图层还需要一定时间 $nextTick也不知道为什么不好使,可能是因为是第三方组件的缘故,暂时先用定时器解决
    //   // 前面先选择musicListDetail，避免选错 选到别的地方的表单
    //   // 解决了一个选择报错，原因不是很清楚 应该是修改第三方组件DOM引起的
    //   if (document.querySelector(".musicListDetail")) {
    //     let tableRows = document
    //       .querySelector(".musicListDetail")
    //       .querySelectorAll(".el-table__row");
    //     if (currentIndex != -1) {
    //       // console.log(tableRows[currentIndex].firstChild);
    //       // 将正在播放的音乐前面的索引换成小喇叭
    //       tableRows[currentIndex].children[0].querySelector(
    //         ".cell"
    //       ).innerHTML = `<div><i class="iconfont icon-yinliang"></i></div>`;

    //       // 直接修改dom样式的颜色无效  可能是因为第三方组件的原故
    //       // 通过引入全局样式解决
    //       tableRows[currentIndex].children[0]
    //         .querySelector(".iconfont")
    //         .classList.add("currentRow");
    //       tableRows[currentIndex].children[2]
    //         .querySelector(".cell")
    //         .classList.add("currentRow");
    //     }
    //     if ((lastIndex && lastIndex != -1) || lastIndex == 0) {
    //       // 将上一个播放的dom的小喇叭换回索引
    //       tableRows[lastIndex].children[0].querySelector(
    //         ".cell"
    //       ).innerHTML = `<div>${
    //         lastIndex + 1 < 10 ? "0" + (lastIndex + 1) : lastIndex + 1
    //       }</div>`;

    //       // 将上一首的类名删掉  小喇叭的html已经被替换了，不需要再还原
    //       tableRows[lastIndex].children[2]
    //         .querySelector(".cell")
    //         .classList.remove("currentRow");
    //     }
    //   }
    // },

    handleDOM(current, last) {
      if (document.querySelector(".musicListDetail")) {
        let tableRows = document
          .querySelector(".musicListDetail")
          .querySelectorAll(".el-table__row");
        // 遍历当前musicList 找到当前播放的index的行进行渲染
        // console.log(tableRows);
        let index = this.musicListDetail.tracks.findIndex(
          (item) => item.id == current
        );
        // console.log(index);
        if (index != -1) {
          // 直接修改dom样式的颜色无效  可能是因为第三方组件的原故
          // 通过引入全局样式解决
          // 将正在播放的音乐前面的索引换成小喇叭
          tableRows[index].children[0].querySelector(
            ".cell"
          ).innerHTML = `<div><i class="iconfont icon-yinliang"></i></div>`;
          tableRows[index].children[0]
            .querySelector(".iconfont")
            .classList.add("currentRow");
          tableRows[index].children[2]
            .querySelector(".cell")
            .classList.add("currentRow");
        }
        // 清除上一首的样式
        if (last != -1) {
          let lastIndex = this.musicListDetail.tracks.findIndex(
            (item) => item.id == last
          );
          if (lastIndex != -1) {
            // 将上一个播放的dom的小喇叭换回索引
            tableRows[lastIndex].children[0].querySelector(
              ".cell"
            ).innerHTML = `<div>${
              lastIndex + 1 < 10 ? "0" + (lastIndex + 1) : lastIndex + 1
            }</div>`;

            // 将上一首的类名删掉  小喇叭的html已经被替换了，不需要再还原
            tableRows[lastIndex].children[2]
              .querySelector(".cell")
              .classList.remove("currentRow");
          }
        }
      }
    },
    // 点击el-tab-pane的回调
    clickTab(e) {
      console.log(e.index);
      if (e.index == 1 && !this.comments.comments) {
        this.getMusicListComment();
      }
    },
    // 点击加载所有音乐的回调
    loadMore() {
      if (!this.$store.state.isLogin) {
        this.$message.error("请先进行登录操作!");
        return;
      }
      console.log("加载所有音乐");
      this.isMore = false;
      let arr = this.musicListDetail.trackIds.slice(20);
      let ids = "";
      arr.forEach((item) => {
        ids += item.id + ",";
      });
      ids = ids.substr(0, ids.length - 1);
      console.log(ids);
      this.getMusicDetail(ids);
    },
    // 判断用户是否收藏了该歌单
    getIsSub() {
      this.isSub = this.$store.state.collectMusicList.find(
        (item) => item.id == this.$route.params.id
      );
    },
    // 判断是否是用户创建的歌单
    getIsCreated() {
      this.isCreated = this.$store.state.createdMusicList.find(
        (item) => item.id == this.$route.params.id
      );
    },
    // 点击收藏按钮的回调
    async collectList() {
      if (!this.$store.state.isLogin) {
        this.$message.error("请先进行登录操作!");
        return;
      }
      this.isSub = !this.isSub;
      // 请求
      let timestamp = Date.parse(new Date());
      await this.$request("/playlist/subscribe", {
        id: this.$route.params.id,
        t: this.isSub ? 1 : 2,
        timestamp,
      });
      // 请求收藏歌单列表并保存至vuex
      this.getUserMusicList();
    },
  },
  computed: {},
  watch: {
    // "$store.state.currentIndex"(currentIndex, lastIndex) {
    //   // 目前没什么好思路 直接操作原生DOM
    //   console.log(currentIndex, lastIndex);
    //   // this.handleTableDOM(currentIndex, lastIndex);
    // },
    "$store.state.musicId"(current, last) {
      this.handleDOM(current, last);
    },
    // 监听createdMusicList的变化
    "$store.state.createdMusicList"(current, last) {
      // 如果在收藏页面刷新，收藏歌单还没获取到，但是收藏按钮已经渲染了，所以在第一次获取到数据时，再渲染一次
      // 如果是刚刷新，last则为空
      if (last.length == 0) {
        this.getIsSub();
      }
    },
  },
  filters: {
    showDate(value) {
      // 1、先将时间戳转成Date对象
      const date = new Date(value);

      // 2、将date进行格式化
      return formatDate(date, "yyyy-MM-dd");
    },
    handleNum,
  },
  created() {},
  async mounted() {
    if (this.$store.state.isLogin) {
      // 先判断是否是用户创建的歌单
      this.getIsCreated();
      // 如果不是 再判断是否是收藏的歌单
      if (!this.isCreated) {
        this.getIsSub();
      }
    }
    await this.getMusicListDetail();
    // 判断是否和上一次打开的歌单相同
    if (this.$route.params.id == this.$store.state.musicListId) {
      this.handleDOM(this.$store.state.musicId);
    }
  },
};
</script>

<style scoped>
.listInfo {
  display: flex;
  padding: 25px 15px;
  align-items: center;
}

.listAvatar {
  width: 150px;
  height: 150px;
  overflow: hidden;
  border-radius: 10px;
  margin-right: 15px;
}

.listAvatar img {
  width: 100%;
}

.right {
  width: calc(100% - 200px);
}

.title {
  display: flex;
  align-items: center;
}

.titleTag {
  font-size: 12px;
  color: #ec4141;
  border: 1px solid #ec4141;
  padding: 1px 2px;
  border-radius: 2px;
  margin-right: 5px;
  transform: scale(0.8);
}

.titleContent {
  font-size: 20px;
  font-weight: 600;
  color: #373737;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  width: 90%;
}

.user {
  display: flex;
  align-items: center;
  margin-top: 8px;
  font-size: 12px;
}

.userAvatar {
  height: 25px;
  width: 25px;
  margin-right: 8px;
}

.userAvatar img {
  width: 100%;
  border-radius: 50%;
}

.userName {
  color: #6191c2;
  margin-right: 8px;
}

.createTime {
  transform: scale(0.9);
}

.buttons {
  margin: 8px 0 0 -5px;
  display: flex;
}

.buttonItem {
  font-size: 12px;
  padding: 8px 15px;
  border: 1px solid #ddd;
  border-radius: 20px;
  transform: scale(0.9);
}

.buttonItem i {
  font-size: 12px;
  margin-right: 3px;
  transform: scale(0.9);
}

.playAll {
  background-color: #ec4141;
  color: white;
}

.tags {
  margin: 8px 0 0 -30px;
  display: flex;
  font-size: 12px;
  transform: scale(0.9);
}

.tagItem {
  color: #6191c2;
  margin-right: 5px;
}

.otherInfo {
  margin: 5px 0 0 -30px;
  display: flex;
  font-size: 12px;
  transform: scale(0.9);
}

.musicNum {
  margin-right: 13px;
}

.desc {
  margin: 5px 0 0 -30px;
  font-size: 12px;
  transform: scale(0.9);
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.musicList {
  margin: -15px 15px 0;
}

.page {
  width: 100%;
  text-align: center;
  padding-bottom: 20px;
}

.placeholder {
  width: 100%;
  height: 50px;
}

.loadMore {
  width: 100%;
  height: 50px;
  font-size: 12px;
  color: #aaa;
  text-align: center;
  line-height: 50px;
}

.red {
  color: #ec4141;
}
</style>