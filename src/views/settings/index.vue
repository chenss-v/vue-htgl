<template>
  <div class="">
    <el-card class="box-card">
      <div slot="header" class="clearfix">
        <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
          <el-breadcrumb-item>个人设置</el-breadcrumb-item>
        </el-breadcrumb>
      </div>
      <el-row>
        <el-col :span="15">
          <el-form ref="form" :model="user" :rules="formRules" label-width="80px">
            <el-form-item label="编号：">
              {{ user.id }}
            </el-form-item>
            <el-form-item label="手机：">
              {{ user.mobile }}
            </el-form-item>
            <el-form-item label="媒体名称:" prop="name">
              <el-input v-model="user.name"></el-input>
            </el-form-item>
            <el-form-item label="媒体介绍:" prop="intro">
              <el-input type="textarea" v-model="user.intro"></el-input>
            </el-form-item>
            <el-form-item label="邮箱:" prop="email">
              <el-input v-model="user.email"></el-input>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="onUpdataUser" :loading="updataUserProfileLoading">保存</el-button>
            </el-form-item>
          </el-form>
        </el-col>
        <el-col :offset="2" :span="4">
          <label for="file">
            <el-avatar shape="square" :size="100" fit="cover" :src="user.photo"></el-avatar>
            <p>点击修改头像</p>
          </label>
          <!-- <p @click="$refs.file.click()">点击修改头像</p> -->
          <input id="file" type="file" hidden ref="file" @change="onFileChange()">
        </el-col>
      </el-row>
    </el-card>

    <el-dialog
      title="提示"
      :visible.sync="dialogVisible"
      append-to-body
      @opened="onDialogOpened"
      @closed="onDialogClosed"
      >
      <div class="preview-image-wrap">
        <img class="preview-image" :src="previewImage" ref="preview-image">
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="onUpdataPhoto" :loading="updataPhotoLoading">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
import { getUserProfile, updataUserPhoto, updataUserProfile } from '@/network/user'
import 'cropperjs/dist/cropper.css'
import Cropper from 'cropperjs'
import globalBus from '@/network/global-bus'

export default {
  name: 'SettingIndex',
  data () {
    return {
      user: {
        email: '',
        id: null,
        intro: '',
        mobile: '',
        name: '',
        photo: ''
      },
      dialogVisible: false,
      previewImage: '',
      cropper: null,
      updataPhotoLoading: false,
      updataUserProfileLoading: false,
      formRules: {
        name: [
          { min: 1, max: 7, message: '长度在 1 到 7 个字符', trigger: 'blur' }
        ],
        intro: [
          { required: true, message: '请填写媒体介绍', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请填写媒体介绍', trigger: 'blur' }
        ]
      }
    }
  },
  components: {

  },
  created () {
    this.loadUser()
  },
  methods: {
    onUpdataUser () {
      this.updataUserProfileLoading = true
      const { name, intro, email } = this.user
      updataUserProfile({
        name,
        intro,
        email
      }).then(res => {
        this.$message({
          type: 'success',
          message: '保存成功'
        })
        this.updataUserProfileLoading = false
        globalBus.$emit('updata-user', this.user)
      })
    },
    loadUser () {
      getUserProfile().then(res => {
        this.user = res.data.data
      })
    },
    onFileChange () {
      const file = this.$refs.file
      const blobData = window.URL.createObjectURL(file.files[0])
      this.previewImage = blobData
      this.dialogVisible = true
      this.$refs.file.value = ''
    },
    onDialogOpened () {
      const image = this.$refs['preview-image']
      this.cropper = new Cropper(image, {
        aspectRatio: 1,
        viewMode: 1,
        dragMode: 'none',
        cropBoxResizable: false
      })
    },
    onDialogClosed () {
      this.cropper.destroy()
    },
    onUpdataPhoto () {
      this.updataPhotoLoading = true
      this.cropper.getCroppedCanvas().toBlob(fild => {
        const fd = new FormData()
        fd.append('photo', fild)
        updataUserPhoto(fd).then(res => {
          this.dialogVisible = false
          this.user.photo = window.URL.createObjectURL(fild)
          this.updataPhotoLoading = false
          this.$message({
            type: 'success',
            message: '更新头像成功'
          })
          globalBus.$emit('updata-user', this.user)
        })
      })
    }
  }
}
</script>

<style scoped lang="less">
.preview-image-wrap {
  .preview-image {
    display: block;
    max-width: 100%;
    height: 200px;
  }
}
</style>
