<template>
  <div class="login">
    <el-form ref="loginRef" :model="loginForm" :rules="loginRules" class="login-form">
      <h3 class="title">ruoyi-wms后台管理系统</h3>
      <el-form-item prop="username">
        <el-input
          v-model="loginForm.username"
          type="text"
          size="large"
          auto-complete="off"
          placeholder="账号"
        >
          <template #prefix><svg-icon icon-class="user" class="el-input__icon input-icon" /></template>
        </el-input>
      </el-form-item>
      <el-form-item prop="password">
        <el-input
          v-model="loginForm.password"
          type="password"
          size="large"
          auto-complete="off"
          placeholder="密码"
          @keyup.enter="handleLogin"
        >
          <template #prefix><svg-icon icon-class="password" class="el-input__icon input-icon" /></template>
        </el-input>
      </el-form-item>
      <el-form-item prop="code" v-if="captchaEnabled">
        <el-input
          v-model="loginForm.code"
          size="large"
          auto-complete="off"
          placeholder="验证码"
          style="width: 63%"
          @keyup.enter="handleLogin"
        >
          <template #prefix><svg-icon icon-class="validCode" class="el-input__icon input-icon" /></template>
        </el-input>
        <div class="login-code">
          <img :src="codeUrl" @click="getCode" class="login-code-img"/>
        </div>
      </el-form-item>
      <el-checkbox v-model="loginForm.rememberMe" style="margin:0px 0px 25px 0px;">记住密码</el-checkbox>
      <el-form-item style="width:100%;">
        <el-button
          :loading="loading"
          size="large"
          type="primary"
          class="login-btn"
          style="width:45%;"
          @click.prevent="handleLogin"
        >
          <span v-if="!loading">登 录</span>
          <span v-else>登 录 中...</span>
        </el-button>
        <el-button
          size="large"
          type="primary"
          class="try-btn"
          style="width:45%;"
          @click.native.prevent="handleTry"
        >
          <span>获取体验账号</span>
        </el-button>
        <div style="float: right;" v-if="register">
          <router-link class="link-type" :to="'/register'">立即注册</router-link>
        </div>
      </el-form-item>
    </el-form>
    <el-dialog
      title="公众号二维码"
      v-model="dialogVisible"
      append-to-body
      :show-close="false"
      width="30%">
      <div style="text-align: center">
        <span class="font-title-large"><span class="color-main font-extra-large">关注公众号</span>回复<span class="color-main font-extra-large">库存</span>获取体验账号</span>
        <br>
        <img src="@/assets/logo/gzh.jpg" width="160" height="160" style="margin-top: 10px">
      </div>
      <template #footer>
        <div class="dialog-footer">
          <el-button type="primary" @click="dialogConfirm">确定</el-button>
        </div>
      </template>
    </el-dialog>
    <!--  底部  -->
    <div class="el-login-footer">
      <span>Copyright © 2017-2024 ichengle.top 技术支持：关注“程序员诚哥”微信公众号，回复：支持</span>
    </div>
  </div>
</template>

<script setup>
import { getCodeImg } from "@/api/login";
import Cookies from "js-cookie";
import { encrypt, decrypt } from "@/utils/jsencrypt";
import useUserStore from '@/store/modules/user'

const userStore = useUserStore()
const router = useRouter();
const { proxy } = getCurrentInstance();

const loginForm = ref({
  username: "",
  password: "",
  rememberMe: false,
  code: "",
  uuid: ""
});

const loginRules = {
  username: [{ required: true, trigger: "blur", message: "请输入您的账号" }],
  password: [{ required: true, trigger: "blur", message: "请输入您的密码" }],
  code: [{ required: true, trigger: "change", message: "请输入验证码" }]
};

const codeUrl = ref("");
const loading = ref(false);
// 验证码开关
const captchaEnabled = ref(true);
// 注册开关
const register = ref(false);
const redirect = ref(undefined);
const dialogVisible = ref(false);

function handleTry(){
  dialogVisible.value =true
}
function dialogConfirm(){
  dialogVisible.value =false;
}

function handleLogin() {
  proxy.$refs.loginRef.validate(valid => {
    if (valid) {
      loading.value = true;
      // 勾选了需要记住密码设置在 cookie 中设置记住用户名和密码
      if (loginForm.value.rememberMe) {
        Cookies.set("username", loginForm.value.username, { expires: 30 });
        Cookies.set("password", encrypt(loginForm.value.password), { expires: 30 });
        Cookies.set("rememberMe", loginForm.value.rememberMe, { expires: 30 });
      } else {
        // 否则移除
        Cookies.remove("username");
        Cookies.remove("password");
        Cookies.remove("rememberMe");
      }
      // 调用action的登录方法
      userStore.login(loginForm.value).then(() => {
        router.push({ path: redirect.value || "/" });
      }).catch(() => {
        loading.value = false;
        // 重新获取验证码
        if (captchaEnabled.value) {
          getCode();
        }
      });
    }
  });
}

