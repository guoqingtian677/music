<template>
  <div class="SearchSong">
    <div class="tip">找到 {{ songCount }} 首单曲</div>
    <div class="multiMatch" v-if="multiMatch.artist && currentPage == 1">
      <div class="matchTitle">最佳匹配</div>
      <div
        class="matchResult"
        @click="goToSingerDetail(multiMatch.artist[0].id)"
      >
        <div class="matchCover">
          <img
            :src="multiMatch.artist[0].img1v1Url + '?param==200y200'"
            alt=""
          />
        </div>
        <div class="matchName">歌手：{{ multiMatch.artist[0].name }}</div>
        <i class="iconfont icon-arrow-right-bold"></i>
      </div>
    </div>
    <!-- 表格 -->
    <!-- 第三方组件的table在退出后不会销毁，而是继续保留，导致多个table同时存在 ref变得不唯一 
              解决办法：避免使用refs..（拼接也不好使） -->
    <!-- 这里的播放效果样式不能直接简单的操作DOM，因为有分页，也可以做成无限滚动，或者根据索引渲染，但是我懒得搞了 -->
    <el-table
      :data="searchSongList"
      size="mini"
      style="width: 100%"
      @row-dblclick="clickRow"
      highlight-current-row
      stripe
    >
      <el-table-column label="" width="30" type="index" :index="handleIndex">
      </el-table-column>
      <el-table-column label="" width="55">
        <i class="iconfont icon-xihuan"></i>
        <i class="iconfont icon-download"></i>
      </el-table-column>
      <el-table-column prop="name" label="音乐标题" min-width="200">
      </el-table-column>
      <el-table-column prop="ar[0].name" label="歌手" min-width="100">
      </el-table-column>
      <el-table-column prop="al.name" label="专辑" min-width="170">
      </el-table-column>
      <el-table-column prop="dt" label="时长" min-width="60"> </el-table-column>
    </el-table>
    <!-- 分页 -->
    <div class="page" v-if="searchSongList.length != 0">
      <el-pagination
        background
        layout="prev, pager, next"
        :total="songCount"
        small
        :page-size="30"
        :current-page="currentPage"
        @current-change="pageChange"
      >
      </el-pagination>
    </div>
  </div>
</template>

<script>
import { handleMusicTime } from "plugins/utils";
export default {
  name: "SearchSong",
  data() {
    return {
      multiMatch: {},
      searchSongList: [],
      songCount: 0,
      currentPage: 1,
    };
  },
  methods: {
    // 请求
    // 获取最佳匹配
    async getMultMath() {
      let res = await this.$request("/search/multimatch", {
        keywords: this.$route.params.id,
      });
      this.multiMatch = res.data.result;
    },

    // 请求搜索单曲
    async getSearchSong() {
      let res = await this.$request("/cloudsearch", {
        keywords: this.$route.params.id,
        offset: 30 * (this.currentPage - 1),
      });
      console.log(res);
      this.songCount = res.data.result.songCount;
      this.searchSongList = res.data.result.songs;
      // 对数据的音乐时长进行处理
      this.searchSongList.forEach((item, index) => {
        this.searchSongList[index].dt = handleMusicTime(item.dt);
      });
    },

    // 事件响应
    // 处理索引
    handleIndex(index) {
      // console.log(index);
      index = index + 1 + 30 * (this.currentPage - 1);
      if (index < 10) {
        return "0" + index;
      } else {
        return index;
      }
    },
    // 双击table的row的回调
    async clickRow(row) {
      // 这里双击应该是要把当前双击的歌曲插入到当前的歌单中
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
    // 评论点击翻页的回调
    pageChange(page) {
      this.currentPage = page;
      this.searchSongList = [];
      this.getSearchSong();
    },
    // 点击最佳匹配的回调
    goToSingerDetail(id) {
      this.$router.push({ name: "singerDetail", params: { id } });
    },
  },
  created() {
    this.getMultMath();
    this.getSearchSong();
    // console.log(this.$route.params.id);
  },
  watch: {},
};
</script>

<style scoped>
.SearchSong {
  padding: 0 15px;
}

.tip {
  font-weight: bold;
  color: rgb(34, 34, 34);
  margin-bottom: 10px;
}

.matchTitle {
  font-size: 12px;
  color: rgb(34, 34, 34);
  margin: 5px 0;
}

.matchResult {
  display: flex;
  align-items: center;
  position: relative;
  border-radius: 5px;
  overflow: hidden;
  background-color: #f4f4f4;
  width: 270px;
  height: 65px;
  margin-bottom: 20px;
}

.matchCover {
  width: 65px;
  height: 65px;
}

.matchCover img {
  width: 100%;
  height: 100%;
}

.matchName {
  font-size: 13px;
  color: rgb(31, 31, 31);
  margin-left: 10px;
}

.multiMatch i {
  position: absolute;
  transform: translateY(-50%);
  right: 15px;
  top: 50%;
}

.page {
  width: 100%;
  text-align: center;
  padding-bottom: 20px;
  margin: 20px 0;
}
</style>