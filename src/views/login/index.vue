<template>
  <div class="login-wrap">
    <div class="form-wrap">
      <div class="login-head">
        <img src="@/assets/logo_index.png" alt="黑马头条号">
      </div>
      <el-form ref="form" :model="form">
        <el-form-item>
          <el-input v-model="form.mobile" placeholder="手机号"></el-input>
        </el-form-item>
        <el-form-item>
          <el-col :span="14">
            <el-input v-model="form.code" placeholder="验证码"></el-input>
          </el-col>
          <el-col :offset="1" :span="9" >
            <el-button class="sendCode" @click="handleSendCode">获取验证码</el-button>
          </el-col>
        </el-form-item>
        <el-form-item>
          <el-button class="btn-login" type="primary" @click="handleLogin">登录</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import '@/vendor/gt'
export default {
  name: 'AppLogin',
  data () {
    return {
      form: {
        mobile: '',
        code: ''
      }
    }
  },
  methods: {
    handleLogin () {
      axios({
        method: 'POST',
        url: 'http://ttapi.research.itcast.cn/mp/v1_0/authorizations',
        data: this.form
      })
        .then(res => { // >=200 && < 400 的状态码会进入 then 成功
          console.log(res.data)
          this.$message({
            message: '登录成功',
            type: 'success'
          })
          this.$router.push({
            name: 'home'
          })
        })
        .catch((e) => {
          this.$message.error('登录失败，手机号或验证码错误')
        }) // >= 400 的状态码都会进入这里
    },
    handleSendCode () {
      const { mobile } = this.form
      axios({
        methods: 'GET',
        url: `http://ttapi.research.itcast.cn/mp/v1_0/captchas/${mobile}`
      }).then(res => {
        const { data } = res.data
        window.initGeetest({
          gt: data.gt,
          challenge: data.challenge,
          offline: !data.success,
          new_captcha: data.new_captcha,
          product: 'bind'
        }, function (captchaObj) {
          captchaObj.onReady(function () {
            captchaObj.verify()
          }).onSuccess(function () {
            const {
              geetest_challenge: challenge,
              geetest_seccode: seccode,
              geetest_validate: validate } =
            captchaObj.getValidate()
            axios({
              method: 'GET',
              url: `http://ttapi.research.itcast.cn/mp/v1_0/sms/codes/${mobile}`,
              params: {
                challenge,
                validate,
                seccode
              }
            }).then(res => {
              console.log(res.data)
            })
          }).onError(function () {
          })
        })
      })
    }
  }
}
</script>

<style lang="less" scoped>
.login-wrap {
  height: 100%;
  background-color: #4f0;
  display: flex;
  justify-content: center;
  align-items: center;
  .login-head {
    display: flex;
    justify-content: center;
    align-items: center;
    margin-bottom: 10px;
    .img {
      width: 200px;
    }
  }
  .form-wrap {
    width: 400px;
    background-color: #fff;
    padding: 20px;
    border-radius: 10px;
    .sendCode {
      float:right;
    }
    .btn-login {
      width: 100%;
    }
  }
}
</style>
