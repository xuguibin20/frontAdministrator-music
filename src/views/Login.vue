<template>
  <div class="login-wrap">
    <div class="total">
      <div class="ms-title">music后台管理登录</div>
      <div class="ms-login">
        <el-form :model="ruleForm" :rules="rules">
          <el-form-item prop="username">
            <el-input
              v-model="ruleForm.username"
              placeholder="username"
            ></el-input>
          </el-form-item>
          <el-form-item prop="password">
            <el-input
              type="password"
              v-model="ruleForm.password"
              placeholder="password"
              @keyup.enter="submitForm"
            ></el-input>
          </el-form-item>
          <el-form-item>
            <el-button class="login-btn" type="primary" @click="submitForm"
              >登录</el-button
            >
          </el-form-item>
        </el-form>
      </div>
    </div>
  </div>
</template>

<script>
import { mixin } from "@/mixins/index";
import { getLoginStatus } from "@/api/index";

export default {
  mixins: [mixin],
  data() {
    return {
      ruleForm: {
        username: "admin",
        password: "123",
      },
      rules: {
        username: [{ require: true, message: "请输入用户名", trigger: "blur" }],
        password: [{ require: true, message: "请输入密码", trigger: "blur" }],
      },
    };
  },
  methods: {
    submitForm() {
      let params = new URLSearchParams();
      params.append("name", this.ruleForm.username);
      params.append("password", this.ruleForm.password);
      getLoginStatus(params).then((res) => {
        if (res.code == 1) {
          localStorage.setItem("userName", this.ruleForm.username);
          this.notify("登陆成功", "success");
          this.$router.push("/Info");
        } else {
          this.notify("登陆失败", "error");
        }
      });
    },
  },
};
</script>

<style lang="less" scoped>
.login-wrap {
  position: relative;
  background-image: url("@/assets/images/background.jpg");
  background-repeat: no-repeat;
  background-attachment: fixed;
  background-size: cover;
  width: 100%;
  height: 100%;
}
.total {
  position: absolute;
  width: 500px;
  height: 260px;
  left: calc(50% - 320px);
  top: calc(50% - 200px);

  padding: 40px;
  border-radius: 5px;
}
.ms-title {
  position: absolute;
  left: 33%;
  top: 5%;
  width: 100%;
  font-size: 30px;
  font-weight: 600;
  color: black;
}

.ms-login {
  position: absolute;
  width: 300px;
  height: 160px;
  left: calc(50% - 150px);
  top: 25%;

  padding: 40px;
  border-radius: 5px;
  background: #fff;
}

.login-btn {
  width: 100%;
}
</style>
