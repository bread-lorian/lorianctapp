<template>
  <view class="page">
    <scroll-view class="scroll" scroll-y>

      <!-- 版本 1.2.0 -->
      <view class="version-card">
        <view class="version-header">
          <view class="version-badge latest">
            <text class="version-badge-text">最新</text>
          </view>
          <text class="version-number">v1.0.2</text>
          <text class="version-date">2025年6月</text>
        </view>

        <view class="section">
          <view class="section-title new">
            <text class="section-icon">✨</text>
            <text class="section-label">新增功能</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">新增小世界模块，所有用户可以浏览和发布动态</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">新增通知中心，接收系统公告和推送消息</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">新增后台管理系统，支持数据看板、公告管理、推送管理</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">新增更多设置页面</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">新增神态动作气泡内/气泡外显示模式切换</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">新增神态动作字数限制（10/20/30字）</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">新增收藏表情包功能，支持添加和删除自定义表情</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">新增图片发送功能，支持AI视觉识别</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">新增表情包发送功能，AI可识别表情包内容</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">新增设备自动上报和统计功能</text>
          </view>
        </view>

        <view class="section">
          <view class="section-title fix">
            <text class="section-icon">🔧</text>
            <text class="section-label">问题修复</text>
          </view>
          <view class="item">
            <text class="item-dot fix"></text>
            <text class="item-text">修复神态动作开关关闭后仍然显示动作的问题</text>
          </view>
          <view class="item">
            <text class="item-dot fix"></text>
            <text class="item-text">修复聊天设置与聊天页面神态开关不同步的问题</text>
          </view>
          <view class="item">
            <text class="item-dot fix"></text>
            <text class="item-text">修复收藏表情包重启后消失变成透明的问题</text>
          </view>
          <view class="item">
            <text class="item-dot fix"></text>
            <text class="item-text">修复发送图片四周带有黑框的问题</text>
          </view>
          <view class="item">
            <text class="item-dot fix"></text>
            <text class="item-text">修复记忆页面点击添加无反应的问题（手机端适配）</text>
          </view>
          <view class="item">
            <text class="item-dot fix"></text>
            <text class="item-text">修复API设置中不支持视觉的提供商错误调用视觉模型的问题</text>
          </view>
          <view class="item">
            <text class="item-dot fix"></text>
            <text class="item-text">修复DeepSeek等不支持视觉的提供商发送图片报错的问题</text>
          </view>
          <view class="item">
            <text class="item-dot fix"></text>
            <text class="item-text">修复聊天页面模板语法错误导致编译失败的问题</text>
          </view>
          <view class="item">
            <text class="item-dot fix"></text>
            <text class="item-text">修复App.vue设备ID重复声明导致编译报错的问题</text>
          </view>
          <view class="item">
            <text class="item-dot fix"></text>
            <text class="item-text">修复消息首页添加弹窗位置不适配安卓的问题</text>
          </view>
        </view>

        <view class="section">
          <view class="section-title improve">
            <text class="section-icon">⚡</text>
            <text class="section-label">优化改进</text>
          </view>
          <view class="item">
            <text class="item-dot improve"></text>
            <text class="item-text">优化AI伴侣只能添加一个的限制，朋友可转为伴侣</text>
          </view>
          <view class="item">
            <text class="item-dot improve"></text>
            <text class="item-text">优化消息列表显示最后一条消息预览</text>
          </view>
          <view class="item">
            <text class="item-dot improve"></text>
            <text class="item-text">优化添加弹窗改为微信风格下拉菜单</text>
          </view>
          <view class="item">
            <text class="item-dot improve"></text>
            <text class="item-text">优化表情面板添加按钮固定在最前面</text>
          </view>
          <view class="item">
            <text class="item-dot improve"></text>
            <text class="item-text">优化不支持视觉的提供商自动关闭视觉选项</text>
          </view>
          <view class="item">
            <text class="item-dot improve"></text>
            <text class="item-text">优化页面适配安卓不同屏幕尺寸</text>
          </view>
        </view>
      </view>

      <!-- 版本 1.1.0 -->
      <view class="version-card">
        <view class="version-header">
          <text class="version-number">v1.0.1</text>
          <text class="version-date">2025年5月</text>
        </view>

        <view class="section">
          <view class="section-title new">
            <text class="section-icon">✨</text>
            <text class="section-label">新增功能</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">新增AI记忆系统，自动记住用户提到的信息</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">新增聊天背景自定义（纯色/图片）</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">新增AI头像自定义设置</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">新增聊天记录导出/导入功能</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">新增自动备份功能</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">新增置顶聊天和黑名单功能</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">新增AI主动发消息功能（超过3小时未回复）</text>
          </view>
        </view>

        <view class="section">
          <view class="section-title improve">
            <text class="section-icon">⚡</text>
            <text class="section-label">优化改进</text>
          </view>
          <view class="item">
            <text class="item-dot improve"></text>
            <text class="item-text">支持多个AI提供商（DeepSeek、OpenAI、硅基流动等）</text>
          </view>
          <view class="item">
            <text class="item-dot improve"></text>
            <text class="item-text">支持手动输入和自动拉取模型列表</text>
          </view>
          <view class="item">
            <text class="item-dot improve"></text>
            <text class="item-text">优化API连接测试功能</text>
          </view>
        </view>
      </view>

      <!-- 版本 1.0.0 -->
      <view class="version-card">
        <view class="version-header">
          <text class="version-number">v1.0.0</text>
          <text class="version-date">2025年4月</text>
        </view>

        <view class="section">
          <view class="section-title new">
            <text class="section-icon">✨</text>
            <text class="section-label">首次发布</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">AI伴侣和AI朋友两种角色类型</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">自定义AI人设、性别、背景、习惯用语</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">AI对话聊天功能</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">神态动作显示功能</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">默认表情面板</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">个人信息编辑</text>
          </view>
          <view class="item">
            <text class="item-dot new"></text>
            <text class="item-text">API设置页面</text>
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
    return {}
  },
  methods: {}
}
</script>

