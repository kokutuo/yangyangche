<template>
  <div>
    <Nav/>
    <div>
      <div class="container">
        <form @submit.prevent="submit" class="login-form">
          <h1>登录</h1>
          <div v-if="login_failed" class="error-list">
            <div class="error">用户名或密码有误</div>
          </div>
          <div class="input-control">
            <input 
                id="username" 
                v-validator="'required'"
                v-model="current.$unique"
                type="text"
                placeholder="用户名/手机号/邮箱"
                autocomplete="off">
          </div>
          <div class="input-control">
            <input 
                id="password" 
                v-validator="'required'"
                v-model="current.password"
                type="password" 
                placeholder="密码">
          </div>
          <div class="input-control">
            <button class="btn-primary" type="submit">登录</button>
          </div>
          <div class="input-control">
            <p>没有账号？<a class="signup" href="#">注册</a></p>
          </div>
          <div class="well">
            <span>管理员账号：admin</span>
            <span>密码：yoyoyo</span>
          </div>
        </form>
      </div>
    </div>
    <!-- <Footer/> -->
  </div>
</template>

<script>
import "../css/auth.css";

import api from "../lib/api.js";
import session from "../lib/session.js";

import Nav from "../components/Nav";
import Footer from "../components/Footer";

export default {
  //   directives: { validator }
  components: { Nav, Footer },

  data() {
    return {
      current: {},
      login_failed: false
    };
  },

  methods: {
    submit() {
      let unique, password;

      if (
        !(unique = this.current.$unique) ||
        !(password = this.current.password)
      ) {
        return;
      }

      if (unique === "admin" && password === "yoyoyo") {
        this.on_login_succeed({ id: 1, username: "admin", is_admin: true });
        return;
      }

      api("user/read", {
        where: {
          or: [
            ["username", "=", unique],
            ["phone", "=", unique],
            ["e_mail", "=", unique]
          ]
        }
      }).then(r => {
        let row;
        if (!(row = r.data.data[0]) || row.password != password) {
          this.login_failed = true;
          return;
        }
        this.on_login_succeed(row);
      });
    },

    on_login_succeed(row) {
      this.login_failed = false;
      delete row.password;
      session.login(row);
      alert("登录成功");
      if (session.is_admin()) {
        this.$router.push("/admin/user");
      }
      this.$router.push("/");
    }
  }
};
</script>

<style scoped>
.login-form {
  width: 500px;
  text-align: center;
  padding: 10px 20px;
  margin: 50px auto 0;
  background: #f5f5f5;
  border: 1px solid rgba(0, 0, 0, 0.1);
  border-radius: 5px;
}

.login-form input,
.login-form button {
  width: 100%;
  margin-bottom: 13px;
}

.signup {
  color: #555;
}
</style>
