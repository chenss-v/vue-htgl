<template>
  <div class="image-list">

    <div class="action-head">
      <el-radio-group v-model="collect" size="mini" @change="loadImages(1)">
        <el-radio-button :label="false">全部</el-radio-button>
        <el-radio-button :label="true">收藏</el-radio-button>
      </el-radio-group>
      <el-button v-if="isShowAdd" size="mini" type="success" @click="dialogUploadVisible = true">上传素材</el-button>
    </div>

    <el-row :gutter="10">
      <el-col
        class="image-item"
        :xs="12" :lg="4"
        :md="6" :sm="6"
        v-for="(img, index) in images"
        :key="index"
        @click.native="selected = index"
      >
        <el-image style="height: 100px" :src="img.url" fit="cover"></el-image>
        <div class="selected" v-if="isShowSelected && selected === index"></div>
        <div class="image-action" v-if="isShowAction">
          <el-button
            type="warning"
            :icon="img.is_collected ? 'el-icon-star-on' : 'el-icon-star-off'"
            circle
            size="small"
            @click="onCollect(img)"
            :loading="img.loading"
          ></el-button>
          <el-button
            type="danger"
            size="small"
            icon="el-icon-delete-solid"
            circl
            @click="onDelete(img)"
            :loading="img.loading"
          ></el-button>
        </div>
      </el-col>
    </el-row>

    <el-pagination
      class="image-pagination"
      layout="prev, pager, next"
      background
      :total="totalCount"
      :page-size="pageSize"
      :current-page.sync="page"
      @current-change="onPageChange">
    </el-pagination>

    <el-dialog title="上传素材" :visible.sync="dialogUploadVisible" :append-to-body="true">
      <el-upload
        class="upload-demo"
        drag
        action="http://api-toutiao-web.itheima.net/mp/v1_0/user/images"
        :headers="uploadHeaders"
        name="image"
        :on-success="onUploadSuccess"
        multiple>
        <i class="el-icon-upload"></i>
        <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
        <div class="el-upload__tip" slot="tip">只能上传jpg/png文件，且不超过500kb</div>
      </el-upload>
    </el-dialog>
  </div>
</template>

<script>
import { getImage, collectImage, deleteImage } from '@/network/image'

export default {
  name: 'ImageList',
  data () {
    const user = JSON.parse(window.localStorage.getItem('user'))
    return {
      collect: false,
      images: [],
      dialogUploadVisible: false,
      uploadHeaders: {
        Authorization: `Bearer ${user.token}`
      },
      totalCount: 0,
      pageSize: 20,
      page: 1,
      selected: null
    }
  },
  props: {
    isShowAdd: {
      type: Boolean,
      default: true
    },
    isShowAction: {
      type: Boolean,
      default: true
    },
    isShowSelected: { // 素材图片 被选择 默认不显示
      type: Boolean,
      default: false
    }
  },
  components: {
  },
  created () {
    this.loadImages(1)
  },
  methods: {
    loadImages (page = 1) {
      this.page = page
      getImage({
        collect: this.collect,
        page,
        per_page: this.pageSize
      }).then(res => {
        const results = res.data.data.results
        results.forEach(img => {
          img.loading = false
        })
        this.images = results
        this.totalCount = res.data.data.total_count
      })
    },
    onCollectChange (value) {
      this.loadImages(value)
    },
    onUploadSuccess () {
      this.dialogUploadVisible = false
      this.loadImages(this.page)
    },
    onPageChange (page) {
      this.loadImages(page)
    },
    onCollect (img) {
      img.loading = true
      collectImage(img.id, !img.is_collected).then(res => {
        img.is_collected = !img.is_collected
        img.loading = false
      })
    },
    onDelete (img) {
      img.loading = true
      deleteImage(img.id).then(res => {
        this.loadImages(this.page)
        img.loading = false
      })
    }
  }
}
</script>

<style scoped lang="less">
.image-pagination{
  float: right;
  padding: 20px 0;
}
.image-item{
  position: relative;
}
.image-action{
  font-size: 22px;
  height: 33px;
  color: #ccc;
  position: absolute;
  left: 10px;
  right: 10px;
  bottom: 14px;
  display: flex;
  justify-content: space-evenly;
  align-items: center;
  background-color: rgba(0, 0, 0, .3);
}
.selected {
  background: url('./selected.png') no-repeat;
  background-size: cover;
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  bottom: 0;
}
</style>
