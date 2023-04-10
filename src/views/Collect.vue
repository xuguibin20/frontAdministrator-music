
</script>
<style lang="less" scoped></style>
<template>
  <div class="test">

    <div class="container">
      <div class="handle-box">
        <el-button type="primary" size="mini" @click="delAll"
          >批量删除</el-button
        >

        <el-input
          v-model="select_word"
          size="mini"
          placeholder="请输入歌曲名"
          class="handle-input"
        ></el-input>
      
      </div>
    </div>
    <el-table
      size="mini"
      border
      style="width: 100%"
      height="666px"
      :data="data"
      @selection-change="handleSelectionChange"
    >
      <el-table-column type="selection" width="40px"></el-table-column>
      <el-table-column label="歌曲图片" width="110" align="center">
        <template slot-scope="scope">
          <div class="song-img">
            <img
              :src="getUrl(scope.row.pic)"
              style="width: 100%; aspect-ratio: 1 / 1"
            />
            <div class="mask" style="width: 100%; aspect-ratio: 1 / 1"></div>
          </div>

          <div class="play">
            <div
              v-if="toggle == scope.row.id"
              class="icon-total"
              @click="getSongUrl(scope.row.url, scope.row.id)"
            >
              <svg class="icon"><use xlink:href="#icon-zanting"></use></svg>
            </div>
            <div
              v-if="toggle != scope.row.id"
              class="icon-total"
              @click="getSongUrl(scope.row.url, scope.row.id)"
            >
              <svg class="icon"><use xlink:href="#icon-bofanganniu"></use></svg>
            </div>
          </div>
        </template>
      </el-table-column>
      <el-table-column
        prop="singerName"
        label="歌手"
        width="120"
        align="center"
      >
      </el-table-column>
      <el-table-column prop="songName" label="歌名" width="120" align="center">
      </el-table-column>
      <el-table-column
        prop="introduction"
        label="专辑"
        width="150"
        align="center"
      >
      </el-table-column>
      <el-table-column label="歌词" align="center">
        <template slot-scope="scope">
          <ul style="height: 100px; overflow: scroll">
            <li
              v-for="(item, index) in parseLyric(scope.row.lyric)"
              :key="index"
            >
              {{ item }}
            </li>
          </ul>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="170px" align="center">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="danger"
            @click="handleDelete(scope.row.id)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <div class="pagination">
      <el-pagination
        background
        layout="total,prev,pager,next"
        :current-page="currentPage"
        :page-size="pageSize"
        :total="tableData.length"
        @current-change="handleCurrentChange"
      ></el-pagination>
    </div>

  
  </div>
</template>

<script>
import { mixin } from "@/mixins/index";
import { mapGetters } from "vuex";
import "@/assets/js/iconfont.js";
import { getCollectOfUserId, delCollection } from "@/api/index";
export default {
  mixins: [mixin],
  name: "Collect",

  data() {
    return {
      singerName: "", //歌手名
      delVisible: false, //删除弹窗是否显示

      tableData: [],
      tempData: [],
      select_word: "",
      pageSize: 5, //分页每页大小
      currentPage: 1, //当前页
      idx: -1, //当前选择项目
      multipleSelection: [], //哪些项已经打勾了
      toggle: false, //播放器显示图标的状态
    };
  },
  watch: {
    select_word() {
      //搜索框里面的内容发生变化的时候，搜索结果table列表的内容跟着它的内容发生变化
      if (this.select_word == "") {
        this.tableData = this.tempData;
      } else {
        this.tableData = [];
        for (let item of this.tempData) {
          if (item.name.includes(this.select_word)) {
            this.tableData.push(item);
          }
        }
      }
    },
    isPlay() {
      if (this.isPlay) {
        this.$store.commit("setplayButtonUrl", "#icon-zanting");
      } else {
        this.$store.commit("setplayButtonUrl", "#icon-bofang");
      }
    },
  },

  computed: {
    ...mapGetters([
      "isPlay", //getters里的
      "id", //歌曲id
      "singerId", //歌手id
      "songId", //歌曲id
    ]),
    //计算当前搜索结果表里的数据
    data() {
      return this.tableData.slice(
        (this.currentPage - 1) * this.pageSize,
        this.currentPage * this.pageSize
      );
    },
  },

  created() {
    this.getData();
  },
  destroyed() {
    this.$store.commit("setisPlay", false);
  },
  methods: {
    handleClose(done) {
      this.$confirm("确认关闭？")
        .then((_) => {
          done();
        })
        .catch((_) => {});
    },
    //获取当前页
    handleCurrentChange(val) {
      this.currentPage = val;
    },
    //根据用户id查询所有收藏的歌曲
    getData() {
      this.tempData = [];
      this.tableData = [];
      getCollectOfUserId(this.$route.query.id).then((res) => {
        this.tableData = res;
        this.tempData = res;
        this.currentPage = 1;
      });
    },

    //解析歌词
    parseLyric(text) {
      let lines = text.split("\n");
      let pattern = /\[\d{2}:\d{2}.(\d{3}|\d{2})\]/g;
      let result = [];
      if (text == "") {
        result = "暂无歌词";
      } else {
        for (let item of lines) {
          let value = item.replace(pattern, "");
          result.push(value);
        }
      }
      return result;
    },
    //删除一首歌
    handleDelete(id) {
      let params = new URLSearchParams();
      params.append("userId", this.$route.query.id);
      params.append("songId", id);

      delCollection(params)
        .then((res) => {
          if (res.code == 1) {
            this.getData();

            this.notify("删除成功", "success");
          } else {
            this.notify("删除失败", "error");
          }
        })
        .catch((err) => {
          console.log(err);
        });
    },
    //切换播放歌曲
    getSongUrl(url, id) {
      this.$store.commit("setUrl", this.$store.state.HOST + "/" + url);
      if (id == this.$store.state.id) {
        if (this.isPlay) {
          this.$store.commit("setisPlay", false);
          this.toggle = " id";
        } else {
          this.$store.commit("setisPlay", true);
          this.toggle = id;
        }
      } else {
        this.$store.commit("setId", id);
        this.$store.commit("setisPlay", true);
        this.toggle = id;
      }
      // this.$store.commit("setId", id);
    },
  },
};
</script>
<style scoped>
.table {
  min-width: 800px;
}

.container {
  padding: 20px;
  background-color: #fff;
  border: 1px solid #ddd;
  border-radius: 5px;
}

.song-img {
  height: 80px;
  border-radius: 5px;
  margin-bottom: 5px;
  overflow: hidden;
}

.handle-input {
  width: 300px;
  display: inline-block;
  margin-left: 10px;
  margin-right: 10px;
}
.pagination {
  display: flex;
  justify-content: center;
}

.crumbs {
  margin-bottom: 20px;
}
.play {
  position: absolute;
  z-index: 100;
  width: 87px;
  height: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  top: 15px;
  left: 11px;
  overflow: hidden;
}
.play:hover {
  background-color: rgba(71, 71, 71, 0.4);
  border-radius: 5px;
}

.icon-total {
  margin-top: 10px;
  margin-left: 5px;
}
.icon {
  width: 2em;
  height: 2em;
  color: rgb(254, 254, 255);
  fill: currentColor;
  overflow: hidden;
}
</style>