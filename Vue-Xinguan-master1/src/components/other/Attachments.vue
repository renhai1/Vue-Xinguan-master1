<template>
  <div class="attachments">
    <!-- 面包导航 -->
    <el-breadcrumb
      separator="/"
      style="padding-left: 10px; padding-bottom: 10px; font-size: 12px"
    >
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>系统管理</el-breadcrumb-item>
      <el-breadcrumb-item>附件管理</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 图片列表部分 -->
    <el-card :style="{height:getHeight+'px'}">
      <!-- 搜索部分 -->
      <el-form :inline="true" :model="queryMap" class="demo-form-inline">
        <el-form-item label="图片路径">
          <el-input
            clearable
            @clear="search"
            v-model="queryMap.path"
            placeholder="输入图片路径查询"
          ></el-input>
        </el-form-item>
        <el-form-item label="图片后缀">
          <el-select
            clearable
            @clear="search"
            v-model="queryMap.suffix"
            placeholder="请选择"
          >
            <el-option label="jpg/JPG" value=".jpg"></el-option>
            <el-option label="png/PNG" value=".png"></el-option>
            <el-option label="gif/GIF" value=".gif"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" icon="el-icon-search" @click="search"
            >查询</el-button
          >
          <el-button type="primary" @click="centerDialogVisible = true">
            上传
            <i class="el-icon-upload el-icon--right"></i>
          </el-button>
        </el-form-item>
      </el-form>
      <!-- 图片展示部分 -->
      <el-row :gutter="20" style="height: 430px" v-loading="loading">
        <el-col
          style="margin-top: 10px"
          v-for="image in this.list"
          :key="image.id"
          :span="6"
        >
          <div class="grid-content bg-purple">
            <el-image
              :alt="image.path"
              :fit="fits"
              :preview-src-list="srcList"
              style="width: 200px; height: 170px"
              :src="'http://127.0.0.1:8082/xinguanxiangmutupian/' + image.path"
            >
              <div slot="error" class="image-slot">
                <i class="el-icon-picture-outline"></i>
              </div>
            </el-image>
            <div>
              <el-tag
                size="mini"
                effect="dark"
                type="success"
                style="margin-left: 50px"
                >{{ image.width }}px X {{ image.height }}px</el-tag
              >
              <el-popconfirm
                title="这是一段内容确定删除吗？"
                @confirm="del(image.id)"
              >
                <el-button
                  v-hasPermission="'image:delete'"
                  style="margin-left: 30px"
                  icon="el-icon-delete"
                  size="mini"
                  type="text"
                  slot="reference"
                  >删除</el-button
                >
              </el-popconfirm>
            </div>
          </div>
        </el-col>
      </el-row>

      <!-- 分页 -->
      <el-pagination
        style="margin-top: 80px"
        background
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryMap.pageNum"
        :page-sizes="[8, 20, 30, 40]"
        :page-size="queryMap.pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
      <!-- 上传弹出框 -->
      <el-dialog
        title="上传图片附件"
        @close="closeUpload"
        :visible.sync="centerDialogVisible"
        width="38%"
        center
      >
        <span>
          <el-upload
            accept="image/*"
            :auto-upload="false"
            :multiple="true"
            ref="upload"
            :limit="10"
            :on-exceed="exceed"
            class="upload-demo"
            drag
            :headers="headerObject"
            :on-success="handleUploadSuccess"
            :action="uploadUrl"
          >
            <i class="el-icon-upload"></i>
            <div class="el-upload__text">
              将文件拖到此处，或
              <em>点击上传</em>
            </div>
            <div class="el-upload__tip" slot="tip">
              只能上传jpg/png文件，且不超过500kb,最多支持10张图片一起上传
            </div>
          </el-upload>
        </span>
        <span slot="footer" class="dialog-footer">
          <el-button @click="closeUpload" size="small">取消返回</el-button>
          <el-button
            style="margin-left: 10px"
            size="small"
            type="success"
            @click="submitUpload"
            >上传到服务器</el-button
          >
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>


<script>
export default {
  data() {
    return {
      //http://localhost:8082/qlpqlp/api/upload/image
      uploadUrl: "http://www.localhost:8081/upload/image",
      centerDialogVisible: false,
      loading: true,
      total: 0,
      fits: "contain",
      queryMap: {},
      list: [],
      srcList: [],
      headerObject: {
        Authorization: window.localStorage.getItem("JWT_TOKEN"),
      }, //图片上传请求头
    };
  },
  computed: {
    getHeight() {
      return document.documentElement.clientHeight;
    },
  },
  methods: {
    /**
     * 取消上传
     */
    closeUpload() {
      this.centerDialogVisible = false;
      this.$refs.upload.clearFiles();
    },
    /**
     * 上传之后的回调
     */
    handleUploadSuccess: function (response, file, fileList) {
      if (200 !== response.code) {
        this.$refs.upload.clearFiles();
        return this.$message.error("上传失败:" + response.msg);
      } else {
        this.getImgeList();
      }
    },
    /**
     *
     * 点击上传到服务器
     */
    submitUpload() {
      this.$refs.upload.submit();
    },
    /**
     * 删除图片
     */
    del: async function (id) {
      const { data: res } = await this.$http.delete("upload/delete/" + id);
      if (res.code === 200) {
        this.$message.success("删除图片成功");
        this.getImgeList();
      } else {
        this.$message.error(res.msg);
      }
    },
    /**
     * 加载附件列表
     */
    async getImgeList() {
      const { data: res } = await this.$http.get("upload/findImageList", {
        params: this.queryMap,
      });
      if (res.code !== 200) {
        return this.$message.error("获取附件列表失败:" + res.msg);
      } else {
        var $this = this;
        this.total = res.data.total;
        this.list = res.data.list;
        this.srcList = [];
        this.list.forEach(function (item) {
          $this.srcList.push("http://127.0.0.1:8082/qlpqlp/" + item.path);
        });
      }
    },
    //改变页码
    handleSizeChange(newSize) {
      this.queryMap.pageSize = newSize;
      this.getImgeList();
    },
    //翻页
    handleCurrentChange(current) {
      this.queryMap.pageNum = current;
      this.getImgeList();
    },
    /**
     * 搜索
     */
    search() {
      this.queryMap.pageNum = 1;
      this.getImgeList();
    },
    /**
     * 超出允许上传的时候
     */
    exceed() {
      this.$message.warning("超出允许上传图片的数量");
    },
  },
  created() {
    this.getImgeList();
    setTimeout(() => {
      this.loading = false;
    }, 500);
  },
};
</script>

<style>
.el-upload-dragger {
  width: 530px !important;
}
</style>
