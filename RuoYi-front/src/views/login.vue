<template>
  <div class="login">
    <div class="login-bg" aria-hidden="true">
      <span class="blob blob-1"></span>
      <span class="blob blob-2"></span>
      <span class="blob blob-3"></span>
    </div>

    <div class="login-panel">
      <div class="brand">
        <img
          class="brand-logo"
          src="@/assets/login/luxshare-logo.svg"
          alt="Luxshare ICT"
        />
        <img
          class="brand-mascot"
          src="@/assets/login/likeniu-mascot.png"
          alt=""
        />
      </div>

      <h3 class="title">{{ title }}</h3>
      <p class="subtitle">面向生产数据查询、质量分析与报告生成的智能工作台</p>

      <el-form
        ref="loginRef"
        :model="loginForm"
        :rules="loginRules"
        class="login-form"
      >
        <el-form-item prop="username">
          <el-input
            v-model="loginForm.username"
            type="text"
            size="large"
            auto-complete="off"
            placeholder="账号"
          >
            <template #prefix>
              <svg-icon icon-class="user" class="input-icon" />
            </template>
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
            <template #prefix>
              <svg-icon icon-class="password" class="input-icon" />
            </template>
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
            <template #prefix>
              <svg-icon icon-class="validCode" class="input-icon" />
            </template>
          </el-input>
          <div class="login-code">
            <img :src="codeUrl" @click="getCode" class="login-code-img" />
          </div>
        </el-form-item>

        <el-checkbox v-model="loginForm.rememberMe" class="remember"
          >记住密码</el-checkbox
        >

        <el-form-item style="width: 100%">
          <el-button
            class="login-btn"
            :loading="loading"
            size="large"
            type="primary"
            @click.prevent="handleLogin"
          >
            <span v-if="!loading">登 录</span>
            <span v-else>登 录 中...</span>
          </el-button>
          <div style="float: right" v-if="register">
            <router-link class="link-type" :to="'/register'"
              >立即注册</router-link
            >
          </div>
        </el-form-item>
      </el-form>

      <div class="el-login-footer">
        <span>{{ footerContent }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
// ↓↓↓ 与原来完全一致，未做任何修改 ↓↓↓
import { getCodeImg } from "@/api/login";
import Cookies from "js-cookie";
import { encrypt, decrypt } from "@/utils/jsencrypt";
import useUserStore from "@/store/modules/user";
import defaultSettings from "@/settings";

const title = import.meta.env.VITE_APP_TITLE;
const footerContent = defaultSettings.footerContent;
const userStore = useUserStore();
const route = useRoute();
const router = useRouter();
const { proxy } = getCurrentInstance();

const loginForm = ref({
  username: "",
  password: "",
  rememberMe: false,
  code: "",
  uuid: "",
});

const loginRules = {
  username: [{ required: true, trigger: "blur", message: "请输入您的账号" }],
  password: [{ required: true, trigger: "blur", message: "请输入您的密码" }],
  code: [{ required: true, trigger: "change", message: "请输入验证码" }],
};

const codeUrl = ref("");
const loading = ref(false);
const captchaEnabled = ref(true);
const register = ref(false);
const redirect = ref(undefined);

watch(
  route,
  (newRoute) => {
    redirect.value = newRoute.query && newRoute.query.redirect;
  },
  { immediate: true }
);

function handleLogin() {
  proxy.$refs.loginRef.validate((valid) => {
    if (valid) {
      loading.value = true;
      if (loginForm.value.rememberMe) {
        Cookies.set("username", loginForm.value.username, { expires: 30 });
        Cookies.set("password", encrypt(loginForm.value.password), {
          expires: 30,
        });
        Cookies.set("rememberMe", loginForm.value.rememberMe, { expires: 30 });
      } else {
        Cookies.remove("username");
        Cookies.remove("password");
        Cookies.remove("rememberMe");
      }
      userStore
        .login(loginForm.value)
        .then(() => {
          const query = route.query;
          const otherQueryParams = Object.keys(query).reduce((acc, cur) => {
            if (cur !== "redirect") {
              acc[cur] = query[cur];
            }
            return acc;
          }, {});
          router.push({ path: redirect.value || "/", query: otherQueryParams });
        })
        .catch(() => {
          loading.value = false;
          if (captchaEnabled.value) {
            getCode();
          }
        });
    }
  });
}

function getCode() {
  getCodeImg().then((res) => {
    captchaEnabled.value =
      res.captchaEnabled === undefined ? true : res.captchaEnabled;
    if (captchaEnabled.value) {
      codeUrl.value = "data:image/gif;base64," + res.img;
      loginForm.value.uuid = res.uuid;
    }
  });
}

function getCookie() {
  const username = Cookies.get("username");
  const password = Cookies.get("password");
  const rememberMe = Cookies.get("rememberMe");
  loginForm.value = {
    username: username === undefined ? loginForm.value.username : username,
    password:
      password === undefined ? loginForm.value.password : decrypt(password),
    rememberMe: rememberMe === undefined ? false : Boolean(rememberMe),
  };
}

getCode();
getCookie();
</script>

<style lang="scss" scoped>
/* 与 MES 智能助手保持同一套品牌色：
   深蓝 #063b77 / 主蓝 #0f6fbd / 强调 #22a7d8
   背景 #ffffff / #f8fafc / 文字 #1e293b / #64748b */

.login {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  overflow: hidden;
  background: linear-gradient(135deg, #f7fbff 0%, #eef5fd 45%, #e6f0fa 100%);
}

.login-bg {
  position: absolute;
  inset: 0;
  pointer-events: none;

  .blob {
    position: absolute;
    border-radius: 50%;
    filter: blur(90px);
  }
  .blob-1 {
    width: 520px;
    height: 520px;
    top: -180px;
    right: -120px;
    background: rgba(34, 167, 216, 0.32);
  }
  .blob-2 {
    width: 460px;
    height: 460px;
    bottom: -200px;
    left: -140px;
    background: rgba(15, 111, 189, 0.24);
  }
  .blob-3 {
    width: 300px;
    height: 300px;
    top: 42%;
    left: 56%;
    background: rgba(6, 59, 119, 0.1);
  }
}

.login-panel {
  position: relative;
  z-index: 1;
  width: 420px;
  padding: 38px 36px 24px;
  border-radius: 20px;
  background: rgba(255, 255, 255, 0.94);
  border: 1px solid rgba(226, 232, 240, 0.9);
  box-shadow: 0 24px 60px rgba(6, 59, 119, 0.12),
    0 2px 8px rgba(6, 59, 119, 0.06);
}

.brand {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 14px;
  margin-bottom: 18px;
}
.brand-logo {
  height: 38px;
}
.brand-mascot {
  height: 54px;
}

.title {
  margin: 0 0 6px;
  text-align: center;
  font-size: 24px;
  font-weight: 600;
  letter-spacing: 2px;
  color: #063b77;
}

.subtitle {
  margin: 0 0 26px;
  text-align: center;
  font-size: 13px;
  color: #64748b;
}

.login-form {
  :deep(.el-form-item) {
    margin-bottom: 20px;
  }

  :deep(.el-input__wrapper) {
    height: 44px;
    border-radius: 10px;
    padding-left: 12px;
    box-shadow: 0 0 0 1px #e2e8f0 inset;
    transition: box-shadow 0.2s ease;
  }
  :deep(.el-input__wrapper:hover) {
    box-shadow: 0 0 0 1px #cbd5e1 inset;
  }
  :deep(.el-input__wrapper.is-focus) {
    box-shadow: 0 0 0 2px rgba(15, 111, 189, 0.28) inset;
  }
  :deep(.el-input__inner) {
    height: 44px;
  }

  .input-icon {
    height: 18px;
    width: 16px;
    color: #94a3b8;
  }
}

.remember {
  margin: 0 0 22px 0;
  :deep(.el-checkbox__label) {
    color: #64748b;
    font-size: 13px;
  }
}

.login-btn {
  width: 100%;
  height: 44px;
  border: none;
  border-radius: 10px;
  font-size: 15px;
  letter-spacing: 4px;
  background: linear-gradient(135deg, #0f6fbd 0%, #063b77 100%);
  box-shadow: 0 8px 20px rgba(15, 111, 189, 0.28);
  transition: all 0.2s ease;

  &:hover {
    background: linear-gradient(135deg, #1380d6 0%, #0a4d94 100%);
    box-shadow: 0 10px 24px rgba(15, 111, 189, 0.34);
  }
}

.login-code {
  width: 33%;
  height: 44px;
  float: right;

  .login-code-img {
    height: 44px;
    border-radius: 8px;
    cursor: pointer;
    border: 1px solid #e2e8f0;
  }
}

.el-login-footer {
  margin-top: 18px;
  text-align: center;
  font-size: 12px;
  letter-spacing: 0.5px;
  color: #94a3b8;
}

@media (max-width: 480px) {
  .login-panel {
    width: calc(100% - 32px);
    padding: 32px 22px 20px;
  }
}

/* 暗色模式：保持同一套层次，只换底色 */
html.dark .login {
  background: linear-gradient(135deg, #0d1420 0%, #111a28 55%, #0a1626 100%);

  .blob-1 {
    background: rgba(34, 167, 216, 0.18);
  }
  .blob-2 {
    background: rgba(15, 111, 189, 0.14);
  }
  .blob-3 {
    background: rgba(6, 59, 119, 0.12);
  }
}

html.dark .login-panel {
  background: rgba(22, 30, 44, 0.92);
  border-color: rgba(255, 255, 255, 0.06);
  box-shadow: 0 24px 60px rgba(0, 0, 0, 0.5);

  .title {
    color: #cfe3fb;
  }
  .subtitle,
  .remember :deep(.el-checkbox__label) {
    color: #94a3b8;
  }
}
</style>
