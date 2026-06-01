<template>
  <view class="page">
    <scroll-view class="scroll" scroll-y>

      <view class="empty-box">
        <text class="empty-text">更多设置</text>
        <text class="empty-hint">功能开发中，敬请期待...</text>
      </view>

      <view style="height: 40px;"></view>
    </scroll-view>
  </view>
</template>

<script>
export default {
  data() {
    return {}
  },
  methods: {}
}
</script>

<style>
page { background: #F2F2F2; }
.page { width: 100%; min-height: 100vh; background: #F2F2F2; }
.scroll { width: 100%; min-height: 100vh; }
.empty-box { display: flex; flex-direction: column; align-items: center; padding-top: 120px; }
.empty-text { font-size: 18px; font-weight: 700; color: #CCC; }
.empty-hint { font-size: 14px; color: #DDD; margin-top: 8px; }
</style>