function getCode() {
  getCodeImg().then(res => {
    captchaEnabled.value = res.data.captchaEnabled === undefined ? true : res.data.captchaEnabled;
    if (captchaEnabled.value) {
      codeUrl.value = "data:image/gif;base64," + res.data.img;
      loginForm.value.uuid = res.data.uuid;
    }
  });
}

function getCookie() {
  const username = Cookies.get("username");
  const password = Cookies.get("password");
  const rememberMe = Cookies.get("rememberMe");
  loginForm.value = {
    username: username === undefined ? loginForm.value.username : username,
    password: password === undefined ? loginForm.value.password : decrypt(password),
    rememberMe: rememberMe === undefined ? false : Boolean(rememberMe)
  };
}

getCode();
getCookie();
</script>

<style lang='scss' scoped>
// 浅色（默认）：绿色背景 + 白色登录卡片
// 正文/提示文本与白底对比度 >= 4.5:1，UI 边界 >= 3:1
$login-bg-base: #1F6F58;
$login-bg-mid: #26795F;
$login-bg-deep: #1B5E4A;
$login-text: #303133; // 白底 12.6:1
$login-text-muted: #595959; // 白底 7.0:1
$login-border: #767676; // 白底 4.5:1
$login-accent: #1D4ED8; // 深蓝，白底 6.7:1
$login-accent-hover: #1E40AF;
$login-accent-active: #1E3A8A;

// 深色（prefers-color-scheme: dark）：近黑绿背景 + 深色卡片
$dark-bg-base: #0A2018;
$dark-bg-mid: #0F2E21;
$dark-bg-deep: #06140E;
$dark-surface: #1C1C1C;
$dark-field: #121212;
$dark-text: #F5F5F5; // 深色卡片 15.6:1
$dark-text-muted: #C8C8C8; // 深色卡片 10.2:1
$dark-border: #8A8A8A; // 深色卡片 4.9:1
$dark-accent: #93C5FD; // 深色卡片 9.5:1
$dark-btn-bg: #60A5FA;
$dark-btn-bg-hover: #93C5FD;
$dark-btn-bg-active: #BFDBFE;
$dark-btn-text: #0B1F17; // 与 #60A5FA 对比 6.8:1

.color-main {
  color: $login-accent;
}
.font-extra-large {
  font-size: 20px;
}
.login {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  background-color: $login-bg-base;
  background-image: linear-gradient(160deg, $login-bg-deep 0%, $login-bg-mid 50%, $login-bg-base 100%);
  background-repeat: no-repeat;
  background-size: cover;
}
.title {
  margin: 0px auto 30px auto;
  text-align: center;
  color: $login-text;
}

.login-form {
  border-radius: 6px;
  background: #ffffff;
  border: 1px solid rgba(0, 0, 0, 0.16);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.28);
  width: 400px;
  padding: 25px 25px 5px 25px;
  .el-input {
    height: 40px;
    input {
      height: 40px;
    }
  }
  .input-icon {
    height: 39px;
    width: 14px;
    margin-left: 0px;
    color: #606266;
  }
  // 输入框保持白底，边框加深以保证可见性
  :deep(.el-input__wrapper) {
    background-color: #ffffff;
    box-shadow: 0 0 0 1px $login-border inset;

    &:hover,
    &.is-focus {
      box-shadow: 0 0 0 1px $login-accent inset;
    }
  }
  :deep(.el-input__inner) {
    color: $login-text;
  }
  :deep(.el-input__inner::placeholder) {
    color: $login-border;
  }
  :deep(.el-checkbox__label) {
    color: $login-text;
  }
  :deep(.el-checkbox__inner) {
    border-color: $login-border;
  }
  .link-type,
  .link-type:focus {
    color: $login-accent;

    &:hover {
      color: $login-accent-active;
    }
  }
  // 主按钮：深蓝，与白色卡片、绿色背景均形成明显区分
  .login-btn {
    background-color: $login-accent;
    border-color: $login-accent;
    color: #ffffff;
    font-weight: 600;

    &:hover,
    &:focus {
      background-color: $login-accent-hover;
      border-color: $login-accent-hover;
      color: #ffffff;
    }

    &:active {
      background-color: $login-accent-active;
      border-color: $login-accent-active;
      color: #ffffff;
    }
  }
  // 次级按钮：白底描边，不与主按钮争夺视觉焦点
  .try-btn {
    background-color: #ffffff;
    border-color: #606266;
    color: $login-text;

    &:hover,
    &:focus {
      background-color: #eef4f1;
      border-color: $login-accent;
      color: $login-accent;
    }

    &:active {
      background-color: #dde8e3;
      border-color: $login-accent-active;
      color: $login-accent-active;
    }
  }
}
.login-tip {
  font-size: 13px;
  text-align: center;
  color: $login-text-muted;
}
.login-code {
  width: 33%;
  height: 40px;
  float: right;
  img {
    cursor: pointer;
    vertical-align: middle;
  }
}
.el-login-footer {
  height: 40px;
  line-height: 40px;
  position: fixed;
  bottom: 0;
  width: 100%;
  text-align: center;
  color: #ffffff;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.55);
  font-family: Arial;
  font-size: 12px;
  letter-spacing: 1px;
}
.login-code-img {
  height: 40px;
  padding-left: 12px;
  border-radius: 4px;
  background-color: #ffffff;
}

