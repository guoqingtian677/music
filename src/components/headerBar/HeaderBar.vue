<template>
  <div class="headerBar">
    <div class="left">
      <img src="~assets/img/logo.png" alt="" />
    </div>
    <div class="center">
      <div class="buttons">
        <i class="iconfont icon-arrow-left-bold back" @click="goBack"></i>
        <i
          class="iconfont icon-arrow-right-bold forward"
          @click="goForward"
        ></i>
      </div>
      <div class="search">
        <el-popover
          placement="bottom"
          width="300"
          v-model="isSearchPopShow"
          popper-class="searchPop"
          trigger="focus"
        >
          <el-input
            placeholder="请输入内容"
            prefix-icon="el-icon-search"
            size="mini"
            slot="reference"
            @input="inputSearch"
            @keyup.enter.native="onSubmit"
            v-model="searchInput"
          >
          </el-input>
          <!-- 搜索历史不想做了，做的话可以将搜索历史存到localstorage中 -->
          <!-- 热搜榜 -->
          <div class="hotSearch" v-if="!searchSuggestList.songs">
            <div class="hotSearchTitle">热搜榜</div>
            <div
              class="hotSearchItem"
              v-for="(item, index) in hotSearchList"
              :key="index"
              @click="clickHotSearchItem(item.searchWord)"
            >
              <div class="hotSearchIndex" :class="index < 3 ? 'topThree' : ''">
                {{ index + 1 }}
              </div>
              <div class="hotSearchInfo">
                <div
                  class="hotSearchWord"
                  :class="index < 3 ? 'hotSearchWordTopThree' : ''"
                >
                  {{ item.searchWord }}
                </div>
                <div class="hotSearchContent">{{ item.content }}</div>
              </div>
            </div>
          </div>
          <!-- 搜索建议 -->
          <div class="searchSuggest">
            <div class="hotSearchTitle">搜索建议</div>
            <!-- 单曲 -->
            <div
              class="searchSuggestItem"
              v-if="
                searchSuggestList.songs && searchSuggestList.songs.length != 0
              "
            >
              <div class="searchSuggestItemTitle">
                <i class="iconfont icon-yinle"></i> 单曲
              </div>
              <div
                class="suggestItemDetail"
                v-for="(item, index) in searchSuggestList.songs"
                :key="index"
                @click="clickSuggestSong(item.id)"
              >
                {{ item.name + " - " + item.artists[0].name }}
              </div>
            </div>
            <!-- 歌手 -->
            <div
              class="searchSuggestItem"
              v-if="
                searchSuggestList.artists &&
                searchSuggestList.artists.length != 0
              "
            >
              <div class="searchSuggestItemTitle">
                <i class="iconfont icon-mic"></i> 歌手
              </div>
              <div
                class="suggestItemDetail"
                v-for="(item, index) in searchSuggestList.artists"
                :key="index"
                @click="clickSuggestSinger(item.id)"
              >
                {{ item.name }}
              </div>
            </div>
            <!-- 专辑 -->
            <div
              class="searchSuggestItem"
              v-if="
                searchSuggestList.albums && searchSuggestList.albums.length != 0
              "
            >
              <div class="searchSuggestItemTitle">
                <i class="iconfont icon-more"></i> 专辑
              </div>
              <div
                class="suggestItemDetail"
                v-for="(item, index) in searchSuggestList.albums"
                :key="index"
                @click="clickSuggestAlbum(item.id)"
              >
                {{ item.name + " - " + item.artist.name }}
              </div>
            </div>
            <!-- 歌单 -->
            <div
              class="searchSuggestItem"
              v-if="
                searchSuggestList.playlists &&
                searchSuggestList.playlists.length != 0
              "
            >
              <div class="searchSuggestItemTitle">
                <i class="iconfont icon-gedan"></i> 歌单
              </div>
              <div
                class="suggestItemDetail"
                v-for="(item, index) in searchSuggestList.playlists"
                :key="index"
                @click="clickSuggestMusicList(item.id)"
              >
                {{ item.name }}
              </div>
            </div>
          </div>
          <!-- <img src="~assets/img/searh.png" alt="" /> -->
        </el-popover>
      </div>
    </div>
    <div class="right">
      <div class="user">
        <div class="avatar">
          <!-- 登录框 -->
          <el-popover
            placement="bottom"
            width="330"
            trigger="manual"
            v-if="!userInfo.avatarUrl"
            :value="isPopoverShow"
          >
            <!-- <input type="text" class="phoneNum" />
            <input type="password" class="password" /> -->
            <el-form
              ref="form"
              v-model="loginForm"
              label-width="80px"
              label-position="right"
              size="mini"
            >
              <el-form-item
                label="手机号码："
                size="mini"
                label-width="100px"
                required
              >
                <input type="text" v-model="loginForm.phoneNum" />
              </el-form-item>
              <el-form-item
                label="密码："
                size="mini"
                label-width="100px"
                required
              >
                <input type="password" v-model="loginForm.password" />
              </el-form-item>
              <div class="loginButton">
                <el-button type="danger" @click="login" size="mini"
                  >登录</el-button
                >
                <el-button size="mini" @click="isPopoverShow = !isPopoverShow"
                  >取消</el-button
                >
              </div>
            </el-form>
            <!-- <el-button slot="reference">click 激活</el-button> -->
            <img
              src="~assets/img/test.jpg"
              alt=""
              slot="reference"
              @click="isPopoverShow = !isPopoverShow"
            />
          </el-popover>
          <!-- 已登录状态 -->
          <el-popover
            placement="bottom"
            width="150"
            trigger="click"
            v-else
            v-model="isPopoverShow"
          >
            <img :src="userInfo.avatarUrl" alt="" slot="reference" />
            <div class="loginButton">
              <el-button size="mini" type="info" class="logout" @click="logout"
                >退出登录</el-button
              >
            </div>
          </el-popover>
        </div>
        <div class="userName" v-if="userInfo.nickname">
          {{ userInfo.nickname }}
        </div>
        <div class="userName" v-else>点击头像登录</div>
      </div>
    </div>
  </div>
