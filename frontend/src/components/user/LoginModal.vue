<template>
  <div class="modal-container">
    <div class="container">
      <div class="modal" :class="{ show: isOpen }">
        <div class="modal-dialog">
          <div class="modal-content">
            <div class="modal-header">
              <h4 class="modal-title">로그인</h4>
              <button class="close" @click="$emit('onCloseModal')">X</button>
            </div>
            <div class="modal-body">
              <div class="social-login-container">
                <template v-for="social in socials">
                  <a :href="socialLoginUrl(social.socialType)" class="social_btn" :key="social.socialType">
                    <img :src="social.src" :style="{ width: social.width, height: social.height }" alt="" class="social_login" />
                    {{ social.comment }}
                  </a>
                </template>
              </div>
              <div class="or-separator">
                <span class="or-text">OR</span>
              </div>
              <div class="form-wrap">
                <div class="form-item">
                  <input type="text" class="form-control" placeholder="아이디" v-model="id" @input="inputChanged" @keyup.enter="login" />
                </div>
                <div class="form-item">
                  <input type="password" class="form-control" placeholder="비밀번호" v-model="password" @input="inputChanged" @keyup.enter="login" />
                </div>

                <button
                  @click="$emit('onOpenSignUpModal')"
                  class="btn btn-block"
                  style="color: #fff; background-color: #5fc02b; border-color: #5fc02b; font-size: 14px; border-radius: 4px; height: 45px; margin-bottom: 5px"
                >
                  회원가입
                </button>
                <div class="form-item">
                  <button @click="login" class="btn btn-block btn-primary">로그인</button>
                </div>
              </div>
            </div>
            <div class="modal-footer">
              <button class="btn btn-danger" @click="$emit('onCloseModal')">닫기</button>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="modal-backdrop" :class="{ show: isOpen }"></div>
  </div>
</template>

<script scope>
import $ from "@/utils";
import accountApi from "@/api/account";
import { mapActions, mapMutations } from "vuex";

const account = "account";

export default {
  props: ["isOpen"],
  data() {
    return {
      id: "",
      password: "",
      isProcess: false,
      cannotLogin: false,
      isLoginFailed: false,
      socials: [
        {
          socialType: "google",
          src: $.getSocialImage("google"),
          width: "32px",
          height: "32px",
          comment: "구글 로그인",
        },
        {
          socialType: "naver",
          src: $.getSocialImage("naver"),
          width: "32px",
          height: "32px",
          comment: "네이버 로그인",
        },
        {
          socialType: "kakao",
          src: $.getSocialImage("kakao"),
          width: "32px",
          height: "32px",
          comment: "카카오 로그인",
        },
      ],
    };
  },
  methods: {
    ...mapActions(account, ["fetchUser"]),
    ...mapMutations(account, ["setToken"]),
    login() {
      if (this.isProcess) return;
      if (this.id.trim() === "" || this.password.trim() === "") {
        this.cannotLogin = true;
        return;
      }
      accountApi.login(
        {
          id: this.id,
          password: this.password,
          socialType: "LOCAL",
        },
        (body) => {
          this.setToken(body.token);
          this.id = this.password = "";
          this.isProcess = false;
          this.fetchUser(() => {
            this.$emit("onCloseModal");
          });
        },
        (err) => {
          if (err.response.data.status === 401) {
            this.isLoginFailed = true;
          }
        }
      );
    },
    inputChanged() {
      if (!this.cannotLogin) return;
      if (this.id.trim() !== "" && this.password.trim() !== "") {
        this.cannotLogin = false;
      }
    },
    socialLoginUrl(socialType) {
      return $.getSocialLoginUrl(socialType);
    },
  },
};
</script>

<style scoped>
.container {
  width: 100%;
  padding-right: 15px;
  padding-left: 15px;
  margin-right: auto;
  margin-left: auto;
}
.modal-open .modal {
  overflow-x: hidden;
  overflow-y: auto;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 1050;
  display: none;
  width: 100%;
  height: 100%;
  overflow: hidden;
  outline: 0;
}

.modal-dialog {
  position: relative;
  width: auto;
  margin: 0.5rem;
  pointer-events: none;
}

.modal.show .modal-dialog {
  -webkit-transform: none;
  transform: none;
}

.modal-content {
  position: relative;
  display: -ms-flexbox;
  display: -webkit-box;
  display: flex;
  -ms-flex-direction: column;
  -webkit-box-orient: vertical;
  -webkit-box-direction: normal;
  flex-direction: column;
  width: 100%;
  pointer-events: auto;
  background-color: #fff;
  background-clip: padding-box;
  border: 1px solid rgba(0, 0, 0, 0.2);
  border-radius: 0.3rem;
  outline: 0;
}

.modal-header {
  display: -ms-flexbox;
  display: -webkit-box;
  display: flex;
  -ms-flex-align: start;
  -webkit-box-align: start;
  align-items: flex-start;
  -ms-flex-pack: justify;
  -webkit-box-pack: justify;
  justify-content: space-between;
  padding: 1rem 1rem;
  border-bottom: 1px solid #dee2e6;
  border-top-left-radius: calc(0.3rem - 1px);
  border-top-right-radius: calc(0.3rem - 1px);
}

