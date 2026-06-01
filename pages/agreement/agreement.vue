<template>
  <view class="page">
    <scroll-view class="scroll" scroll-y>
      <view class="tabs">
        <view class="tab-item" :class="{ active: curTab === 0 }" @click="curTab = 0">
          <text class="tab-text">用户协议</text>
        </view>
        <view class="tab-item" :class="{ active: curTab === 1 }" @click="curTab = 1">
          <text class="tab-text">隐私政策</text>
        </view>
      </view>

      <view class="content-card" v-if="curTab === 0">
        <text class="content-title">落雨AI用户协议</text>
        <text class="content-date">更新日期：2026年5月30日</text>
        <text class="content-h2">一、服务条款的确认</text>
        <text class="content-text">欢迎使用落雨AI（以下简称"本应用"）。请您在使用本应用前仔细阅读以下条款。使用本应用即表示您同意遵守本协议。</text>
        <text class="content-h2">二、服务内容</text>
        <text class="content-text">本应用为用户提供AI伴侣聊天服务。用户可通过配置API接入第三方大模型，实现与AI的对话交流。</text>
        <text class="content-h2">三、用户行为规范</text>
        <text class="content-text">用户不得利用本应用从事违法违规活动，不得发布违反国家法律法规的内容。用户应对其使用本应用的行为承担法律责任。</text>
        <text class="content-h2">四、知识产权</text>
        <text class="content-text">本应用的所有内容（包括但不限于文字、图片、音频、视频、软件、代码）均受知识产权保护。未经授权，任何人不得复制、修改、传播本应用的任何内容。</text>
        <text class="content-h2">五、免责声明</text>
        <text class="content-text">本应用提供的AI回复内容由第三方大模型生成，不代表本应用立场。用户应自行判断AI回复内容的准确性和适用性。</text>
        <text class="content-h2">六、协议修改</text>
        <text class="content-text">本应用有权根据需要修改本协议，修改后的协议将在应用内公布。继续使用本应用即视为同意修改后的协议。</text>
      </view>

      <view class="content-card" v-if="curTab === 1">
        <text class="content-title">落雨AI隐私政策</text>
        <text class="content-date">更新日期：2026年5月30日</text>
        <text class="content-h2">一、信息收集</text>
        <text class="content-text">本应用尊重并保护用户隐私。我们可能收集的信息包括：设备信息、使用日志、用户主动填写的个人信息。</text>
        <text class="content-h2">二、信息使用</text>
        <text class="content-text">收集的信息仅用于提供和改进服务，不会用于其他商业目的。您的聊天内容仅保存在本地设备，不会上传至我们的服务器。</text>
        <text class="content-h2">三、信息存储</text>
        <text class="content-text">用户的聊天数据存储在本地设备中。API Key等敏感信息仅保存在本地，不会上传至任何服务器。</text>
        <text class="content-h2">四、信息安全</text>
        <text class="content-text">我们采取合理的技术措施保护用户信息安全，但无法保证信息传输的绝对安全。</text>
        <text class="content-h2">五、第三方服务</text>
        <text class="content-text">用户配置的API由第三方服务商提供，相关数据处理受第三方隐私政策约束。</text>
        <text class="content-h2">六、联系我们</text>
        <text class="content-text">如对本隐私政策有任何疑问，请通过应用内的"联系我们"功能与我们取得联系。</text>
      </view>

      <view style="height: 30px;"></view>
    </scroll-view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      curTab: 0
    }
  }
}
</script>

<style>
page { background: #F2F2F2; }
.page { width: 100%; min-height: 100vh; background: #F2F2F2; }
.scroll { width: 100%; min-height: 100vh; }

.tabs { display: flex; margin: 12px 12px 0; background: #FFF; border-radius: 14px; overflow: hidden; }
.tab-item { flex: 1; height: 44px; display: flex; align-items: center; justify-content: center; position: relative; }
.tab-item:active { background: #F8F8F8; }
.tab-text { font-size: 14px; color: #999; }
.tab-item.active .tab-text { color: #2D2A26; font-weight: 600; }
.tab-item.active::after { content: ''; position: absolute; bottom: 0; left: 50%; transform: translateX(-50%); width: 24px; height: 3px; background: #E8936A; border-radius: 2px; }

.content-card { background: #FFF; margin: 12px; border-radius: 14px; padding: 20px; }
.content-title { font-size: 18px; font-weight: 700; color: #2D2A26; display: block; }
.content-date { font-size: 12px; color: #B0B0B0; margin-top: 6px; display: block; padding-bottom: 16px; border-bottom: 0.5px solid #F0F0F0; }
.content-h2 { font-size: 15px; font-weight: 600; color: #2D2A26; margin-top: 16px; display: block; }
.content-text { font-size: 14px; color: #555; line-height: 1.7; margin-top: 6px; display: block; }
</style>
