<template>
  <view class="page">
    <scroll-view class="scroll" scroll-y>
      <view class="about-header">
        <image class="about-logo" src="/static/logo.png" mode="aspectFit" />
        <text class="about-name">落雨</text>
        <text class="about-ver">v1.0.0</text>
        <text class="about-slogan">懂你的AI伴侣</text>
      </view>

      <view class="card-group">
        <view class="info-item" @click="navigate('changelog')">
          <text class="info-label">更新说明</text>
          <view class="info-right">
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
        <view class="info-item" @click="navigate('team')">
          <text class="info-label">开发团队</text>
          <view class="info-right">
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
        <view class="info-item" @click="navigate('thanks')">
          <text class="info-label">特别鸣谢</text>
          <view class="info-right">
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
        <view class="info-item" @click="navigate('agreement')">
          <text class="info-label">用户协议</text>
          <view class="info-right">
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
        <view class="info-item" @click="navigate('filing')">
          <text class="info-label">备案信息</text>
          <view class="info-right">
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
      </view>

      <view class="about-desc">
        <text class="desc-text">落雨是一款AI伴侣应用，让你拥有一个懂你、记住你、陪伴你的AI伙伴。支持多种大模型接入，自定义AI人设，AI自动记忆等功能。</text>
      </view>

      <view class="footer">
        <text class="footer-line">鲁ICP备2025170504号-1A</text>
        <text class="footer-line">Copyright © 2026 落雨</text>
      </view>
      <view style="height: 30px;"></view>
    </scroll-view>
  </view>
</template>

<script>
export default {
  methods: {
    navigate(page) {
      var map = {
        changelog: '/pages/changelog/changelog',
        team: '/pages/team/team',
        thanks: '/pages/thanks/thanks',
        agreement: '/pages/agreement/agreement',
        filing: '/pages/filing/filing'
      }
      if (map[page]) uni.navigateTo({ url: map[page] })
    }
  }
}
</script>

<style>
page { background: #F2F2F2; }
.page { width: 100%; min-height: 100vh; background: #F2F2F2; }
.scroll { width: 100%; min-height: 100vh; }

.about-header { display: flex; flex-direction: column; align-items: center; padding: 30px 16px 20px; }
.about-logo { width: 80px; height: 80px; border-radius: 20px; }
.about-name { font-size: 22px; font-weight: 700; color: #2D2A26; margin-top: 12px; }
.about-ver { font-size: 13px; color: #B0B0B0; margin-top: 4px; }
.about-slogan { font-size: 14px; color: #E8936A; margin-top: 6px; }

.card-group { background: #FFF; margin: 12px; border-radius: 14px; overflow: hidden; }
.info-item { display: flex; align-items: center; justify-content: space-between; padding: 14px 16px; }
.info-item + .info-item { border-top: 0.5px solid #F0F0F0; }
.info-item:active { background: #F8F8F8; }
.info-label { font-size: 15px; color: #2D2A26; }
.info-right { display: flex; align-items: center; }
.arrow-img { width: 16px; height: 16px; }

.about-desc { background: #FFF; margin: 0 12px; border-radius: 14px; padding: 16px; }
.desc-text { font-size: 14px; color: #888; line-height: 1.7; }

.footer { text-align: center; padding: 30px 0 0; }
.footer-line { font-size: 11px; color: #C0C0C0; line-height: 1.6; display: block; }
</style>
