<template>
  <view class="page">
    <scroll-view class="scroll" scroll-y>

      <!-- 用户卡片 -->
      <view class="user-card" @click="goProfile">
        <view class="avatar-box">
          <image v-if="profile.avatar" :src="profile.avatar" mode="aspectFill" class="avatar-img" />
          <view v-else class="avatar-default">
            <text class="avatar-letter">{{ profile.nickname ? profile.nickname[0] : '落' }}</text>
          </view>
        </view>
        <view class="user-info">
          <text class="nickname">{{ profile.nickname || '落雨用户' }}</text>
          <text class="sign">点击设置个人信息</text>
        </view>
        <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
      </view>

      <!-- 设置 -->
      <text class="section-title">设置</text>
      <view class="group">
        <view class="item" @click="goSetting">
          <text class="item-label">更多设置</text>
          <view class="item-right">
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
        <view class="item" @click="goApiSetting">
          <text class="item-label">API设置</text>
          <view class="item-right">
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        
        </view>

      </view>

      <!-- 更多 -->
      <text class="section-title">更多</text>
      <view class="group">
        <view class="item" @click="goPage('devteam')">
          <text class="item-label">开发团队</text>
          <view class="item-right">
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
        <view class="item" @click="goPage('contact')">
          <text class="item-label">联系我们</text>
          <view class="item-right">
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
        <view class="item" @click="goPage('support')">
          <text class="item-label">支持我们</text>
          <view class="item-right">
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
        <view class="item" @click="goPage('agreement')">
          <text class="item-label">用户协议和隐私条款</text>
          <view class="item-right">
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
        <view class="item" @click="goPage('record')">
          <text class="item-label">软件备案</text>
          <view class="item-right">
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
        <view class="item" @click="goAbout">
          <text class="item-label">关于软件</text>
          <view class="item-right">
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
      </view>
	  


      <view style="height: 40px;"></view>
    </scroll-view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      profile: {
        nickname: '',
        signature: '',
        avatar: ''
      }
    }
  },

  onShow() {
    this.loadProfile()
  },

  methods: {

    loadProfile() {
      var d = uni.getStorageSync('user_profile')
      if (d) {
        try {
          this.profile = typeof d === 'string' ? JSON.parse(d) : d
        } catch(e) {}
      }
    },

    goProfile() {
      uni.navigateTo({ url: '/pages/profile/profile' })
    },

    goSetting() {
      uni.navigateTo({ url: '/pages/setting/setting' })
    },

    goApiSetting() {
      uni.navigateTo({ url: '/pages/api-setting/api-setting' })
    },

    goChangelog() {
      uni.navigateTo({ url: '/pages/changelog/changelog' })
    },

    checkUpdate() {
      uni.showToast({ title: '已是最新版本', icon: 'none' })
    },

    goPage(name) {
      uni.navigateTo({ url: '/pages/' + name + '/' + name })
    },

    goAbout() {
      uni.navigateTo({ url: '/pages/about/about' })
    }
  }
}
</script>

<style>
page { background: #F2F2F2; }
.page { width: 100%; min-height: 100vh; background: #F2F2F2; }
.scroll { width: 100%; min-height: 100vh; }

/* 用户卡片 */
.user-card {
  margin: 12px;
  background: #FFF;
  border-radius: 14px;
  padding: 16px;
  display: flex;
  align-items: center;
}
.user-card:active { background: #F8F8F8; }
.avatar-box {
  width: 56px;
  height: 56px;
  border-radius: 50%;
  overflow: hidden;
  flex-shrink: 0;
}
.avatar-img { width: 56px; height: 56px; }
.avatar-default {
  width: 56px;
  height: 56px;
  background: linear-gradient(135deg, #FFE5D0, #FFD4B8);
  display: flex;
  align-items: center;
  justify-content: center;
}
.avatar-letter { font-size: 22px; font-weight: 700; color: #E8936A; }
.user-info { flex: 1; margin-left: 12px; overflow: hidden; }
.nickname {
  font-size: 17px;
  font-weight: 700;
  color: #2D2A26;
  display: block;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
.sign {
  font-size: 13px;
  color: #B0B0B0;
  margin-top: 4px;
  display: block;
}
.arrow-img { width: 16px; height: 16px; }

/* 分区标题 */
.section-title {
  font-size: 13px;
  font-weight: 500;
  color: #888;
  padding: 20px 16px 8px;
  display: block;
}

/* 列表组 */
.group {
  background: #FFF;
  margin: 0 12px;
  border-radius: 14px;
  overflow: hidden;
}
.item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 14px 16px;
}
.item:active { background: #F8F8F8; }
.item + .item { border-top: 0.5px solid #F0F0F0; }
.item-label { font-size: 15px; color: #2D2A26; }
.item-right { display: flex; align-items: center; gap: 4px; }
.item-value { font-size: 13px; color: #B0B0B0; }
</style>
