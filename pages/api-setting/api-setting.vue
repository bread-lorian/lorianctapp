<template>
  <view class="page">
    <scroll-view class="scroll" scroll-y>

      <text class="section-title">API设置</text>

      <!-- 提供商 -->
      <view class="group">
        <view class="item">
          <text class="item-label">AI提供商</text>
          <picker :range="providers" :range-key="'name'" @change="changeProvider">
            <view class="item-right">
              <text class="item-value">{{ currentProvider.name || '请选择' }}</text>
              <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
            </view>
          </picker>
        </view>
        <view class="item" v-if="currentProvider.id === 'custom'">
          <text class="item-label">API地址</text>
          <view class="item-right">
            <input class="api-input" v-model="apiUrl" placeholder="https://api.example.com" />
          </view>
        </view>
        <view class="item">
          <text class="item-label">API Key</text>
          <view class="item-right">
            <input class="api-input" v-model="apiKey" placeholder="输入API Key" :password="!showKey" />
          </view>
        </view>
        <view class="item">
          <text class="item-label">显示密钥</text>
          <switch :checked="showKey" @change="showKey = $event.detail.value" color="#E8936A" />
        </view>
      </view>

      <!-- 聊天模型 -->
      <text class="section-title">聊天模型</text>
      <view class="group">
        <view class="item" @click="fetchModels('chat')">
          <text class="item-label">{{ isFetchingChat ? '正在拉取...' : '拉取模型列表' }}</text>
          <view class="item-right">
            <text class="item-value" v-if="chatFetchStatus === 'success'">已获取{{ chatModelList.length }}个</text>
            <text class="item-value test-fail" v-else-if="chatFetchStatus === 'fail'">拉取失败</text>
            <text class="item-value" v-else>点击拉取</text>
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
        <view class="item" v-if="chatFetchStatus === 'none'">
          <text class="item-label">模型名称</text>
          <view class="item-right">
            <text class="item-value">请先拉取模型列表</text>
          </view>
        </view>
        <view class="item" v-if="chatFetchStatus === 'success' && chatModelList.length > 0">
          <text class="item-label">选择模型</text>
          <picker :range="chatModelList" :range-key="'id'" @change="changeChatModel">
            <view class="item-right">
              <text class="item-value">{{ modelName || '请选择' }}</text>
              <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
            </view>
          </picker>
        </view>
        <view class="item" v-if="chatFetchStatus === 'fail'">
          <text class="item-label">模型名称</text>
          <view class="item-right">
            <input class="api-input" v-model="modelName" placeholder="手动输入模型名称" />
          </view>
        </view>
      </view>

      <!-- 视觉模型 -->
      <text class="section-title">视觉模型（图片识别）</text>

      <!-- 不支持视觉的提供商 -->
      <view class="group" v-if="!currentProvider.visionSupport && currentProvider.id !== 'custom'">
        <view class="item">
          <view class="item-label-box">
            <text class="item-label" style="color: #B0B0B0;">该提供商不支持视觉模型</text>
            <text class="item-hint">{{ currentProvider.name }}暂不支持图片识别，请切换其他提供商。</text>
          </view>
        </view>
      </view>

      <!-- 支持视觉的提供商 -->
      <view class="group" v-if="currentProvider.visionSupport || currentProvider.id === 'custom'">
        <view class="item">
          <text class="item-label">启用视觉模型</text>
          <switch :checked="useVision" @change="toggleVision" color="#E8936A" />
        </view>

        <template v-if="useVision">
          <view class="item" v-if="currentProvider.id === 'custom'">
            <text class="item-label">视觉API地址</text>
            <view class="item-right">
              <input class="api-input" v-model="visionApiUrl" placeholder="留空则用主API地址" />
            </view>
          </view>

          <view class="item" @click="fetchModels('vision')">
            <text class="item-label">{{ isFetchingVision ? '正在拉取...' : '拉取视觉模型列表' }}</text>
            <view class="item-right">
              <text class="item-value" v-if="visionFetchStatus === 'success'">已获取{{ visionModelList.length }}个</text>
              <text class="item-value test-fail" v-else-if="visionFetchStatus === 'fail'">拉取失败</text>
              <text class="item-value" v-else>点击拉取</text>
              <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
            </view>
          </view>

          <view class="item" v-if="visionFetchStatus === 'none'">
            <text class="item-label">视觉模型名称</text>
            <view class="item-right">
              <text class="item-value">请先拉取模型列表</text>
            </view>
          </view>

          <view class="item" v-if="visionFetchStatus === 'success' && visionModelList.length > 0">
            <text class="item-label">选择视觉模型</text>
            <picker :range="visionModelList" :range-key="'id'" @change="changeVisionModel">
              <view class="item-right">
                <text class="item-value">{{ visionModelName || '请选择' }}</text>
                <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
              </view>
            </picker>
          </view>

          <view class="item" v-if="visionFetchStatus === 'fail'">
            <text class="item-label">视觉模型名称</text>
            <view class="item-right">
              <input class="api-input" v-model="visionModelName" placeholder="手动输入视觉模型" />
            </view>
          </view>
        </template>
      </view>

      <!-- 测试 -->
      <text class="section-title">测试</text>
      <view class="group">
        <view class="item" @click="testConnection('chat')">
          <text class="item-label">测试聊天模型</text>
          <view class="item-right">
            <text class="item-value" v-if="isTestingChat">测试中...</text>
            <text class="item-value test-success" v-else-if="testChatResult === 'success'">连接成功</text>
            <text class="item-value test-fail" v-else-if="testChatResult === 'fail'">连接失败</text>
            <text class="item-value" v-else>点击测试</text>
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
        <view class="item" v-if="useVision && (currentProvider.visionSupport || currentProvider.id === 'custom')" @click="testConnection('vision')">
          <text class="item-label">测试视觉模型</text>
          <view class="item-right">
            <text class="item-value" v-if="isTestingVision">测试中...</text>
            <text class="item-value test-success" v-else-if="testVisionResult === 'success'">连接成功</text>
            <text class="item-value test-fail" v-else-if="testVisionResult === 'fail'">连接失败</text>
            <text class="item-value" v-else>点击测试</text>
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
      </view>

      <!-- 文档 -->
      <text class="section-title">官方文档</text>
      <view class="group">
        <view class="item" @click="openDoc">
          <text class="item-label">{{ currentProvider.name }} 官方文档</text>
          <view class="item-right">
            <text class="item-value">点击跳转</text>
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
      </view>

      <!-- 保存 -->
      <view class="save-btn" @click="saveConfig">
        <text class="save-text">保存设置</text>
      </view>

      <view style="height: 60px;"></view>
    </scroll-view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      providers: [
        {
          id: 'deepseek',
          name: 'DeepSeek',
          baseUrl: 'https://api.deepseek.com',
          modelUrl: 'https://api.deepseek.com/v1/models',
          docUrl: 'https://platform.deepseek.com/api-docs',
          visionSupport: false
        },
        {
          id: 'openai',
          name: 'OpenAI',
          baseUrl: 'https://api.openai.com',
          modelUrl: 'https://api.openai.com/v1/models',
          docUrl: 'https://platform.openai.com/docs/api-reference',
          visionSupport: true
        },
        {
          id: 'xiaomi',
          name: '小米MiMo',
          baseUrl: 'https://api.xiaomi.com',
          modelUrl: 'https://api.xiaomi.com/v1/models',
          docUrl: 'https://dev.mi.com/ai',
          visionSupport: false
        },
        {
          id: 'siliconflow',
          name: '硅基流动',
          baseUrl: 'https://api.siliconflow.cn',
          modelUrl: 'https://api.siliconflow.cn/v1/models',
          docUrl: 'https://docs.siliconflow.cn',
          visionSupport: true
        },
        {
          id: 'zhipu',
          name: '智谱AI',
          baseUrl: 'https://open.bigmodel.cn/api/paas',
          modelUrl: 'https://open.bigmodel.cn/api/paas/v4/models',
          docUrl: 'https://open.bigmodel.cn/dev/api',
          visionSupport: true
        },
        {
          id: 'moonshot',
          name: '月之暗面',
          baseUrl: 'https://api.moonshot.cn',
          modelUrl: 'https://api.moonshot.cn/v1/models',
          docUrl: 'https://platform.moonshot.cn/docs',
          visionSupport: true
        },
        {
          id: 'tongyi',
          name: '通义千问',
          baseUrl: 'https://dashscope.aliyuncs.com/compatible-mode',
          modelUrl: 'https://dashscope.aliyuncs.com/compatible-mode/v1/models',
          docUrl: 'https://help.aliyun.com/zh/dashscope/developer-reference/api-details',
          visionSupport: true
        },
        {
          id: 'custom',
          name: '自定义API',
          baseUrl: '',
          modelUrl: '',
          docUrl: '',
          visionSupport: true
        }
      ],
      currentProvider: {},
      apiKey: '',
      apiUrl: '',
      showKey: false,
      modelName: '',
      chatModelList: [],
      isFetchingChat: false,
      chatFetchStatus: 'none',
      useVision: false,
      visionModelName: '',
      visionApiUrl: '',
      visionModelList: [],
      isFetchingVision: false,
      visionFetchStatus: 'none',
      isTestingChat: false,
      testChatResult: '',
      isTestingVision: false,
      testVisionResult: ''
    }
  },

  onLoad() {
    this.loadConfig()
  },

  methods: {

    loadConfig() {
      var c = uni.getStorageSync('api_config')
      if (c) {
        try {
          var obj = typeof c === 'string' ? JSON.parse(c) : c
          this.apiKey = obj.apiKey || ''
          this.apiUrl = obj.apiUrl || ''
          this.modelName = obj.modelName || ''
          this.useVision = obj.useVision || false
          this.visionModelName = obj.visionModelName || ''
          this.visionApiUrl = obj.visionApiUrl || ''
          var pid = obj.provider || 'deepseek'
          for (var i = 0; i < this.providers.length; i++) {
            if (this.providers[i].id === pid) {
              this.currentProvider = this.providers[i]
              break
            }
          }
          if (!this.currentProvider.id) this.currentProvider = this.providers[0]
          if (obj.modelName) this.chatFetchStatus = 'success'
          if (obj.visionModelName && obj.useVision) this.visionFetchStatus = 'success'
        } catch(e) {
          this.currentProvider = this.providers[0]
        }
      } else {
        this.currentProvider = this.providers[0]
      }

      // 如果当前提供商不支持视觉，强制关闭
      if (!this.currentProvider.visionSupport && this.currentProvider.id !== 'custom') {
        this.useVision = false
      }
    },

    changeProvider(e) {
      var idx = e.detail.value
      this.currentProvider = this.providers[idx]
      this.apiUrl = this.currentProvider.id === 'custom' ? '' : this.currentProvider.baseUrl
      this.chatModelList = []
      this.modelName = ''
      this.chatFetchStatus = 'none'
      this.visionModelList = []
      this.visionModelName = ''
      this.visionFetchStatus = 'none'
      this.testChatResult = ''
      this.testVisionResult = ''

      // 不支持视觉的提供商，自动关闭视觉
      if (!this.currentProvider.visionSupport && this.currentProvider.id !== 'custom') {
        this.useVision = false
      }
    },

    changeChatModel(e) {
      this.modelName = this.chatModelList[e.detail.value].id
      this.testChatResult = ''
    },

    changeVisionModel(e) {
      this.visionModelName = this.visionModelList[e.detail.value].id
      this.testVisionResult = ''
    },

    toggleVision(e) {
      this.useVision = e.detail.value
      if (!this.useVision) {
        this.visionFetchStatus = 'none'
        this.visionModelList = []
        this.testVisionResult = ''
      }
    },

    // ========== 拉取模型 ==========
    fetchModels(type) {
      var self = this
      var isVision = type === 'vision'
      if (isVision ? self.isFetchingVision : self.isFetchingChat) return
      if (!self.currentProvider.id) { uni.showToast({ title: '请先选择提供商', icon: 'none' }); return }

      if (isVision) self.isFetchingVision = true; else self.isFetchingChat = true
      if (isVision) self.visionModelList = []; else self.chatModelList = []

      var modelUrl = ''
      var header = { 'Content-Type': 'application/json' }

      if (self.currentProvider.id === 'custom') {
        var url = isVision ? (self.visionApiUrl || self.apiUrl) : self.apiUrl
        if (!url) { uni.showToast({ title: '请先填写API地址', icon: 'none' }); if (isVision) self.isFetchingVision = false; else self.isFetchingChat = false; return }
        modelUrl = url.replace(/\/+$/, '') + '/v1/models'
      } else {
        modelUrl = self.currentProvider.modelUrl
      }

      if (self.apiKey) header['Authorization'] = 'Bearer ' + self.apiKey

      uni.request({
        url: modelUrl,
        method: 'GET',
        header: header,
        timeout: 15000,
        success: function(res) {
          var models = []
          if (res.statusCode === 200 && res.data) {
            var data = res.data
            if (data.data && Array.isArray(data.data)) {
              for (var i = 0; i < data.data.length; i++) models.push({ id: data.data[i].id })
            } else if (Array.isArray(data)) {
              for (var j = 0; j < data.length; j++) models.push({ id: data[j].id || data[j].name || data[j].model })
            }
          }
          if (models.length > 0) {
            if (isVision) { self.visionModelList = models; self.visionFetchStatus = 'success' }
            else { self.chatModelList = models; self.chatFetchStatus = 'success' }
            uni.showToast({ title: '获取到' + models.length + '个模型', icon: 'none' })
          } else {
            if (isVision) self.visionFetchStatus = 'fail'; else self.chatFetchStatus = 'fail'
            uni.showToast({ title: '拉取失败，请手动输入', icon: 'none' })
          }
        },
        fail: function() {
          if (isVision) self.visionFetchStatus = 'fail'; else self.chatFetchStatus = 'fail'
          uni.showToast({ title: '网络错误', icon: 'none' })
        },
        complete: function() {
          if (isVision) self.isFetchingVision = false; else self.isFetchingChat = false
        }
      })
    },

    // ========== 测试 ==========
    testConnection(type) {
      var self = this
      if (type === 'chat') {
        if (self.isTestingChat) return
        if (!self.apiKey) { uni.showToast({ title: '请填写API Key', icon: 'none' }); return }
        if (!self.modelName) { uni.showToast({ title: '请选择聊天模型', icon: 'none' }); return }
      }
      if (type === 'vision') {
        if (self.isTestingVision) return
        if (!self.visionModelName) { uni.showToast({ title: '请选择视觉模型', icon: 'none' }); return }
      }

      if (type === 'chat') self.isTestingChat = true; else self.isTestingVision = true
      if (type === 'chat') self.testChatResult = ''; else self.testVisionResult = ''

      var baseUrl = ''
      if (self.currentProvider.id === 'custom') {
        baseUrl = type === 'vision' ? (self.visionApiUrl || self.apiUrl) : self.apiUrl
        baseUrl = baseUrl.replace(/\/+$/, '')
      } else {
        baseUrl = self.currentProvider.baseUrl
      }

      var mdl = type === 'vision' ? self.visionModelName : self.modelName

      uni.request({
        url: baseUrl + '/v1/chat/completions',
        method: 'POST',
        header: { 'Authorization': 'Bearer ' + self.apiKey, 'Content-Type': 'application/json' },
        data: { model: mdl, messages: [{ role: 'user', content: '你好' }], max_tokens: 10 },
        timeout: 15000,
        success: function(res) {
          var ok = res.statusCode === 200 && res.data && res.data.choices
          if (type === 'chat') self.testChatResult = ok ? 'success' : 'fail'
          else self.testVisionResult = ok ? 'success' : 'fail'
          uni.showToast({ title: ok ? '连接成功' : '连接失败', icon: ok ? 'success' : 'none' })
        },
        fail: function() {
          if (type === 'chat') self.testChatResult = 'fail'; else self.testVisionResult = 'fail'
          uni.showToast({ title: '网络错误', icon: 'none' })
        },
        complete: function() {
          if (type === 'chat') self.isTestingChat = false; else self.isTestingVision = false
        }
      })
    },

    openDoc() {
      var url = this.currentProvider.docUrl
      if (!url) { uni.showToast({ title: '自定义API无官方文档', icon: 'none' }); return }
      // #ifdef APP-PLUS
      plus.runtime.openURL(url)
      // #endif
      // #ifndef APP-PLUS
      uni.setClipboardData({ data: url, success: function() { uni.showToast({ title: '链接已复制', icon: 'none' }) } })
      // #endif
    },

    // ========== 保存 ==========
    saveConfig() {
      var self = this
      if (!self.apiKey) { uni.showToast({ title: '请填写API Key', icon: 'none' }); return }
      if (!self.modelName) { uni.showToast({ title: '请选择聊天模型', icon: 'none' }); return }

      // 不支持视觉的提供商，强制关闭视觉
      var canVision = self.currentProvider.visionSupport || self.currentProvider.id === 'custom'
      var saveUseVision = canVision ? self.useVision : false

      var config = {
        provider: self.currentProvider.id,
        apiKey: self.apiKey,
        apiUrl: self.currentProvider.id === 'custom' ? self.apiUrl.replace(/\/+$/, '') : self.currentProvider.baseUrl,
        modelName: self.modelName,
        visionSupport: canVision,
        useVision: saveUseVision,
        visionModelName: saveUseVision ? self.visionModelName : '',
        visionApiUrl: saveUseVision ? (self.currentProvider.id === 'custom' ? (self.visionApiUrl || self.apiUrl).replace(/\/+$/, '') : self.currentProvider.baseUrl) : ''
      }

      uni.setStorageSync('api_config', JSON.stringify(config))
      uni.showToast({ title: '设置已保存', icon: 'success' })
      setTimeout(function() { uni.navigateBack() }, 800)
    }
  }
}
</script>

