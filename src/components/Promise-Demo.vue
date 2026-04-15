<script setup>
import { ref } from 'vue'
import qaqImg from '../assets/QAQ.png'

const status = ref('pending')
const result = ref('还没有开始任务')
const logs = ref([])

function now() {
  return new Date().toLocaleTimeString('zh-CN', { hour12: false })
}

function pushLog(message) {
  logs.value = [...logs.value, `[${now()}] ${message}`]
}

function resetDemo() {
  status.value = 'pending'
  result.value = '还没有开始任务'
  logs.value = []
}

function fakeTask(ms, value, shouldReject = false) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (shouldReject) {
        reject(new Error(String(value)))
        return
      }
      resolve(value)
    }, ms)
  })
}

async function runSuccessFlow() {
  resetDemo()
  pushLog('开始任务：去奶茶店下单（模拟网络请求）')
  pushLog('状态是 pending：正在等待结果...')

  try {
    const drink = await fakeTask(900, '珍珠奶茶')
    status.value = 'fulfilled'
    result.value = `成功拿到：${drink}`
    pushLog('状态变成 fulfilled：任务成功')
    pushLog('then 的作用：拿到成功结果后继续做下一步')
  } catch (error) {
    status.value = 'rejected'
    result.value = `失败：${error.message}`
    pushLog('状态变成 rejected：任务失败')
  }
}

async function runFailFlow() {
  resetDemo()
  pushLog('开始任务：去奶茶店下单（模拟网络请求）')
  pushLog('状态是 pending：正在等待结果...')

  try {
    await fakeTask(900, '门店缺货', true)
  } catch (error) {
    status.value = 'rejected'
    result.value = `失败：${error.message}`
    pushLog('状态变成 rejected：任务失败')
    pushLog('catch 的作用：集中处理错误')
  }
}
</script>

<template>
  <main class="page">
    <section class="hero">
      <p class="eyebrow">PIPERAS Demo</p>
      <h1>Promise 实践学习</h1>
    </section>

    <section class="grid">
      <article class="panel">
        <h2>1) 先看最简单的成功/失败</h2>
        <p>点击按钮后，观察状态怎么变化：pending -> fulfilled / rejected。</p>
        <div class="actions">
          <button @click="runSuccessFlow">运行成功流程</button>
          <button @click="runFailFlow">运行失败流程</button>
          <button class="ghost" @click="resetDemo">重置</button>
        </div>
        <p class="status-line">
          当前状态：
          <span class="state" :class="status">{{ status }}</span>
        </p>
        <p class="result">结果：{{ result }}</p>
        <pre class="log">{{ logs.join('\n') || '点击按钮开始演示...' }}</pre>
        <img :src="qaqImg" alt="QAQ" class="demo-image" />
      </article>

      <article class="panel">
        <h2>2) Promise 怎么用</h2>
        <p>你可以把它理解成：先“登记一个任务”，等结果回来再处理。</p>

        <div class="mini-grid">
          <div class="mini-card">
            <h3>第 1 步：创建</h3>
            <p>new Promise(...) 代表“我要开始一个异步任务”。</p>
          </div>
          <div class="mini-card">
            <h3>第 2 步：成功后做事</h3>
            <p>then(...) 只有在成功时才会执行。</p>
          </div>
          <div class="mini-card">
            <h3>第 3 步：失败时处理</h3>
            <p>catch(...) 专门接住失败原因。</p>
          </div>
          
        </div>

        <div class="snippet-box">
          <p class="snippet-title">一个最小的使用例子</p>
          <pre class="log compact">const drink = new Promise((resolve, reject) => {
  setTimeout(() => resolve('珍珠奶茶'), 1000)
})

drink
  .then((value) => {
    console.log('成功拿到：', value)
  })
  .catch((error) => {
    console.log('失败了：', error)
  })
  .finally(() => {
    console.log('不管成功失败，最后都会执行')
  })</pre>
        </div>

        <ul class="tips simple-list">
          <li>then 是“成功回调”。</li>
          <li>另外：then 可以链式调用，进行多次处理。</li>
          <li>catch 是“失败回调”。</li>
          <li>finally 是“收尾动作”。</li>
          <li>finally 无论成功失败都会执行。但是本身不是必要的，有没有都可以</li>
          <hr>
          <li>推荐文章：<a href="https://juejin.cn/post/7626366335779340351" target="_blank" rel="noopener noreferrer">来自掘金稀土：JS炼化:手写一下promise</a></li>
          <li>推荐文章：<a href="https://zhuanlan.zhihu.com/p/230860803" target="_blank" rel="noopener noreferrer">来自知乎：初学者应该看的JavaScript Promise 完整指南</a></li>
          <hr>
        </ul>
      </article>

      <article class="panel wide">
        <h2>3) Promise 简略原理</h2>
        <p>Promise 的内部可以分成 4 件事来理解：</p>

        <ul class="tips">
          <li>第一步：刚创建时，状态是 pending，表示“还在等”。</li>
          <li>第二步：任务成功时，调用 resolve，状态变成 fulfilled。</li>
          <li>第三步：任务失败时，调用 reject，状态变成 rejected。</li>
          <li>第四步：then 和 catch 并不是“立刻执行”，它们是把后面要做的事先记下来。</li>
        </ul>

        <div class="steps">
          <div class="step">
            <strong>1. 创建</strong>
            <span>先准备一个异步任务。</span>
          </div>
          <div class="step">
            <strong>2. 改状态</strong>
            <span>成功用 resolve，失败用 reject。</span>
          </div>
          <div class="step">
            <strong>3. 交结果</strong>
            <span>then/catch 接收到结果后继续处理。</span>
          </div>
          <div class="step">
            <strong>4. 只改一次</strong>
            <span>状态一旦确定，就不能再反悔。</span>
          </div>
        </div>

        <div class="snippet-box">
          <p class="snippet-title">Promise 内部的最简逻辑</p>
          <pre class="log compact">new Promise((resolve, reject) => {
  // 这里写异步任务
  // 成功就调用 resolve(结果)
  // 失败就调用 reject(原因)
})
  .then((value) => {
    // 成功处理
  })
  .catch((err) => {
    // 失败处理
  })</pre>
        </div>
      </article>
    </section>
  </main>
</template>
