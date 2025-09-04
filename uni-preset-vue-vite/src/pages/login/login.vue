<template>
  <view class="page">
    <view class="card">
      <text class="title">账号密码登录</text>
      <form class="form" @submit.prevent>
        <view class="field">
          <text class="label">账号</text>
          <input class="input" type="text" placeholder="请输入账号" v-model.trim="username" />
        </view>
        <view class="field">
          <text class="label">密码</text>
          <input class="input" type="password" placeholder="请输入密码" v-model.trim="password" />
        </view>
        <button class="btn" @click="onLogin">登录</button>
        </form>
        <view class="foot">
          <text class="foot-text">没有账号？</text>
          <view class="link" @click="goRegister" role="button">注册</view>
        </view>
        <text class="tip">{{tipText}}</text>
    </view>
  </view>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const username = ref('')
const password = ref('')
const tipText = ref('请输入账号和密码进行登录。')

const onLogin = () => {
  if (!username.value || !password.value) {
    tipText.value = '账号和密码不能为空。'
    return
  }
  // 模拟校验
  if (username.value === 'admin' && password.value === '123456') {
    tipText.value = '登录成功，正在跳转...'
    // 使用 uni API 在真实 uni-app 环境中跳转
    if (typeof uni !== 'undefined' && uni.navigateTo) {
      uni.showToast({ title: '登录成功', icon: 'success' })
      setTimeout(() => {
        uni.reLaunch({ url: '/pages/memo/memo' })
      }, 600)
    } else {
      // H5 dev preview fallback：尝试用 location
      setTimeout(() => {
        tipText.value = '模拟跳转：请在 uni-app 中体验真实跳转。'
      }, 600)
    }
  } else {
    tipText.value = '账号或密码错误，请重试。'
  }
}

const goRegister = () => {
  console.log('goRegister clicked')
  if (typeof uni !== 'undefined' && uni.navigateTo) {
    uni.navigateTo({ url: '/pages/register/register' })
    return
  }
  // H5 fallback: try to change hash to uni-app style route
  try {
    window.location.hash = '#/pages/register/register'
    tipText.value = '已尝试 H5 跳转，请查看地址栏或刷新。'
  } catch (e) {
    tipText.value = '跳转到注册页面（H5 模拟）'
  }
}
</script>

<style scoped>
  .page {
    min-height: 100vh;
    padding: 24px 12px;
    box-sizing: border-box;
    background: linear-gradient(135deg, #f8fafc 0%, #e9eeff 100%);
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  .card {
    width: 92vw;
    max-width: 360px;
    background: #fff;
    border-radius: 12px;
    padding: 18px;
    box-shadow: 0 6px 20px rgba(60,80,180,0.06);
    box-sizing: border-box;
    margin-top: 24px;
  }
  .title {
    display: block;
    text-align: center;
    color: #3b4cca;
    font-size: 18px;
    font-weight: 700;
    margin-bottom: 14px;
  }
  .field { margin-bottom: 10px }
  .label { color: #6b7280; font-size: 13px; margin-bottom: 6px; display:block }
  .input {
    width: 100%;
    height: 36px;
    padding: 6px 10px;
    border-radius: 6px;
    border: 1px solid #e6e9f2;
    background: #fff;
    box-sizing: border-box;
    font-size: 14px;
  }
  .input:focus { outline: none }
  .btn {
    width: 100%;
    height: 42px;
    margin-top: 8px;
    border-radius: 8px;
    background: linear-gradient(90deg,#3b4cca 0%,#6366f1 100%);
    color: #fff;
    font-weight: 600;
    border: none;
  }
  .tip { display:block; text-align:center; color:#9ca3af; font-size:13px; margin-top:10px }
  .foot { display:flex; justify-content:center; align-items:center; gap:6px; margin-top:8px }
  .foot-text { color:#6b7280; font-size:13px }
  .link { color:#4f47e6; font-size:13px; font-weight:600 }
  .design { max-width:360px; padding:12px 8px; margin-top:18px; color:#374151 }
  .design-title { font-weight:700; margin-bottom:6px; display:block }
  .design-item { display:block; font-size:13px; line-height:1.5; margin-bottom:4px }

  @media (min-width: 420px) {
    .card { margin-top: 40px }
  }
</style>
