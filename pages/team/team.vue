<template>
  <view class="page">
    <scroll-view class="scroll" scroll-y>
      <view class="team-header">
        <image class="team-logo" src="/static/logo.png" mode="aspectFit" />
        <text class="team-name">落雨开发团队</text>
        <text class="team-slogan">用心做好每一个细节</text>
      </view>

      <view class="section-title"><text>核心成员</text></view>
      <view class="member-group">
        <view class="member-card" v-for="(m, i) in members" :key="i">
          <view class="member-avatar" :style="{ background: m.bg }">
            <image class="member-avatar-img" :src="m.avatar" mode="aspectFit" />
          </view>
          <view class="member-info">
            <text class="member-name">{{ m.name }}</text>
            <text class="member-role">{{ m.role }}</text>
            <text class="member-desc">{{ m.desc }}</text>
          </view>
        </view>
      </view>

      <view class="section-title"><text>团队介绍</text></view>
      <view class="about-card">
        <text class="about-text">落雨是一款专注于AI伴侣体验的应用，我们致力于让每个人都能拥有一个懂自己的AI伙伴。</text>
        <text class="about-text">团队成立于2026年，由一群热爱AI技术的年轻人组成，希望通过技术让陪伴不再孤独。</text>
      </view>

      <view class="footer">
        <text class="footer-text">Copyright © 2026 落雨</text>
      </view>
      <view style="height: 30px;"></view>
    </scroll-view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      members: [
        { avatar: '/static/avatar-dev.png', name: '面包bread.', role: '落雨全栈开发者', desc: 'QQ: 2353960655', bg: 'linear-gradient(135deg, #D0E8FF, #B8D8FF)' },
        { avatar: '/static/avatar-design.png', name: '面包²', role: '技术工程师', desc: 'QQ: 535902116', bg: 'linear-gradient(135deg, #FFE5D0, #FFD4B8)' },
        { avatar: '/static/avatar-front.png', name: 'Vincent', role: '功能建议提议员', desc: 'QQ: 不明', bg: 'linear-gradient(135deg, #D5F5E3, #B8E8D0)' },
        { avatar: '/static/avatar-test.png', name: '闻风', role: '问题反馈处理员', desc: 'QQ: 2241719172', bg: 'linear-gradient(135deg, #F5D0FF, #E8B8FF)' }
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

.team-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 30px 16px 20px;
}

.team-logo {
  width: 72px;
  height: 72px;
  border-radius: 18px;
}

.team-name {
  font-size: 20px;
  font-weight: 700;
  color: #2D2A26;
  margin-top: 12px;
}

.team-slogan {
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

.member-group {
  background: #FFF;
  margin: 0 12px;
  border-radius: 14px;
  overflow: hidden;
}

.member-card {
  display: flex;
  align-items: center;
  padding: 16px;
}

.member-card + .member-card {
  border-top: 0.5px solid #F0F0F0;
}

.member-avatar {
  width: 52px;
  height: 52px;
  border-radius: 16px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.member-avatar-img {
  width: 52px;
  height: 52px;
}

.member-info {
  flex: 1;
  margin-left: 14px;
}

.member-name {
  font-size: 16px;
  font-weight: 600;
  color: #2D2A26;
  display: block;
}

.member-role {
  font-size: 12px;
  color: #E8936A;
  margin-top: 2px;
  display: block;
}

.member-desc {
  font-size: 12px;
  color: #B0B0B0;
  margin-top: 3px;
  display: block;
}

.about-card {
  background: #FFF;
  margin: 0 12px;
  border-radius: 14px;
  padding: 16px;
}

.about-text {
  font-size: 14px;
  color: #555;
  line-height: 1.7;
  display: block;
  margin-bottom: 8px;
}

.about-text:last-child {
  margin-bottom: 0;
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

