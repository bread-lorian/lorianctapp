<template>
  <view class="page">
    <scroll-view class="scroll" scroll-y>
      <view class="filing-header">
        <image class="filing-icon-img" src="/static/icon-filing-big.png" mode="aspectFit" />
        <text class="filing-title">软件ICP备案</text>
      </view>

      <view class="card-group">
        <view class="info-item" v-for="(item, i) in infos" :key="i">
          <text class="info-label">{{ item.label }}</text>
          <text class="info-value">{{ item.value }}</text>
        </view>
      </view>

      <view class="notice-card">
        <text class="notice-title">备案说明</text>
        <text class="notice-text">本软件已完成相关备案手续，所有信息均可在工信部备案系统中查询验证。如有疑问，请通过"联系我们"与我们取得联系。</text>
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
  data() {
    return {
      infos: [
        { label: '软件名称', value: '落雨' },
        { label: '软件版本', value: 'v1.0.0' },
        { label: '备案号', value: '鲁ICP备2025170504号-1A（变更中）' },
        { label: '主办单位', value: '落雨纪元团队' },
        { label: '备案地区', value: '山东省' },
        { label: '首次备案时间', value: '2026年5月' }
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

.filing-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 30px 16px 20px;
}

.filing-icon-img {
  width: 56px;
  height: 56px;
}

.filing-title {
  font-size: 20px;
  font-weight: 700;
  color: #2D2A26;
  margin-top: 10px;
}

.card-group {
  background: #FFF;
  margin: 12px;
  border-radius: 14px;
  overflow: hidden;
}

.info-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 14px 16px;
}

.info-item + .info-item {
  border-top: 0.5px solid #F0F0F0;
}

.info-label {
  font-size: 14px;
  color: #2D2A26;
}

.info-value {
  font-size: 14px;
  color: #888;
  text-align: right;
  flex: 1;
  margin-left: 20px;
}

.notice-card {
  background: #FFF;
  margin: 0 12px;
  border-radius: 14px;
  padding: 16px;
}

.notice-title {
  font-size: 15px;
  font-weight: 600;
  color: #2D2A26;
  display: block;
  margin-bottom: 8px;
}

.notice-text {
  font-size: 13px;
  color: #888;
  line-height: 1.7;
  display: block;
}

.footer {
  text-align: center;
  padding: 30px 0 0;
}

.footer-line {
  font-size: 11px;
  color: #C0C0C0;
  line-height: 1.6;
  display: block;
}
</style>
