<template>
  <view class="page">
    <scroll-view class="scroll" scroll-y>
      <view class="service-header">
        <image class="service-icon-img" src="/static/icon-service-big.png" mode="aspectFit" />
        <text class="service-title">联系我们</text>
        <text class="service-desc">有任何问题或建议，欢迎联系我们</text>
      </view>

      <view class="section-title"><text>联系方式</text></view>
      <view class="card-group">
        <view class="card-item" v-for="(c, i) in contacts" :key="i" @click="handleContact(c)">
          <view class="card-left">
            <view class="card-icon-box" :style="{ background: c.bg }">
              <image class="card-icon-img" :src="c.icon" mode="aspectFit" />
            </view>
            <view class="card-info">
              <text class="card-name">{{ c.name }}</text>
              <text class="card-desc">{{ c.desc }}</text>
            </view>
          </view>
          <view class="card-right">
            <text class="card-value">{{ c.value }}</text>
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
      </view>

      <view class="section-title"><text>常见问题</text></view>
      <view class="card-group">
        <view class="faq-item" v-for="(f, i) in faqs" :key="i" @click="toggleFaq(i)">
          <view class="faq-header">
            <text class="faq-q">{{ f.q }}</text>
            <text class="faq-arrow" :class="{ open: f.open }">›</text>
          </view>
          <view class="faq-body" v-if="f.open">
            <text class="faq-a">{{ f.a }}</text>
            <text class="faq-link" v-if="f.link" @click.stop="openLink(f.link)">{{ f.linkText }}</text>
          </view>
        </view>
      </view>

      <view class="footer">
        <text class="footer-text">我们会在24小时内回复您</text>
      </view>
      <view style="height: 30px;"></view>
    </scroll-view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      contacts: [
        { icon: '/static/icon-email.png', name: '邮箱', desc: '发送邮件联系我们', value: '2353960655@qq.com', link: 'mailto:2353960655@qq.com', bg: '#E3F2FD' },
        { icon: '/static/icon-wechat.png', name: '微信公众号', desc: '关注公众号获取帮助', value: '落雨LorianAI', link: '', bg: '#E8F5E9' },
        { icon: '/static/icon-website.png', name: '官方网站', desc: '访问官网了解更多', value: 'www.luoyeltd.com.cn', link: 'https://www.luoyeltd.com.cn', bg: '#FFF3E0' }
      ],
      faqs: [
        { q: '如何配置API？', a: '在"我的"页面点击API设置，选择你使用的模型，填入API Key即可。', link: 'https://www.yuque.com/qingtianlive/ph9vbm/sei28lcv9meup9l0', linkText: '参考API接入文档', open: false },
        { q: 'AI回复很慢怎么办？', a: '请检查网络连接，或者尝试更换其他模型。部分模型在高峰期可能会较慢。', link: '', linkText: '', open: false },
        { q: '聊天记录会丢失吗？', a: '聊天记录保存在本地，请不要清除应用数据。建议定期使用导出功能备份数据。', link: '', linkText: '', open: false },
        { q: '如何删除AI伴侣？', a: '在消息列表长按AI头像，选择删除即可。', link: '', linkText: '', open: false }
      ]
    }
  },
  methods: {
    toggleFaq(i) {
      this.faqs[i].open = !this.faqs[i].open
    },
    handleContact(c) {
      if (c.link) {
        // #ifdef APP-PLUS
        plus.runtime.openURL(c.link)
        // #endif
        // #ifdef H5
        window.open(c.link, '_blank')
        // #endif
      } else {
        uni.setClipboardData({ data: c.value, success: function() { uni.showToast({ title: '已复制', icon: 'success' }) } })
      }
    },
    openLink(url) {
      // #ifdef APP-PLUS
      plus.runtime.openURL(url)
      // #endif
      // #ifdef H5
      window.open(url, '_blank')
      // #endif
      // #ifdef MP
      uni.setClipboardData({ data: url, success: function() { uni.showToast({ title: '链接已复制，请在浏览器打开', icon: 'none' }) } })
      // #endif
    }
  }
}
</script>

<style>
page {
  background: #F2F2F2;
}

.page {
  width: 100%;
  min-height: 100vh;
  background: #F2F2F2;
}

.scroll {
  width: 100%;
  min-height: 100vh;
}

.service-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 30px 16px 20px;
}

.service-icon-img {
  width: 56px;
  height: 56px;
}

.service-title {
  font-size: 20px;
  font-weight: 700;
  color: #2D2A26;
  margin-top: 10px;
}

.service-desc {
  font-size: 13px;
  color: #B0B0B0;
  margin-top: 6px;
}

.section-title {
  padding: 20px 16px 8px;
}

.section-title text {
  font-size: 13px;
  color: #888;
}

.card-group {
  background: #FFF;
  margin: 0 12px;
  border-radius: 14px;
  overflow: hidden;
}

.card-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px;
}

.card-item + .card-item {
  border-top: 0.5px solid #F0F0F0;
}

.card-item:active {
  background: #F8F8F8;
}

.card-left {
  display: flex;
  align-items: center;
  gap: 12px;
}

.card-icon-box {
  width: 40px;
  height: 40px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.card-icon-img {
  width: 22px;
  height: 22px;
}

.card-info {
  flex: 1;
}

.card-name {
  font-size: 15px;
  font-weight: 600;
  color: #2D2A26;
  display: block;
}

.card-desc {
  font-size: 12px;
  color: #B0B0B0;
  margin-top: 3px;
  display: block;
}

.card-right {
  display: flex;
  align-items: center;
  gap: 6px;
}

.card-value {
  font-size: 13px;
  color: #E8936A;
}

.arrow-img {
  width: 16px;
  height: 16px;
  flex-shrink: 0;
}

/* 常见问题 - 无图标 */
.faq-item {
  padding: 0 16px;
}

.faq-item + .faq-item {
  border-top: 0.5px solid #F0F0F0;
}

.faq-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 14px 0;
}

.faq-header:active {
  background: #F8F8F8;
}

.faq-q {
  font-size: 15px;
  color: #2D2A26;
  flex: 1;
}

.faq-arrow {
  font-size: 20px;
  color: #CCC;
  transition: transform 0.25s ease;
  flex-shrink: 0;
}

.faq-arrow.open {
  transform: rotate(90deg);
}

.faq-body {
  padding: 0 0 16px 0;
  animation: fadeIn 0.2s ease;
}

.faq-a {
  font-size: 13px;
  color: #888;
  line-height: 1.6;
  display: block;
}

.faq-link {
  font-size: 13px;
  color: #E8936A;
  font-weight: 600;
  margin-top: 8px;
  display: block;
  text-decoration: underline;
}

.footer {
  text-align: center;
  padding: 30px 0 0;
}

.footer-text {
  font-size: 12px;
  color: #C0C0C0;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
</style>