// 深色模式：背景改为深绿/近黑，卡片与文本反色
@media (prefers-color-scheme: dark) {
  .login {
    background-color: $dark-bg-base;
    background-image: linear-gradient(160deg, $dark-bg-deep 0%, $dark-bg-mid 50%, $dark-bg-base 100%);
  }
  .login-form {
    background: $dark-surface;
    border-color: rgba(255, 255, 255, 0.28);
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.6);

    .input-icon {
      color: #D4D4D4;
    }
    :deep(.el-input__wrapper) {
      background-color: $dark-field;
      box-shadow: 0 0 0 1px $dark-border inset;

      &:hover,
      &.is-focus {
        box-shadow: 0 0 0 1px $dark-accent inset;
      }
    }
    :deep(.el-input__inner) {
      color: $dark-text;
    }
    :deep(.el-input__inner::placeholder) {
      color: #A3A3A3;
    }
    :deep(.el-checkbox__label) {
      color: $dark-text;
    }
    :deep(.el-checkbox__inner) {
      background-color: $dark-field;
      border-color: $dark-border;
    }
    .link-type,
    .link-type:focus {
      color: $dark-accent;

      &:hover {
        color: #BFDBFE;
      }
    }
    .login-btn {
      background-color: $dark-btn-bg;
      border-color: $dark-btn-bg;
      color: $dark-btn-text;

      &:hover,
      &:focus {
        background-color: $dark-btn-bg-hover;
        border-color: $dark-btn-bg-hover;
        color: $dark-btn-text;
      }

      &:active {
        background-color: $dark-btn-bg-active;
        border-color: $dark-btn-bg-active;
        color: $dark-btn-text;
      }
    }
    .try-btn {
      background-color: #2A2A2A;
      border-color: #D4D4D4;
      color: $dark-text;

      &:hover,
      &:focus {
        background-color: #3A3A3A;
        border-color: $dark-accent;
        color: $dark-accent;
      }

      &:active {
        background-color: #454545;
        border-color: $dark-accent;
        color: $dark-accent;
      }
    }
  }
  .title {
    color: $dark-text;
  }
  .login-tip {
    color: $dark-text-muted;
  }
  .color-main {
    color: $dark-accent;
  }
  .el-login-footer {
    color: #E8E8E8;
    text-shadow: none;
  }
  .login-code-img {
    background-color: #ffffff;
  }
}

// 强制色模式：使用系统级高对比度配色，移除背景图与阴影
@media (forced-colors: active) {
  .login {
    background-color: Canvas;
    background-image: none;
  }
  .login-form {
    background: Canvas;
    border: 1px solid CanvasText;
    box-shadow: none;

    .input-icon {
      color: CanvasText;
    }
    :deep(.el-input__wrapper) {
      background-color: Field;
      border: 1px solid FieldText;
      box-shadow: none;
    }
    :deep(.el-input__inner) {
      color: FieldText;
    }
    :deep(.el-input__inner::placeholder) {
      color: GrayText;
    }
    :deep(.el-checkbox__label) {
      color: CanvasText;
    }
    :deep(.el-checkbox__inner) {
      background-color: Field;
      border-color: FieldText;
    }
    .link-type,
    .link-type:focus,
    .link-type:hover {
      color: LinkText;
    }
    .login-btn {
      background-color: Highlight;
      border-color: Highlight;
      color: HighlightText;

      &:hover,
      &:focus,
      &:active {
        background-color: Highlight;
        border-color: Highlight;
        color: HighlightText;
      }
    }
    .try-btn {
      background-color: ButtonFace;
      border-color: ButtonText;
      color: ButtonText;

      &:hover,
      &:focus,
      &:active {
        background-color: ButtonFace;
        border-color: ButtonText;
        color: ButtonText;
      }
    }
  }
  .title,
  .login-tip,
  .color-main {
    color: CanvasText;
  }
  .el-login-footer {
    color: CanvasText;
    text-shadow: none;
  }
  .login-code-img {
    background-color: Canvas;
  }
}
</style>
