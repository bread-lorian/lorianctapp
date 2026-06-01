<template>
  <view class="page">
    <scroll-view class="scroll" scroll-y>

      <view class="preview" :style="previewStyle">
        <view class="preview-bubble">
          <text class="preview-text">预览效果</text>
        </view>
      </view>

      <text class="section-title">纯色背景</text>
      <view class="color-grid">
        <view class="color-item" v-for="c in colors" :key="c.value" @click="selectColor(c.value)">
          <view class="color-circle" :style="{ background: c.value }">
            <view class="color-check" v-if="currentType === 'color' && currentValue === c.value">
              <text class="color-check-icon">✓</text>
            </view>
          </view>
          <text class="color-name">{{ c.name }}</text>
        </view>
      </view>

      <text class="section-title">自定义颜色</text>
      <view class="custom-box">
        <view class="custom-colors">
          <view class="custom-color-item" v-for="cc in customColors" :key="cc" @click="selectColor(cc)">
            <view class="custom-color-block" :style="{ background: cc }">
              <view class="color-check" v-if="currentType === 'color' && currentValue === cc">
                <text class="color-check-icon">✓</text>
              </view>
            </view>
          </view>
        </view>
        <view class="custom-input-row">
          <text class="custom-label">HEX色值</text>
          <input class="custom-input" v-model="customHex" placeholder="#FF6B6B" @confirm="applyCustomHex" />
          <view class="custom-apply-btn" @click="applyCustomHex">
            <text class="custom-apply-text">应用</text>
          </view>
        </view>
      </view>

      <text class="section-title">内置背景</text>
      <view class="bg-grid">
        <view class="bg-item" v-for="b in builtInBgs" :key="b.id" @click="selectBuiltIn(b)">
          <image class="bg-img" :src="b.path" mode="aspectFill" />
          <view class="bg-check" v-if="currentType === 'image' && currentValue === b.path">
            <text class="bg-check-icon">✓</text>
          </view>
        </view>
      </view>

      <text class="section-title">自定义图片</text>
      <view class="custom-img-box">
        <view class="custom-img-btn" @click="chooseImage">
          <text class="custom-img-icon">+</text>
          <text class="custom-img-hint">从相册选择</text>
        </view>
      </view>

      <view class="reset-btn" @click="resetBg">
        <text class="reset-text">恢复默认背景</text>
      </view>

      <view style="height: 40px;"></view>
    </scroll-view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      aiId: '',
      currentType: 'default',
      currentValue: '',
      customHex: '',
      colors: [
        { name: '象牙白', value: '#FFFFF0' },
        { name: '浅灰', value: '#F5F5F5' },
        { name: '米色', value: '#FAF0E6' },
        { name: '淡粉', value: '#FFF0F0' },
        { name: '淡蓝', value: '#F0F5FF' },
        { name: '淡绿', value: '#F0FFF0' },
        { name: '暖橙', value: '#FFF5E8' },
        { name: '淡紫', value: '#F5F0FF' }
      ],
      customColors: [
        '#FFD4B8', '#FFB8B8', '#FFE8B8', '#B8E8D0',
        '#B8D8FF', '#D0B8FF', '#FFB8E8', '#B8FFF0'
      ],
      builtInBgs: [
        { id: 'bg1', path: '/static/chat-bg/bg1.png' },
        { id: 'bg8', path: '/static/chat-bg/bg8.png' }
      ]
    }
  },
  computed: {
    previewStyle() {
      if (this.currentType === 'image' && this.currentValue) {
        return { backgroundImage: 'url(' + this.currentValue + ')', backgroundSize: 'cover', backgroundPosition: 'center' }
      } else if (this.currentType === 'color' && this.currentValue) {
        return { background: this.currentValue }
      }
      return { background: '#F5F5F5' }
    }
  },
  onLoad(opts) {
    this.aiId = opts.id || ''
    this.loadBg()
  },
  methods: {
    loadBg() {
      var d = uni.getStorageSync('chat_bg_' + this.aiId)
      if (d) {
        try {
          var bg = typeof d === 'string' ? JSON.parse(d) : d
          if (bg.type) { this.currentType = bg.type; this.currentValue = bg.value || '' }
        } catch(e) {}
      }
    },
    saveBg(type, value) {
      this.currentType = type
      this.currentValue = value
      uni.setStorageSync('chat_bg_' + this.aiId, JSON.stringify({ type: type, value: value }))
      uni.showToast({ title: '背景已更换', icon: 'none' })
    },
    selectColor(value) { this.saveBg('color', value) },
    applyCustomHex() {
      var hex = this.customHex.trim()
      if (!hex) return
      if (hex.charAt(0) !== '#') hex = '#' + hex
      if (!/^#[0-9A-Fa-f]{3,8}$/.test(hex)) { uni.showToast({ title: '颜色格式不正确', icon: 'none' }); return }
      this.saveBg('color', hex)
    },
    selectBuiltIn(b) { this.saveBg('image', b.path) },
    chooseImage() {
      var self = this
      uni.chooseImage({ count: 1, sizeType: ['compressed'], success: function(r) { self.saveBg('image', r.tempFilePaths[0]) } })
    },
    resetBg() { this.saveBg('default', '') }
  }
}
</script>

