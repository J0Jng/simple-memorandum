<template>
  <view class="page">
    <view class="card">
      <text class="title">账号注册</text>
      <form class="form" @submit.prevent>
        <view class="field">
          <text class="label">账号</text>
          <input class="input" type="text" placeholder="请设置账号" v-model.trim="username" />
        </view>
        <view class="field">
          <text class="label">密码</text>
          <input class="input" type="password" placeholder="请设置密码" v-model.trim="password" />
        </view>
        <button class="btn" @click="onRegister">注册</button>
      </form>
      <text class="tip">{{tipText}}</text>
    </view>
  </view>
</template>

<script setup>
import { ref } from 'vue'

const username = ref('')
const password = ref('')
const tipText = ref('请设置账号和密码')

const onRegister = () => {
  if (!username.value || !password.value) {
    tipText.value = '账号和密码不能为空。'
    return
  }
  // 简单模拟注册成功后跳回登录
  tipText.value = '注册成功，正在返回登录...'
  if (typeof uni !== 'undefined' && uni.navigateBack) {
    uni.showToast({ title: '注册成功', icon: 'success' })
    setTimeout(() => {
      uni.navigateBack()
    }, 700)
  } else {
    setTimeout(() => {
      tipText.value = '注册成功（H5 模拟），请返回登录页登录。'
    }, 700)
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

  @media (min-width: 420px) {
    .card { margin-top: 40px }
  }
</style>
