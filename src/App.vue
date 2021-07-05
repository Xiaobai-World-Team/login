<template>
 <div class="xiaobai-world-login-wrapper" :class="{ loading }">
  <template v-if="!user.email">
   <div class="tabs">
    <a
     :class="{ active: activeIndex === 0 }"
     @click="activeIndex = 0"
     href="javascript:void(0)"
     >Login</a
    >
    <a
     :class="{ active: activeIndex === 1 }"
     @click="activeIndex = 1"
     href="javascript:void(0)"
     >Create your account</a
    >
   </div>
   <div>
    <p class="error-message" v-if="errorMessage">{{ errorMessage }}</p>
   </div>
   <form v-if="activeIndex === 0" @submit.prevent="login">
    <fieldset :disabled="pending">
     <div class="item">
      <label for="email"> Email </label>
      <input name="email" v-model="form.email" type="text" />
     </div>
     <div class="item">
      <label>Password</label>
      <input name="password" v-model="form.password" type="password" />
     </div>
     <div class="control">
      <button type="submit">Login</button>
     </div>
    </fieldset>
   </form>
   <form v-if="activeIndex === 1" @submit.prevent="register">
    <fieldset :disabled="pending">
     <div class="item">
      <label for="email"> Email </label>
      <input name="email" v-model="form.email" type="text" />
     </div>
     <div class="item">
      <label>Password</label>
      <input name="password" v-model="form.password" type="password" />
     </div>
     <div class="item">
      <label>Repeat Password</label>
      <input
       name="repeatPassword"
       v-model="form.repeatPassword"
       type="password"
      />
     </div>
     <div class="control">
      <button type="submit">Register</button>
     </div>
    </fieldset>
   </form>
  </template>
  <div v-show="user.email" class="login-success">
   <img v-if="user.avatar" :src="user.avatar" />
   <h1>{{ user.email }}</h1>
   <p class="msg">Signed in ✅! </p>
  </div>
 </div>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";
import { IXiaobaiUser } from "@xiaobai-world/api";

export default defineComponent({
 name: "XiaobaiWorldLogin",
 setup() {
  const form = ref({
   email: "",
   password: "",
   repeatPassword: "",
  });

  const user = ref<IXiaobaiUser>({
   email: "",
   avatar: "",
  });

  const errorMessage = ref("");

  const pending = ref(false);

  function login() {
   pending.value = true;
   fetch("/user/login", {
    body: JSON.stringify({
     email: form.value.email,
     password: form.value.password,
    }),
    headers: {
     "Content-Type": "application/json",
    },
    method: "POST",
   })
    .then((response) => response.json())
    .then((data) => {
     pending.value = false;
     if (data.email) {
      user.value = data;
      errorMessage.value = "";
     } else {
      errorMessage.value = Array.isArray(data.message)
       ? data.message.join(",")
       : data.message;
     }
    })
    .catch((e) => {
     console.log(e);
    });
  }

  function register() {
   fetch("/user/register", {
    body: JSON.stringify(form.value),
    headers: {
     "Content-Type": "application/json",
    },
    method: "POST",
   })
    .then((response) => response.json())
    .then((data) => {
     if (data?.statusCode === 500) {
      errorMessage.value = data.message;
     }
    })
    .catch((e) => {
     console.log("eee", e);
    });
  }

  const activeIndex = ref(0);
  const loading = ref(true);

  return {
   login,
   activeIndex,
   form,
   register,
   user,
   errorMessage,
   pending,
   loading,
  };
 },
 mounted() {
  fetch("/user/info")
   .then((res) => res.json())
   .then((data) => {
    this.loading = false;
    if (data.email) {
     this.user = data;
    }
   })
   .catch((e) => {
    this.loading = false;
   });
 },
});
</script>

<style lang="less">
.xiaobai-world-login-wrapper {
 max-width: 380px;
 margin: 0 auto;
 font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
  Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
 &.loading {
  background: url(./loading.svg) center center no-repeat;
  > * {
   opacity: 0;
  }
 }
 form {
  fieldset {
   margin: 0;
   padding: 0;
   border: none;

   > .item {
    display: flex;
    padding: 0.5em 0;
    align-items: center;
    > label {
     flex: 0 0 120px;
     text-align: right;
     padding-right: 1em;
    }
    input {
     padding: 0.5em;
     flex: 1 1 0;
     border: solid 1px #aaa;
     border-radius: 0.2em;
    }
   }
   .control {
    text-align: right;
    button {
     padding: 0.5em 1em;
    }
   }
  }
 }
 .tabs {
  display: flex;
  justify-content: center;
  border-bottom: solid 2px #eee;
  margin-bottom: 1em;
  align-items: center;
  a {
   font-size: 120%;
   padding: 0.5em 1em;
   text-decoration: none;
   color: #333;
   &.active {
    font-size: 150%;
    font-weight: bold;
   }
  }
 }
 .error-message {
  text-align: center;
  background: rgb(218, 67, 67);
  color: #fff;
  padding: 0.5em;
  border-radius: 0.5em;
 }
 .login-success {
  text-align: center;
  h1 {
   font-size: 120%;
   font-weight: normal;
  }
  .msg {
   color: rgb(54, 128, 72);
  }
 }
}
</style>