</template>

<script>
import { handleMusicTime } from "plugins/utils";
// 节流定时器名称
let timer;

export default {
  name: "HeaderBar",
  data() {
    return {
      loginForm: {
        password: "",
        phoneNum: "",
      },
      userInfo: {},
      isPopoverShow: false,
      // 是否显示searchPop
      isSearchPopShow: false,
      // 热搜列表数据
      hotSearchList: [],
      // 需要搜索的内容
      searchInput: "",
      // 搜索建议列表
      searchSuggestList: {},
    };
  },
  methods: {
    //   请求
    // 手机前端验证正则表达式
    // let phoneReg = /^1(3|4|5|6|7|8|9)\d{9}$/;

    //   手机登录请求
    async loginByCellphone() {
      let result = await this.$request("/login/cellphone", {
        phone: this.loginForm.phoneNum,
        password: this.loginForm.password,
        withCredentials: true,
      });
      // console.log(result);
      // 登录成功
      if (result.data.code == 200) {
        // 将请求到的用户信息存入localstorage
        window.localStorage.setItem(
          "userInfo",
          JSON.stringify(result.data.profile)
        );
        this.userInfo = result.data.profile;
        this.isPopoverShow = false;
        this.$message.success("登录成功!");
        // 刷新页面
        this.$router.go(0);
      } else if (result.data.code == 400) {
        // 手机号错误
        this.$message.error("手机号错误!");
      } else if (result.data.code == 502) {
        // 密码错误
        this.$message.error("密码错误!");
      } else {
        // 登录失败，请稍后重试
        this.$message.error("登录失败，请稍后重试!");
      }

      // 清空输入框的内容
      this.loginForm.password = "";
      this.loginForm.phoneNum = "";
    },
    // 获取热搜列表
    async getHotSearch() {
      let res = await this.$request("/search/hot/detail");
      // console.log(res);
      this.hotSearchList = res.data.data;
    },
    // 获取搜索建议
    async getSearchSuggest(keywords) {
      let res = await this.$request("/search/suggest", { keywords });
      console.log(res);
      this.searchSuggestList = res.data.result;
    },
    // 根据id获取歌曲详情
    async getMusicInfo(ids) {
      let res = await this.$request("/song/detail", { ids });
      console.log(res);
      // 处理时间
      res.data.songs[0].dt = handleMusicTime(res.data.songs[0].dt);
      return res.data.songs[0];
    },

    // 点击登录按钮的回调
    login() {
      // console.log("提交表单");
      //   console.log(this.loginForm);
      this.loginByCellphone();
    },

    // 点击退出登录的回调
    logout() {
      // 清空data和localstorage中的数据，以及cookie
      this.userInfo = {};
      window.localStorage.setItem("userInfo", "");
      this.clearAllCookie();
      //   在vuex中更新登录状态
      this.$store.commit("updataLoginState");
      this.isPopoverShow = false;
      this.$message.success("退出成功!");
      // 刷新页面
      this.$router.go(0);
    },

    //清除所有cookie函数
    clearAllCookie() {
      var date = new Date();
      date.setTime(date.getTime() - 10000);
      var keys = document.cookie.match(/[^ =;]+(?=\=)/g);
      console.log("需要删除的cookie名字：" + keys);
      if (keys) {
        for (var i = keys.length; i--; )
          document.cookie =
            keys[i] + "=0; expire=" + date.toGMTString() + "; path=/";
      }
    },

    // 点击前进按钮的回调
    goForward() {
      this.$router.go(1);
    },
    // 点击后退按钮的回调
    goBack() {
      this.$router.go(-1);
    },

    // 在搜索框中输入的回调
    inputSearch(e) {
      // console.log(e);
      clearTimeout(timer);
      if (e != "") {
        timer = setTimeout(() => {
          this.getSearchSuggest(e);
        }, 500);
      } else {
        // 清空searchSuggestList
        this.searchSuggestList = {};
        return;
      }
    },
    // 在输入框按下回车的回调
    onSubmit(e) {
      if (e.keyCode == 13 && this.searchInput != "") {
        this.goSearch();
      }
    },
    // 跳转至搜索详情页面
    goSearch() {
      this.isSearchPopShow = false;
      this.$router.push({ name: "search", params: { id: this.searchInput } });
    },
    // 点击热搜榜item的回调
    clickHotSearchItem(searchWord) {
      this.searchInput = searchWord;
      this.goSearch();
    },
    // 点击搜索建议中专辑的回调
    clickSuggestAlbum(id) {
      this.$router.push({ name: "album", params: { id } });
      this.isSearchPopShow = false;
    },
    // 点击搜索建议中歌手的回调
    clickSuggestSinger(id) {
      this.$router.push({ name: "singerDetail", params: { id } });
      this.isSearchPopShow = false;
    },
    // 点击搜索建议中歌单的回调
    clickSuggestMusicList(id) {
      this.$router.push({ name: "musicListDetail", params: { id } });
      this.isSearchPopShow = false;
    },
    // 点击搜索建议中单曲的回调
    async clickSuggestSong(id) {
      let row = await this.getMusicInfo(id);
      this.isSearchPopShow = false;
      // 首先获取当前的歌单列表和歌曲索引
      let musicList = this.$store.state.musicList;
      let currentIndex = this.$store.state.currentIndex;
      // 先判断该歌曲是否已经在歌单中存在，避免重复点击导致歌单出现相同歌曲
      let isExist = musicList.find((item) => item.id == row.id);
      if (isExist) {
        // 如果已经存在 则只提交歌曲id并return出去
        this.$store.commit("updateMusicId", row.id);
        return;
      }
      this.$store.commit("changePlayState", false);
      // 将请求到的歌曲详情插入到歌单对应位置并提交至vuex
      musicList.splice(currentIndex + 1, 0, row);
      this.$store.commit("updateMusicId", row.id);
      this.$store.commit("updateMusicList", {
        musicList,
        musicListId: this.$store.state.musicListId,
      });
    },
  },
  created() {
    this.getHotSearch();
    // 从localstorage中读取userInfo
    if (window.localStorage.getItem("userInfo")) {
      // 进入这里表示已经登录，更新登录状态到vuex
      this.$store.commit("updataLoginState");
      // 从localStorage中取userinfo
      this.userInfo = JSON.parse(window.localStorage.getItem("userInfo"));
    }
  },
};
</script>

