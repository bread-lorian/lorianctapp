<template>
  <view class="page">
    <!-- 自愿赞助弹窗 -->
    <view class="modal-mask" v-if="showTip" @click="showTip = false"></view>
    <view class="tip-modal" v-if="showTip">
      <view class="tip-card">
        <image class="tip-icon-img" src="/static/icon-heart-hand.png" mode="aspectFit" />
        <text class="tip-title">自愿赞助</text>
        <text class="tip-text">赞助为自愿行为，不进行任何强求</text>
        <text class="tip-text">赞助后不会获得任何特权或功能解锁</text>
        <text class="tip-text">您可以在任何时候选择是否赞助</text>
        <text class="tip-sub">落雨感谢您的每一份心意</text>
        <view class="tip-btn" @click="showTip = false">
          <text class="tip-btn-text">我知道了</text>
        </view>
      </view>
    </view>

    <scroll-view class="scroll" scroll-y>
      <view class="donate-header">
        <image class="donate-icon-img" src="/static/icon-donate-big.png" mode="aspectFit" />
        <text class="donate-title">支持我们</text>
        <text class="donate-desc">您的每一份支持都是我们前进的动力</text>
      </view>

      <view class="section-title"><text>支持方式</text></view>
      <view class="card-group">
        <view class="card-item" v-for="(item, i) in ways" :key="i" @click="handleWay(item)">
          <view class="card-left">
            <view class="card-icon-box" :style="{ background: item.bg }">
              <image class="card-icon-img" :src="item.icon" mode="aspectFit" />
            </view>
            <view class="card-info">
              <text class="card-name">{{ item.name }}</text>
              <text class="card-desc">{{ item.desc }}</text>
            </view>
          </view>
          <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
        </view>
      </view>



      <view class="thanks-card">
        <text class="thanks-text">感谢每一位支持落雨的朋友</text>
        <text class="thanks-sub">你们的支持是我们最大的动力</text>
      </view>

      <view style="height: 30px;"></view>
    </scroll-view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      showTip: true,
      ways: [
        { icon: '/static/icon-wechat-pay.png', name: '微信', desc: '微信扫码支持', bg: '#E8F5E9', link: 'https://www.luoyeltd.com.cn/zf/wxzf.jpg' },
        { icon: '/static/icon-ali-pay.png', name: '支付宝', desc: '支付宝扫码支持', bg: '#E3F2FD', link: 'https://www.luoyeltd.com.cn/zf/zfb.jpg' },
        { icon: '/static/icon-sponsor-money.png', name: '赞助', desc: '直接赞助支持', bg: '#FFF3E0', link: 'https://www.luoyeltd.com.cn/zf/zsm.jpg' },
        { icon: '/static/icon-afdian.png', name: '爱发电赞助', desc: '通过爱发电支持我们', bg: '#F5D0FF', link: 'https://ifdian.net/a/qingtianlive' }
      ],
      sponsors: [
        { name: '闻风', desc: '感谢大佬的支持', time: '2025年3月' }
      ]
    }
  },
  methods: {
    handleWay(item) {
      if (item.link) {
        // #ifdef APP-PLUS
        plus.runtime.openURL(item.link)
        // #endif
        // #ifdef H5
        window.open(item.link, '_blank')
        // #endif
      } else {
        uni.showToast({ title: '暂未开放', icon: 'none' })
      }
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
  position: relative;
}

.scroll {
  width: 100%;
  min-height: 100vh;
}

/* 弹窗 */
.modal-mask {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.45);
  z-index: 200;
  animation: maskIn 0.25s ease;
}

.tip-modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 201;
  padding: 40px;
}

.tip-card {
  background: #FFF;
  border-radius: 20px;
  padding: 30px 24px 24px;
  display: flex;
  flex-direction: column;
  align-items: center;
  animation: modalIn 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  width: 100%;
}

.tip-icon-img {
  width: 56px;
  height: 56px;
  margin-bottom: 12px;
}

.tip-title {
  font-size: 20px;
  font-weight: 700;
  color: #2D2A26;
  margin-bottom: 16px;
}

.tip-text {
  font-size: 14px;
  color: #666;
  line-height: 1.8;
  text-align: center;
  display: block;
}

.tip-sub {
  font-size: 12px;
  color: #E8936A;
  margin-top: 14px;
  display: block;
}

.tip-btn {
  width: 100%;
  height: 44px;
  background: #E8936A;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 20px;
}

.tip-btn:active {
  opacity: 0.85;
}

.tip-btn-text {
  font-size: 15px;
  font-weight: 600;
  color: #FFF;
}

@keyframes maskIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes modalIn {
  from { opacity: 0; transform: scale(0.85); }
  to { opacity: 1; transform: scale(1); }
}

/* 页面内容 */
.donate-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 30px 16px 20px;
}

.donate-icon-img {
  width: 56px;
  height: 56px;
}

.donate-title {
  font-size: 20px;
  font-weight: 700;
  color: #2D2A26;
  margin-top: 10px;
}

.donate-desc {
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

.arrow-img {
  width: 16px;
  height: 16px;
  flex-shrink: 0;
}

.sponsor-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px;
}

.sponsor-item + .sponsor-item {
  border-top: 0.5px solid #F0F0F0;
}

.sponsor-left {
  display: flex;
  align-items: center;
  gap: 12px;
}

.sponsor-icon-box {
  width: 40px;
  height: 40px;
  border-radius: 12px;
  background: #F5F5F5;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.sponsor-icon-img {
  width: 22px;
  height: 22px;
}

.sponsor-info {
  flex: 1;
}

.sponsor-name {
  font-size: 15px;
  font-weight: 600;
  color: #2D2A26;
  display: block;
}

.sponsor-desc {
  font-size: 12px;
  color: #B0B0B0;
  margin-top: 3px;
  display: block;
}

.sponsor-time {
  font-size: 12px;
  color: #C0C0C0;
  flex-shrink: 0;
}

.thanks-card {
  margin: 20px 12px 0;
  padding: 20px;
  background: linear-gradient(135deg, #FFF0E8, #FFE5D0);
  border-radius: 14px;
  text-align: center;
}

.thanks-text {
  font-size: 15px;
  font-weight: 600;
  color: #E8936A;
  display: block;
}

.thanks-sub {
  font-size: 12px;
  color: #C0A090;
  margin-top: 6px;
  display: block;
}
</style>
