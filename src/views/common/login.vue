<template>
  <div class="site-wrapper site-page--login">
    <div class="site-content_wrapper">
      <div class="login-main">
        <h3 class="login-title">管理者ログイン</h3>
        <el-form :model="dataForm" status-icon :rules="rules" ref="dataForm" label-width="100px" class="demo-dataForm">
          <el-form-item label="アカウント" prop="userName">
            <el-input v-model="dataForm.userName" placeholder="アカウント"></el-input>
          </el-form-item>
          <el-form-item label="パスワード" prop="password">
            <el-input type="password" v-model="dataForm.password" placeholder="パスワード"></el-input>
          </el-form-item>
          <el-form-item label="認証コード" prop="captcha">
            <el-input class="captcha" v-model="dataForm.captcha"></el-input>
            <img class="captcha-img" :src="captchaPath" @click="getCaptcha()" placeholder="認証コード">
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="submitForm()">確認</el-button>
            <el-button type="primary" @click="resetForm()">リセット</el-button>
          </el-form-item>
        </el-form>
      </div>
    </div>
  </div>
</template>

<script>
import { getUUID } from "@/utils"
export default {
  data() {
    var validateUserName = (rule, value, callback) => {
      if (value === '' || value.trim() === '') {
        callback(new Error('アカウントを入力してください'));
      } else if (value.trim().length > 30) {
        callback(new Error('アカウントは長すぎます'));
      } else {
        callback();
      }

    };
    var validatePass = (rule, value, callback) => {
      if (value === '' || value.trim() === '') {
        callback(new Error('パスワードを入力してください'));
      } else if (value.trim().length > 30) {
        callback(new Error('パスワードは長すぎます'));
      } else {
        callback();
      }
    };
    var validateCaptcha = (rule, value, callback) => {
      if (value === '' || value.trim() === '') {
        callback(new Error('認証コードを入力してください'));
      } else if (value.trim().length > 30) {
        callback(new Error('認証コードは長すぎます'));
      } else {
        callback();
      }
    };
    return {
      dataForm: {
        userName: '',
        password: '',
        captcha: '',
        uuid: ''
      },
      rules: {
        userName: [
          { required: true, validator: validateUserName, trigger: 'change' }
        ],
        password: [
          { required: true, validator: validatePass, trigger: 'change' }
        ],
        captcha: [
          { required: true, validator: validateCaptcha, trigger: 'change' }
        ]
      },
      captchaPath: ""
    };
  },
  created() {
    this.getCaptcha();
  },
  methods: {
    getCaptcha() {
      this.dataForm.uuid = getUUID()
      this.captchaPath = this.$http.adornUrl(`/captcha.jpg?uuid=${this.dataForm.uuid}`)
    },
    submitForm() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl('/sys/login'),
            method: 'post',
            data: this.$http.adornData({
              'username': this.dataForm.userName,
              'password': this.dataForm.password,
              'captcha': this.dataForm.captcha,
              'uuid': this.dataForm.uuid
            })
          }).then(({ data }) => {
              if (data && data.code === 0) {
                this.$cookie.set('token', data.token)
                this.$router.replace({ name: 'home' })
              } else {
                this.getCaptcha()
                this.$message.error(data.msg)
              }
            })
        }
      });
    },
    resetForm() {
      this.$refs['dataForm'].resetFields();
    }
  }
}
</script>

<style lang="scss">
* {
  margin: 0px;
  padding: 0px;
}
.site-wrapper.site-page--login {
  width: 100%;
  height: 100%;
  .site-content_wrapper {
    width: 100%;
    height: 100%;
    background-color: rgba(38, 50, 56, 0.6);
    background-image: url(~@/assets/img/login_bg.jpg);
    background-size: cover;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .login-main {
    width: 420px;
  }

  .login-title {
    width: 310px;
    height: 36px;
    margin-left: 100px;
    font-size: 16px;
    color: aqua;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .el-form-item__label {
    color: #5e5e64;
  }
  .captcha {
    width: 280px;
  }
  .captcha-img {
    width: 120px;
    height: 36px;
    margin-left: 6px;
    border-radius: 6px;
  }
  .el-form-item__content {
    display: flex;
    justify-content: center;
  }

}
</style>