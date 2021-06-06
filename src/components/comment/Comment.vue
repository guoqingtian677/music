<template>
  <div class="hotComments">
    <div class="commentHeader"><slot name="title"></slot></div>
    <div class="commentItem" v-for="(item, index) in comments" :key="index">
      <div class="commentCreatorAvatar">
        <img :src="item.user.avatarUrl + '?param=100y100'" alt="" />
      </div>
      <div class="commentMain">
        <div class="commentContent">
          <span class="commentUserNickName"
            >{{ item.user.nickname }}:&nbsp;</span
          >
          <span>{{ item.content }}</span>
        </div>
        <div class="replied">
          <div
            class="repliedItem"
            v-for="(item1, index1) in item.beReplied"
            :key="index1"
          >
            <span class="repliedUser">@{{ item1.user.nickname }}:&nbsp;</span>
            <span class="repliedContent">{{ item1.content }}</span>
          </div>
        </div>
        <div class="commentBottom">
          <div class="commentCreatedTime">
            {{ item.time | showDate }}
          </div>
          <div class="commentButtons">
            <div><i class="iconfont icon-good"></i> {{ item.likedCount }}</div>
            <div><i class="iconfont icon-zhuanfa"></i></div>
            <div><i class="iconfont icon-pinglun"></i></div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { formatDate } from "plugins/utils";

export default {
  name: "Comment",
  props: {
    comments: {
      type: Array,
      default() {
        return [];
      },
    },
  },
  filters: {
    showDate(value) {
      // 1、先将时间戳转成Date对象
      const date = new Date(value);

      // 2、将date进行格式化
      return formatDate(date, "yyyy-MM-dd");
    },
  },
};
</script>

<style scoped>
.hotComments {
  margin-bottom: 20px;
  width: 100%;
}

.commentHeader {
  font-size: 14px;
  color: rgb(26, 26, 26);
  font-weight: 600;
  padding: 10px 0;
}
.commentItem {
  display: flex;
  margin: 5px 0;
  padding: 10px 0 20px;
  border-bottom: 1px solid #eee;
  font-size: 12px;
}

.commentCreatorAvatar {
  width: 35px;
  height: 35px;
  margin: 0px 10px 0 0;
}

.commentCreatorAvatar img {
  width: 100%;
  border-radius: 50%;
}

.commentMain {
  width: calc(100% - 50px);
}

.commentUserNickName {
  color: #5a8ab9;
}

.commentBottom {
  margin-top: 2px;
  color: rgb(172, 172, 172);
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.commentButtons {
  display: flex;
  align-items: center;
}
.commentButtons div {
  margin-top: -3px;
  padding: 0 8px;
  transform: scale(0.85);
}

.commentButtons div:nth-child(1) {
  font-size: 14px;
}

.replied {
  background-color: #f4f4f4;
  margin: 7px 0;
  border-radius: 5px;
}

.repliedItem {
  padding: 7px 10px;
  line-height: 20px;
}

.repliedUser {
  color: #5a8ab9;
}

.page {
  width: 100%;
  text-align: center;
  padding-bottom: 20px;
}

.commentContent > span {
  line-height: 20px;
}
</style>