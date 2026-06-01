<template>
  <view class="page">
    <scroll-view class="scroll" scroll-y>
      <view class="thanks-header">
        <image class="thanks-icon-img" src="/static/icon-thanks-big.png" mode="aspectFit" />
        <text class="thanks-title">特别鸣谢</text>
        <text class="thanks-desc">感谢以下项目和个人的贡献</text>
      </view>

      <view class="section-title"><text>贡献者</text></view>
      <view class="card-group">
        <view class="card-item">
          <view class="card-left">
            <view class="card-avatar" style="background: linear-gradient(135deg, #D0E8FF, #B8D8FF);">
              <image class="card-avatar-img" src="/static/avatar-test.png" mode="aspectFit" />
            </view>
            <view class="card-info">
              <text class="card-name">闻风</text>
              <text class="card-desc">资金赞助</text>
            </view>
          </view>
        </view>
      </view>

      <view class="section-title"><text>技术开源</text></view>
      <view class="card-group">
        <view class="empty-box">
          <text class="empty-text">暂无</text>
        </view>
      </view>

      <view class="section-title"><text>测试用户</text></view>
      <view class="card-group">
        <view class="card-item" v-for="(u, i) in testers" :key="i">
          <view class="card-left">
            <text class="card-emoji">{{ u.emoji }}</text>
            <view class="card-info">
              <text class="card-name">{{ u.name }}</text>
              <text class="card-desc">{{ u.desc }}</text>
            </view>
          </view>
        </view>
      </view>

      <view class="footer">
        <text class="footer-text">感谢每一个支持落雨的人</text>
      </view>
      <view style="height: 30px;"></view>
    </scroll-view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      testers: [
        { emoji: '🌟', name: '内测用户', desc: '感谢所有内测用户的反馈与建议' }
      ]
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

.thanks-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 30px 16px 20px;
}

.thanks-icon-img {
  width: 56px;
  height: 56px;
}

.thanks-title {
  font-size: 20px;
  font-weight: 700;
  color: #2D2A26;
  margin-top: 10px;
}

.thanks-desc {
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
  padding: 16px;
}

.card-item + .card-item {
  border-top: 0.5px solid #F0F0F0;
}

.card-left {
  display: flex;
  align-items: center;
  gap: 12px;
}

.card-avatar {
  width: 48px;
  height: 48px;
  border-radius: 14px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.card-avatar-img {
  width: 48px;
  height: 48px;
}

.card-emoji {
  font-size: 24px;
  flex-shrink: 0;
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

.empty-box {
  padding: 30px 16px;
  text-align: center;
}

.empty-text {
  font-size: 14px;
  color: #C0C0C0;
}

.footer {
  text-align: center;
  padding: 30px 0 0;
}

.footer-text {
  font-size: 12px;
  color: #C0C0C0;
}
</style>
