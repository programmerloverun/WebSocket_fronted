<template>
  <div class="wrapper">
    <div class="main-content-agile">
      <div class="login-form">
        <h2 class="title">登录</h2>
        <form @submit.prevent="login">
          <div class="form-group">
            <input placeholder="用户名" id="username" v-model="user.username" type="text" autocomplete required/>
          </div>

          <div class="form-group">
            <input placeholder="密码" id="password" v-model="user.password" type="password" autocomplete required/>
          </div>

          <div class="form-group">
            <button type="submit">登录</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>
import {ref} from 'vue'
import request from '@/util/request.js'
import {ElMessage} from 'element-plus'
import router from '@/router/index.js'

const user = ref({
  username: '',
  password: ''
})

const login = () => {
  request
      .post('/user/login', user.value)
      .then((response) => {
        console.log('登录接口-后台返回的数据：', response)

        if (response.code === 200) {
          ElMessage.success('登录成功')
          router.push('/home')

        } else {
          ElMessage.error(response.message)
        }
      })
      .catch((error) => {
        console.log('登录失败-错误信息：', error)
      })
}
</script>

<style scoped>
.wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background: linear-gradient(135deg, #667eea, #764ba2) repeat;
}

.header-w3l h1 {
  color: #fff;
  font-size: 2.5em;
  font-weight: 600;
}

.main-content-agile {
  width: 100%;
  display: flex;
  justify-content: center;
}

.login-form {
  width: 75%;
  max-width: 800px;
  background-color: #fff;
  padding: 40px;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  margin: auto;
}

.title {
  color: #333;
  text-align: center;
  margin-bottom: 30px;
}

.form-group {
  margin-bottom: 20px;
}

.form-group input {
  width: 96%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
  margin-bottom: 20px;
  height: 30px;
}

.form-group button {
  width: 99%;
  padding: 10px;
  border: none;
  border-radius: 4px;
  background-color: #5a67d8;
  color: #ffffff;
  font-size: 16px;
  cursor: pointer;
  height: 50px;
}
</style>