<style>
page { background: #F2F2F2; }
.page { width: 100%; min-height: 100vh; background: #F2F2F2; }
.scroll { width: 100%; min-height: 100vh; }

.version-card {
  background: #FFF;
  margin: 12px;
  border-radius: 14px;
  overflow: hidden;
}

.version-header {
  padding: 20px 20px 16px;
  border-bottom: 0.5px solid #F0F0F0;
  display: flex;
  align-items: center;
  gap: 10px;
}
.version-badge {
  padding: 2px 8px;
  border-radius: 4px;
  background: #E8936A;
}
.version-badge-text {
  font-size: 11px;
  color: #FFF;
  font-weight: 600;
}
.version-number {
  font-size: 20px;
  font-weight: 700;
  color: #2D2A26;
}
.version-date {
  font-size: 13px;
  color: #B0B0B0;
  margin-left: auto;
}

.section {
  padding: 16px 20px 4px;
}
.section-title {
  display: flex;
  align-items: center;
  gap: 6px;
  margin-bottom: 12px;
}
.section-icon {
  font-size: 15px;
}
.section-label {
  font-size: 14px;
  font-weight: 600;
}
.section-title.new .section-label { color: #E8936A; }
.section-title.fix .section-label { color: #4CAF50; }
.section-title.improve .section-label { color: #2196F3; }

.item {
  display: flex;
  align-items: flex-start;
  gap: 10px;
  padding: 6px 0;
}
.item-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  flex-shrink: 0;
  margin-top: 7px;
}
.item-dot.new { background: #E8936A; }
.item-dot.fix { background: #4CAF50; }
.item-dot.improve { background: #2196F3; }
.item-text {
  font-size: 14px;
  color: #555;
  line-height: 1.6;
}
</style>