<style>
page { background: #F2F2F2; }
.page { width: 100%; min-height: 100vh; background: #F2F2F2; }
.scroll { width: 100%; min-height: 100vh; }
.section-title { font-size: 13px; font-weight: 500; color: #888; padding: 20px 16px 8px; display: block; }
.group { background: #FFF; margin: 0 12px; border-radius: 14px; overflow: hidden; }
.item { display: flex; align-items: center; justify-content: space-between; padding: 14px 16px; }
.item:active { background: #F8F8F8; }
.item + .item { border-top: 0.5px solid #F0F0F0; }
.item-label-box { display: flex; flex-direction: column; gap: 3px; flex: 1; }
.item-label { font-size: 15px; color: #2D2A26; flex-shrink: 0; }
.item-hint { font-size: 12px; color: #B0B0B0; }
.item-right { display: flex; align-items: center; gap: 4px; flex: 1; justify-content: flex-end; }
.item-value { font-size: 13px; color: #B0B0B0; text-align: right; }
.test-success { color: #4CAF50 !important; }
.test-fail { color: #E85D5D !important; }
.api-input { width: 200px; height: 32px; font-size: 13px; color: #2D2A26; text-align: right; background: transparent; }
.arrow-img { width: 16px; height: 16px; flex-shrink: 0; }
.save-btn { width: calc(100% - 32px); margin: 30px 16px 0; height: 48px; background: #E8936A; border-radius: 12px; display: flex; align-items: center; justify-content: center; }
.save-text { color: #FFF; font-size: 16px; font-weight: 600; }
.save-btn:active { opacity: 0.85; }
</style>