<style scoped>
@import "./HeaderBar-element.css";
.headerBar {
  display: flex;
  align-items: center;
  width: 100%;
  height: 50px;
  position: relative;
}

.left {
  width: 110px;
  margin: 0 50px 0 10px;
}

.left img {
  width: 100%;
}

.center {
  display: flex;
  align-items: center;
}

.buttons {
  color: rgb(235, 235, 235);
  /* display: flex; */
  height: 22px;
}

.buttons i {
  background-color: #e13e3e;
  font-size: 12px;
  transform: scale(0.8);
  padding: 5px;
  height: 22px;
  width: 22px;
  border-radius: 50%;
  margin: 0 3px;
}

.right {
  display: flex;
  position: absolute;
  right: 50px;
  top: 0;
  line-height: 50px;
}

.user {
  display: flex;
  align-items: center;
}

.avatar {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  overflow: hidden;
  margin-right: 7px;
  cursor: pointer;
}

.avatar img {
  width: 100%;
}

.userName {
  font-size: 12px;
  color: rgba(255, 255, 255, 0.85);
  width: 100px;
}

.loginButton {
  width: 100%;
  text-align: center;
}

.hotSearchTitle {
  font-size: 13px;
  margin: 10px 0 5px 20px;
}

.hotSearchItem {
  display: flex;
  align-items: center;
  padding: 6.5px 15px;
}

.hotSearchItem:hover {
  background-color: #f2f2f2;
}

.hotSearchIndex {
  margin-right: 15px;
  color: #aaa;
}

.topThree {
  color: #e13e3e;
}

.hotSearchWord {
  font-size: 12px;
  color: rgb(51, 51, 51);
}

.hotSearchWordTopThree {
  font-weight: 600;
  color: black;
}

.hotSearchContent {
  font-size: 12px;
  transform: scale(0.9) translateX(-5%);
}

.searchSuggest {
  font-size: 12px;
}

.searchSuggestItemTitle {
  background-color: #f5f5f7;
  padding: 4px 8px;
}

.searchSuggestItemTitle i {
  font-size: 15px;
}

.suggestItemDetail {
  padding: 4px 27px;
  font-size: 12px;
}

.suggestItemDetail:hover {
  background-color: #f2f2f2;
}
</style>