.modal-title {
  margin-bottom: 0;
  line-height: 1.5;
  font-size: 1.5rem;
}

.modal-header .close {
  padding: 1rem 1rem;
  margin: -1rem -1rem -1rem auto;
  cursor: pointer;
}

button.close {
  padding: 0;
  background-color: transparent;
  border: 0;
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
}

.close {
  float: right;
  font-size: 1.5rem;
  font-weight: 700;
  line-height: 1;
  color: #000;
  text-shadow: 0 1px 0 #fff;
  opacity: 0.5;
}

.show {
  display: block;
}
.modal-body > ul > li {
  list-style: none;
}
.social_login {
  height: 32px;
  float: left;
  margin-top: 5.2px;
  margin-left: 10px;
}
.social_btn:hover {
  color: #2098f3;
}
.social_btn {
  margin-bottom: 15px;
  font-weight: 400;
  font-size: 16px;
  display: block;
  width: 100%;
  height: 45px;
  line-height: 45px;
  margin-bottom: 15px;
  border-radius: 4px;
  border: 1px solid #e8e8e8;
  cursor: pointer;
  color: black;
  text-decoration: none;
}
.modal-body {
  padding-left: 2em;
  padding-right: 2em;
  position: relative;
  -ms-flex: 1 1 auto;
  -webkit-box-flex: 1;
  flex: 1 1 auto;
  padding: 1rem;
}

.form-control {
  display: block;
  width: calc(100% - 1.5rem);
  height: calc(1.5em + 0.75rem + 2px);
  padding: 0.375rem 0.75rem;
  font-size: 1rem;
  font-weight: 400;
  line-height: 1.5;
  color: #495057;
  background-color: #fff;
  background-clip: padding-box;
  border: 1px solid #ced4da;
  border-radius: 0.25rem;
  -webkit-transition: border-color 0.15s ease-in-out, -webkit-box-shadow 0.15s ease-in-out;
  transition: border-color 0.15s ease-in-out, -webkit-box-shadow 0.15s ease-in-out;
  transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
  transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out, -webkit-box-shadow 0.15s ease-in-out;
}

.social_login_container {
  text-align: center;
}

.or-text {
  position: absolute;
  left: 46%;
  top: 0;
  background: #fff;
  padding: 10px;
  color: rgba(0, 0, 0, 0.45);
}

.or-separator {
  border-bottom: 1px solid #eee;
  padding: 10px 0;
  position: relative;
  display: block;
  margin-top: 20px;
  margin-bottom: 30px;
  font-size: 1em;
}

.form-item {
  margin-bottom: 18px;
}

.btn {
  display: inline-block;
  font-weight: 400;
  color: #212529;
  text-align: center;
  vertical-align: middle;
  cursor: pointer;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  background-color: transparent;
  border: 1px solid transparent;
  padding: 0.375rem 0.75rem;
  font-size: 1rem;
  line-height: 1.5;
  border-radius: 0.25rem;
  -webkit-transition: color 0.15s ease-in-out, background-color 0.15s ease-in-out, border-color 0.15s ease-in-out, -webkit-box-shadow 0.15s ease-in-out;
  transition: color 0.15s ease-in-out, background-color 0.15s ease-in-out, border-color 0.15s ease-in-out, -webkit-box-shadow 0.15s ease-in-out;
  transition: color 0.15s ease-in-out, background-color 0.15s ease-in-out, border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
  transition: color 0.15s ease-in-out, background-color 0.15s ease-in-out, border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out, -webkit-box-shadow 0.15s ease-in-out;
}

.login-error {
  display: none;
  color: red;
}

.login-error.show {
  display: block;
}

.btn-block {
  display: block;
  width: 100%;
}

.btn-primary {
  color: #fff;
  background-color: #2098f3;
  border-color: #2098f3;
  font-size: 14px;
  border-radius: 4px;
  height: 45px;
}

.btn-danger {
  color: #fff;
  background-color: #dc3545;
  border-color: #dc3545;
}

.btn-primary.disabled {
  background-color: #a0d6ff;
  border-color: #a0d6ff;
}

.modal-backdrop.show {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 1040;
  width: 100vw;
  height: 100vh;
  background-color: #000;
  opacity: 0.5;
}

.modal-footer {
  display: -ms-flexbox;
  display: -webkit-box;
  display: flex;
  -ms-flex-wrap: wrap;
  flex-wrap: wrap;
  -ms-flex-align: center;
  -webkit-box-align: center;
  align-items: center;
  -ms-flex-pack: end;
  -webkit-box-pack: end;
  justify-content: flex-end;
  padding: 0.75rem;
  border-top: 1px solid #dee2e6;
  border-bottom-right-radius: calc(0.3rem - 1px);
  border-bottom-left-radius: calc(0.3rem - 1px);
}

@media screen and (min-width: 768px) {
  .container {
    max-width: 720px;
  }
}
@media screen and (min-width: 576px) {
  .container {
    max-width: 540px;
  }
  .modal-dialog {
    max-width: 500px;
    margin: 1.75rem auto;
  }
}
</style>
