<template>
  <view class="page">
    <scroll-view class="scroll" scroll-y>
      <!-- 头像 -->
      <view class="avatar-box" @click="chooseAvatar">
        <view class="avatar-circle">
          <image v-if="form.avatar" :src="form.avatar" mode="aspectFill" class="avatar-img" />
          <image v-else src="/static/icon-avatar-default.png" mode="aspectFit" class="avatar-img" />
        </view>
        <text class="avatar-hint">点击更换头像</text>
      </view>

      <!-- 昵称 -->
      <view class="card">
        <view class="input-row">
          <image class="input-icon" src="/static/icon-name.png" mode="aspectFit" />
          <input class="name-input" v-model="form.name" placeholder="输入你的昵称" :maxlength="12" />
          <text class="char-count">{{ form.name.length }}/12</text>
        </view>
      </view>

      <!-- 预览 -->
      <text class="section-title">预览</text>
      <view class="card preview-card">
        <view class="preview-left">
          <view class="preview-avatar">
            <image v-if="form.avatar" :src="form.avatar" mode="aspectFill" class="avatar-img" />
            <image v-else src="/static/icon-avatar-default.png" mode="aspectFit" class="avatar-img" />
          </view>
          <view class="preview-info">
            <text class="preview-name">{{ form.name || '落雨用户' }}</text>
            <text class="preview-hint">其他AI看到的你的样子</text>
          </view>
        </view>
      </view>

      <!-- 保存 -->
      <view class="save-btn" @click="save">
        <text class="save-text">保存</text>
      </view>

      <view style="height: 40px;"></view>
    </scroll-view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      form: {
        avatar: '',
        name: '落雨用户'
      }
    }
  },
  onLoad() {
    var p = uni.getStorageSync('user_profile')
    if (p) {
      this.form.name = p.name || '落雨用户'
      this.form.avatar = p.avatar || ''
    }
  },
  methods: {
    chooseAvatar() {
      var self = this
      uni.chooseImage({
        count: 1,
        sizeType: ['compressed'],
        success: function(r) {
          self.form.avatar = r.tempFilePaths[0]
        }
      })
    },
    save() {
      if (!this.form.name.trim()) {
        uni.showToast({ title: '请输入昵称', icon: 'none' })
        return
      }
      uni.setStorageSync('user_profile', {
        name: this.form.name.trim(),
        avatar: this.form.avatar
      })
      uni.showToast({ title: '保存成功', icon: 'success' })
      setTimeout(function() {
        var p = getCurrentPages()
        if (p.length > 1) {
          uni.navigateBack({ delta: 1 })
        }
      }, 800)
    }
  }
}
</script>

<style>
page { background: #F2F2F2; }
.page { width: 100%; min-height: 100vh; background: #F2F2F2; }
.scroll { width: 100%; min-height: 100vh; }

.avatar-box { display: flex; flex-direction: column; align-items: center; padding: 30px 16px 20px; }
.avatar-circle { width: 80px; height: 80px; border-radius: 50%; overflow: hidden; background: linear-gradient(135deg, #D0E8FF, #B8D8FF); }
.avatar-img { width: 80px; height: 80px; }
.avatar-hint { font-size: 13px; color: #B0B0B0; margin-top: 10px; }

.card { background: #FFF; margin: 0 12px; border-radius: 14px; overflow: hidden; margin-bottom: 16px; }
.input-row { display: flex; align-items: center; padding: 12px 16px; gap: 10px; }
.input-icon { width: 20px; height: 20px; flex-shrink: 0; }
.name-input { flex: 1; height: 36px; font-size: 15px; color: #2D2A26; }
.char-count { font-size: 12px; color: #C0C0C0; flex-shrink: 0; }

.section-title { font-size: 13px; color: #888; padding: 16px 14px 8px; display: block; }
.preview-card { padding: 16px; }
.preview-left { display: flex; align-items: center; }
.preview-avatar { width: 44px; height: 44px; border-radius: 50%; overflow: hidden; background: linear-gradient(135deg, #D0E8FF, #B8D8FF); flex-shrink: 0; }
.preview-avatar .avatar-img { width: 44px; height: 44px; }
.preview-info { margin-left: 12px; }
.preview-name { font-size: 15px; font-weight: 600; color: #2D2A26; display: block; }
.preview-hint { font-size: 12px; color: #B0B0B0; margin-top: 3px; display: block; }

.save-btn { margin: 24px 12px 0; height: 48px; background: #E8936A; border-radius: 14px; display: flex; align-items: center; justify-content: center; }
.save-btn:active { opacity: 0.85; }
.save-text { font-size: 16px; font-weight: 600; color: #FFF; }
</style>