<style>
page { background: #F2F2F2; }
.page { width: 100%; min-height: 100vh; background: #F2F2F2; }
.scroll { width: 100%; min-height: 100vh; }
.preview { margin: 12px; height: 140px; border-radius: 14px; display: flex; align-items: flex-end; justify-content: flex-end; padding: 16px; background: #F5F5F5; }
.preview-bubble { background: #2D2A26; border-radius: 14px 4px 14px 14px; padding: 8px 14px; }
.preview-text { font-size: 13px; color: #FFF; }
.section-title { font-size: 13px; font-weight: 500; color: #888; padding: 16px 16px 8px; display: block; }
.color-grid { display: flex; flex-wrap: wrap; background: #FFF; margin: 0 12px; border-radius: 14px; padding: 12px; gap: 12px; }
.color-item { width: calc(25% - 9px); display: flex; flex-direction: column; align-items: center; gap: 4px; }
.color-circle { width: 48px; height: 48px; border-radius: 50%; border: 2px solid #E8E8E8; display: flex; align-items: center; justify-content: center; position: relative; }
.color-name { font-size: 11px; color: #999; }
.color-check { width: 20px; height: 20px; border-radius: 10px; background: #E8936A; display: flex; align-items: center; justify-content: center; position: absolute; bottom: -2px; right: -2px; }
.color-check-icon { font-size: 12px; color: #FFF; font-weight: 700; }
.custom-box { background: #FFF; margin: 0 12px; border-radius: 14px; padding: 12px; }
.custom-colors { display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px; }
.custom-color-item { width: calc(12.5% - 7px); }
.custom-color-block { width: 100%; aspect-ratio: 1; border-radius: 8px; border: 2px solid #E8E8E8; position: relative; display: flex; align-items: center; justify-content: center; }
.custom-input-row { display: flex; align-items: center; gap: 8px; }
.custom-label { font-size: 13px; color: #888; flex-shrink: 0; }
.custom-input { flex: 1; height: 36px; border: 1px solid #E0E0E0; border-radius: 8px; padding: 0 10px; font-size: 13px; color: #2D2A26; }
.custom-apply-btn { background: #E8936A; border-radius: 8px; padding: 8px 14px; flex-shrink: 0; }
.custom-apply-btn:active { opacity: 0.85; }
.custom-apply-text { font-size: 13px; color: #FFF; font-weight: 600; }
.bg-grid { display: flex; flex-wrap: wrap; background: #FFF; margin: 0 12px; border-radius: 14px; padding: 12px; gap: 10px; }
.bg-item { width: calc(25% - 8px); position: relative; border-radius: 10px; overflow: hidden; aspect-ratio: 1; }
.bg-item:active { opacity: 0.8; }
.bg-img { width: 100%; height: 100%; display: block; }
.bg-check { position: absolute; top: 4px; right: 4px; width: 20px; height: 20px; border-radius: 10px; background: #E8936A; display: flex; align-items: center; justify-content: center; }
.bg-check-icon { font-size: 12px; color: #FFF; font-weight: 700; }
.custom-img-box { margin: 0 12px; }
.custom-img-btn { background: #FFF; border-radius: 14px; padding: 24px; display: flex; flex-direction: column; align-items: center; gap: 6px; }
.custom-img-btn:active { background: #F8F8F8; }
.custom-img-icon { font-size: 28px; color: #E8936A; }
.custom-img-hint { font-size: 13px; color: #B0B0B0; }
.reset-btn { margin: 20px 12px 0; background: #FFF; border-radius: 14px; padding: 14px; text-align: center; }
.reset-btn:active { background: #F8F8F8; }
.reset-text { font-size: 15px; color: #E85D5D; font-weight: 600; }
</style>
