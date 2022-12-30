<template>
  <div class="login_container">
    <vue-particles
      :click-effect="true"
      :hover-effect="true"
      :line-linked="true"
      :line-opacity="0.4"
      :lines-distance="150"
      :lines-width="1"
      :move-speed="2"
      :particle-opacity="0.7"
      :particle-size="4"
      :particles-number="60"
      class="lizi"
      click-mode="push"
      color="#fff"
      hover-mode="grab"
      lines-color="#fff"
      shape-type="circle"
      style="height:100%;position: relative"
    >
    </vue-particles>
    <el-form
      ref="loginForm"
      :model="loginForm"
      :rules="loginRules"
      auto-complete="on"
      class="login-form"
      label-position="left"
    >

      <div class="title-container">
        <h3 class="title">Login Form</h3>
      </div>

      <el-form-item prop="username">
          <span class="svg-container">
            <svg-icon icon-class="user"/>
          </span>
        <el-input
          ref="username"
          v-model="loginForm.username"
          auto-complete="on"
          name="username"
          placeholder="Username"
          tabindex="1"
          type="text"
        />
      </el-form-item>

      <el-form-item prop="password">
          <span class="svg-container">
            <svg-icon icon-class="password"/>
          </span>
        <el-input
          :key="passwordType"
          ref="password"
          v-model="loginForm.password"
          :type="passwordType"
          auto-complete="on"
          name="password"
          placeholder="Password"
          tabindex="2"
          @keyup.enter.native="handleLogin"
        />
        <span class="show-pwd" @click="showPwd">
            <svg-icon :icon-class="passwordType === 'password' ? 'eye' : 'eye-open'"/>
          </span>
      </el-form-item>

      <el-button
        :loading="loading"
        style="width:100%;margin-bottom:30px;"
        type="primary"
        @click.native.prevent="handleLogin"
      >登录
      </el-button>

    </el-form>
  </div>
</template>
<script>
import { validUsername } from '@/utils/validate'

export default {
  name: 'Login',
  data() {
    const validateUsername = (rule, value, callback) => {
      if (!validUsername(value)) {
        callback(new Error('Please enter the correct user name'))
      } else {
        callback()
      }
    }
    const validatePassword = (rule, value, callback) => {
      if (value.length < 6) {
        callback(new Error('The password can not be less than 6 digits'))
      } else {
        callback()
      }
    }
    return {
      loginForm: {
        username: 'admin',
        password: '111111'
      },
      loginRules: {
        username: [{ required: true, trigger: 'blur', validator: validateUsername }],
        password: [{ required: true, trigger: 'blur', validator: validatePassword }]
      },
      loading: false,
      passwordType: 'password',
      redirect: undefined
    }
  },
  watch: {
    $route: {
      handler: function(route) {
        this.redirect = route.query && route.query.redirect
      },
      immediate: true
    }
  },
  methods: {
    showPwd() {
      if (this.passwordType === 'password') {
        this.passwordType = ''
      } else {
        this.passwordType = 'password'
      }
      this.$nextTick(() => {
        this.$refs.password.focus()
      })
    },
    handleLogin() {
      this.$refs.loginForm.validate(valid => {
        if (valid) {
          this.loading = true
          this.$store.dispatch('user/login', this.loginForm).then(() => {
            this.$router.push({ path: this.redirect || '/' })
            this.loading = false
          }).catch(() => {
            this.loading = false
          })
        } else {
          console.log('error submit!!')
          return false
        }
      })
    }
  }
}
</script>
<style lang="scss" scoped>
$bg: #2d3a4b;
$dark_gray: #889aa4;
$light_gray: #eee;
$cursor: #fff;

.login_container {
  background-image: linear-gradient(-180deg, #1a1454 0%, #0e81a5 100%);
  /*background-image: url("../images/bg_login.png");*/
  background-repeat: no-repeat;
  background-size: cover;
  height: 100%;
  overflow: hidden;

  .login-form {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-50%);
    width: 420px;
    max-width: 100%;
    padding: 50px 35px;
    border-radius: 20px;
    margin: 0 auto;
    overflow: hidden;
    background: #ffffff;
    .el-form-item {
      border: 1px solid #aaaaaa;
      background: #ffffff;
      border-radius: 5px;
      color: #454545;
      .el-input {
        display: inline-block;
        height: 47px;
        width: 85%;

      ::v-deep .el-input__inner {
          //background: transparent;
          border: 0px !important;
          -webkit-appearance: none;
          border-radius: 0px;
          padding: 12px 5px 12px 15px;
          //color: $light_gray;
          height: 47px;
          caret-color: $cursor;

          &:-webkit-autofill {
            box-shadow: 0 0 0px 1000px $bg inset !important;
            -webkit-text-fill-color: $cursor !important;
          }
        }
      }
    }
  }

  .tips {
    font-size: 14px;
    color: #fff;
    margin-bottom: 10px;

    span {
      &:first-of-type {
        margin-right: 16px;
      }
    }
  }

  .svg-container {
    padding: 6px 5px 6px 15px;
    color: $dark_gray;
    vertical-align: middle;
    width: 30px;
    display: inline-block;
  }

  .title-container {
    position: relative;

    .title {
      font-size: 26px;
      color: $light_gray;
      margin: 0px auto 40px auto;
      text-align: center;
      font-weight: bold;
    }
  }

  .show-pwd {
    position: absolute;
    right: 10px;
    top: 7px;
    font-size: 16px;
    color: $dark_gray;
    cursor: pointer;
    user-select: none;
  }

}
</style>
