<template>
  <view class="page">
    <scroll-view class="scroll" scroll-y>

      <!-- AI卡片 -->
      <view class="ai-card" @click="goEdit">
        <view class="card-avatar" :class="aiInfo.bg || 'bg1'">
          <image v-if="aiInfo.avatar" :src="aiInfo.avatar" mode="aspectFill" class="card-avatar-img" />
          <image v-else src="/static/default-avatar.png" mode="aspectFit" class="card-avatar-img" />
        </view>
        <view class="card-info">
          <view class="card-name-row">
            <text class="card-name">{{ aiInfo.name }}</text>
            <view class="type-tag" v-if="aiInfo.type === 'partner'">
              <text class="type-tag-text">伴侣</text>
            </view>
            <view class="type-tag friend" v-else-if="aiInfo.type === 'friend'">
              <text class="type-tag-text">朋友</text>
            </view>
          </view>
          <text class="card-prompt">{{ aiInfo.personality || aiInfo.prompt || '暂无人设' }}</text>
        </view>
        <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
      </view>

      <!-- 基本设置 -->
      <text class="section-title">基本设置</text>
      <view class="group">
        <view class="item" @click="goEdit">
          <text class="item-label">人设设置</text>
          <view class="item-right">
            <text class="item-value">编辑</text>
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
        <view class="item" @click="goMemory">
          <text class="item-label">Ta的记忆</text>
          <view class="item-right">
            <text class="item-value">{{ memCount }}条</text>
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
        <view class="item" @click="goChatBg">
          <text class="item-label">聊天背景</text>
          <view class="item-right">
            <text class="item-value">{{ bgText }}</text>
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
        <view class="item" @click="goAvatar">
          <text class="item-label">头像设置</text>
          <view class="item-right">
            <text class="item-value">{{ aiInfo.avatar ? '已设置' : '默认' }}</text>
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
      </view>

      <!-- 聊天状态 -->
      <text class="section-title">聊天状态</text>
      <view class="group">
        <view class="item">
          <text class="item-label">置顶聊天</text>
          <switch :checked="pinned" @change="togglePinned" color="#E8936A" />
        </view>
        <view class="item">
          <view class="item-label-box">
            <text class="item-label warn-text">加入黑名单</text>
            <text class="item-hint">拉黑后将无法收到该AI的消息</text>
          </view>
          <switch :checked="blocked" @change="toggleBlocked" color="#E85D5D" />
        </view>
      </view>

      <!-- 神态设置 -->
      <text class="section-title">神态设置</text>
      <view class="group">
        <view class="item">
          <view class="item-label-box">
            <text class="item-label">神态动作</text>
            <text class="item-hint">开启后AI回复时会带上动作描写</text>
          </view>
          <switch :checked="showAction" @change="toggleAction" color="#E8936A" />
        </view>

        <view class="item sub-item" v-if="showAction">
          <text class="item-label">显示位置</text>
          <view class="seg-bar">
            <view class="seg-item" :class="{ active: actionPosition === 'outside' }" @click="setActionPos('outside')">
              <text class="seg-text">气泡外</text>
            </view>
            <view class="seg-item" :class="{ active: actionPosition === 'inside' }" @click="setActionPos('inside')">
              <text class="seg-text">气泡内</text>
            </view>
          </view>
        </view>

        <view class="item sub-item" v-if="showAction && actionPosition === 'outside'">
          <text class="item-label">字数限制</text>
          <view class="seg-bar">
            <view class="seg-item" :class="{ active: actionLimit === 10 }" @click="setActionLimit(10)">
              <text class="seg-text">10字</text>
            </view>
            <view class="seg-item" :class="{ active: actionLimit === 20 }" @click="setActionLimit(20)">
              <text class="seg-text">20字</text>
            </view>
            <view class="seg-item" :class="{ active: actionLimit === 30 }" @click="setActionLimit(30)">
              <text class="seg-text">30字</text>
            </view>
          </view>
        </view>
      </view>

      <!-- 神态预览 -->
      <view class="preview-box" v-if="showAction">
        <text class="preview-title">效果预览</text>
        <view class="preview-row" v-if="actionPosition === 'outside'">
          <view class="preview-action">
            <text class="preview-action-text">*微微低下头，脸有点红*</text>
          </view>
          <view class="preview-bubble">
            <text class="preview-bubble-text">我也很想你呀</text>
          </view>
        </view>
        <view class="preview-row" v-if="actionPosition === 'inside'">
          <view class="preview-bubble">
            <text class="preview-bubble-text">（微微低下头，脸有点红）我也很想你呀</text>
          </view>
        </view>
      </view>

      <!-- 消息设置 -->
      <text class="section-title">消息设置</text>
      <view class="group">
        <view class="item">
          <view class="item-label-box">
            <text class="item-label">自动发消息</text>
            <text class="item-hint">超过3小时未回复时AI会主动发消息</text>
          </view>
          <switch :checked="autoMessage" @change="toggleAutoMessage" color="#E8936A" />
        </view>
        <view class="item">
          <view class="item-label-box">
            <text class="item-label">自动备份</text>
            <text class="item-hint">聊天记录自动保存到本地</text>
          </view>
          <switch :checked="autoSave" @change="toggleAutoSave" color="#E8936A" />
        </view>
      </view>

      <!-- 数据管理 -->
      <text class="section-title">数据管理</text>
      <view class="group">
        <view class="item" @click="exportData">
          <view class="item-label-box">
            <text class="item-label">导出数据</text>
            <text class="item-hint">将聊天记录导出为文件</text>
          </view>
          <view class="item-right">
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
        <view class="item" @click="importData">
          <view class="item-label-box">
            <text class="item-label">导入数据</text>
            <text class="item-hint">从文件恢复聊天记录</text>
          </view>
          <view class="item-right">
            <image class="arrow-img" src="/static/icon-right-arrow.png" mode="aspectFit" />
          </view>
        </view>
      </view>

      <!-- 其他 -->
      <text class="section-title">其他</text>
      <view class="group">
        <view class="item" @click="clearMsgs">
          <text class="item-label warn-text">清除聊天记录</text>
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
      aiId: '',
      aiInfo: {},
      pinned: false,
      blocked: false,
      showAction: true,
      actionPosition: 'outside',
      actionLimit: 20,
      autoMessage: true,
      autoSave: true,
      memCount: 0,
      bgText: '默认'
    }
  },

  onLoad(opts) {
    this.aiId = opts.id || ''
    this.loadAiInfo()
    this.loadSettings()
    this.loadMemCount()
    this.loadBgText()
  },

  onShow() {
    this.loadAiInfo()
    this.loadMemCount()
    this.loadBgText()
  },

  methods: {

    loadAiInfo() {
      var self = this
      var raw = uni.getStorageSync('ai_list')
      if (raw) {
        try {
          var list = typeof raw === 'string' ? JSON.parse(raw) : raw
          if (Array.isArray(list)) {
            for (var i = 0; i < list.length; i++) {
              if (list[i].id === self.aiId) { self.aiInfo = list[i]; self.pinned = !!list[i].pinned; break }
            }
          }
        } catch(e) {}
      }
      if (!self.aiInfo.id) self.aiInfo = { id: self.aiId, name: '未知', bg: 'bg1' }
    },

    loadSettings() {
      var d = uni.getStorageSync('chat_settings_' + this.aiId)
      if (d) {
        try {
          var obj = typeof d === 'string' ? JSON.parse(d) : d
          this.showAction = obj.showAction !== false
          this.actionPosition = obj.actionPosition || 'outside'
          this.actionLimit = obj.actionLimit || 20
          this.blocked = obj.blocked || false
          this.autoSave = obj.autoSave !== false
          this.autoMessage = obj.autoMessage !== false
        } catch(e) {}
      }
    },

    saveSettings() {
      var data = {
        showAction: this.showAction,
        actionPosition: this.actionPosition,
        actionLimit: this.actionLimit,
        blocked: this.blocked,
        autoSave: this.autoSave,
        autoMessage: this.autoMessage
      }
      uni.setStorageSync('chat_settings_' + this.aiId, JSON.stringify(data))
    },

    togglePinned(e) {
      var self = this
      self.pinned = e.detail.value
      var raw = uni.getStorageSync('ai_list')
      if (raw) {
        try {
          var list = typeof raw === 'string' ? JSON.parse(raw) : raw
          if (Array.isArray(list)) {
            for (var i = 0; i < list.length; i++) {
              if (list[i].id === self.aiId) { list[i].pinned = self.pinned; break }
            }
            uni.setStorageSync('ai_list', JSON.stringify(list))
          }
        } catch(e) {}
      }
      uni.showToast({ title: self.pinned ? '已置顶' : '已取消置顶', icon: 'none' })
    },

    toggleBlocked(e) {
      var self = this
      var val = e.detail.value
      if (val) {
        uni.showModal({
          title: '加入黑名单', content: '加入黑名单后将无法收到该AI的消息，确定？',
          confirmText: '确定', confirmColor: '#E85D5D',
          success: function(r) {
            if (r.confirm) { self.blocked = true; self.saveSettings(); uni.showToast({ title: '已加入黑名单', icon: 'none' }) }
            else { self.blocked = false }
          }
        })
      } else {
        self.blocked = false; self.saveSettings(); uni.showToast({ title: '已取消黑名单', icon: 'none' })
      }
    },

    toggleAction(e) {
      this.showAction = e.detail.value
      this.saveSettings()
    },

    setActionPos(pos) {
      this.actionPosition = pos
      this.saveSettings()
    },

    setActionLimit(n) {
      this.actionLimit = n
      this.saveSettings()
    },

    toggleAutoMessage(e) { this.autoMessage = e.detail.value; this.saveSettings() },
    toggleAutoSave(e) { this.autoSave = e.detail.value; this.saveSettings() },

    goEdit() { uni.navigateTo({ url: '/pages/ai-add/ai-add?id=' + this.aiId }) },
    goAvatar() { uni.navigateTo({ url: '/pages/ai-add/ai-add?id=' + this.aiId + '&focus=avatar' }) },
    goMemory() { uni.navigateTo({ url: '/pages/memory/memory?id=' + this.aiId }) },
    goChatBg() { uni.navigateTo({ url: '/pages/chat-bg/chat-bg?id=' + this.aiId }) },

    loadMemCount() {
      var d = uni.getStorageSync('chat_memories_' + this.aiId)
      if (d) { try { var obj = typeof d === 'string' ? JSON.parse(d) : d; if (obj && obj.memories) this.memCount = obj.memories.length; else this.memCount = 0 } catch(e) { this.memCount = 0 } }
      else this.memCount = 0
    },

    loadBgText() {
      var d = uni.getStorageSync('chat_bg_' + this.aiId)
      if (d) { try { var bg = typeof d === 'string' ? JSON.parse(d) : d; if (bg.type === 'color') this.bgText = '纯色'; else if (bg.type === 'image') this.bgText = '自定义'; else this.bgText = '默认' } catch(e) { this.bgText = '默认' } }
      else this.bgText = '默认'
    },

    exportData() {
      var self = this
      uni.showActionSheet({
        itemList: ['导出到本地文件', '复制到剪贴板'],
        success: function(res) { if (res.tapIndex === 0) self.exportToFile(); else self.exportToClipboard() }
      })
    },

    exportToFile() {
      var self = this
      var data = self.collectAllData()
      if (!data) { uni.showToast({ title: '暂无数据', icon: 'none' }); return }
      var jsonStr = JSON.stringify(data, null, 2)
      var fileName = self.aiInfo.name + '_备份_' + self.getDateStr() + '.json'
      // #ifdef APP-PLUS
      plus.android.requestPermissions(['android.permission.WRITE_EXTERNAL_STORAGE', 'android.permission.READ_EXTERNAL_STORAGE'], function() { self.doExportFile(jsonStr, fileName) }, function() { uni.showToast({ title: '需要存储权限', icon: 'none' }) })
      // #endif
      // #ifndef APP-PLUS
      uni.setClipboardData({ data: jsonStr, success: function() { uni.showToast({ title: '已复制到剪贴板', icon: 'none' }) } })
      // #endif
    },

    doExportFile(jsonStr, fileName) {
      var dirName = '落雨备份', dirPath = '/storage/emulated/0/' + dirName
      function write(de) { de.getFile(fileName, { create: true }, function(fe) { fe.createWriter(function(w) { w.onwrite = function() { uni.showModal({ title: '导出成功', content: '文件保存到：\n' + dirPath + '/' + fileName, showCancel: false, confirmText: '知道了', confirmColor: '#E8936A' }) }; w.write(jsonStr) }) }) }
      plus.io.resolveLocalFileSystemURL('file://' + dirPath, function(de) { write(de) }, function() { plus.io.resolveLocalFileSystemURL('file:///storage/emulated/0/', function(root) { root.getDirectory(dirName, { create: true, exclusive: false }, function(de) { write(de) }) }) })
    },

    exportToClipboard() {
      var data = this.collectAllData()
      if (!data) { uni.showToast({ title: '暂无数据', icon: 'none' }); return }
      uni.setClipboardData({ data: JSON.stringify(data), success: function() { uni.showToast({ title: '已复制到剪贴板', icon: 'none' }) } })
    },

    importData() {
      var self = this
      uni.showActionSheet({
        itemList: ['从本地文件导入', '从剪贴板导入'],
        success: function(res) { if (res.tapIndex === 0) self.importFromFile(); else self.importFromClipboard() }
      })
    },

    importFromFile() {
      var self = this
      // #ifdef APP-PLUS
      uni.showModal({ title: '导入数据', content: '请将备份文件放入：\n/storage/emulated/0/落雨备份/\n\n确定后自动扫描。', confirmText: '开始扫描', confirmColor: '#E8936A', success: function(r) { if (r.confirm) self.scanAndImport() } })
      // #endif
      // #ifndef APP-PLUS
      uni.showModal({ title: '导入数据', content: '请使用从剪贴板导入。', showCancel: false, confirmText: '知道了', confirmColor: '#E8936A' })
      // #endif
    },

    scanAndImport() {
      var self = this
      plus.io.resolveLocalFileSystemURL('file:///storage/emulated/0/落雨备份', function(de) {
        var reader = de.createReader()
        reader.readEntries(function(entries) {
          var files = []
          for (var i = 0; i < entries.length; i++) { if (entries[i].name.indexOf('.json') !== -1 && entries[i].name.indexOf(self.aiInfo.name) !== -1) files.push({ name: entries[i].name, entry: entries[i] }) }
          if (files.length === 0) { uni.showToast({ title: '未找到备份文件', icon: 'none' }); return }
          var names = []; for (var j = 0; j < files.length; j++) names.push(files[j].name)
          if (names.length === 1) self.readAndImportFile(files[0].entry)
          else uni.showActionSheet({ itemList: names, success: function(r) { self.readAndImportFile(files[r.tapIndex].entry) } })
        })
      }, function() { uni.showToast({ title: '未找到备份目录', icon: 'none' }) })
    },

    readAndImportFile(fe) {
      var self = this
      fe.file(function(file) {
        var reader = new plus.io.FileReader()
        reader.onload = function(e) { self.processImportData(e.target.result) }
        reader.onerror = function() { uni.showToast({ title: '读取失败', icon: 'none' }) }
        reader.readAsText(file)
      })
    },

    importFromClipboard() {
      var self = this
      uni.getClipboardData({ success: function(r) { if (!r.data || !r.data.trim()) { uni.showToast({ title: '剪贴板为空', icon: 'none' }); return }; self.processImportData(r.data) } })
    },

    processImportData(text) {
      var self = this
      var data; try { data = JSON.parse(text) } catch(e) { uni.showToast({ title: '数据格式错误', icon: 'none' }); return }
      if (!data.app || data.app !== '落雨AI') { uni.showToast({ title: '不是落雨AI备份', icon: 'none' }); return }
      var summary = ''; if (data.ai) summary += 'AI：' + (data.ai.name || '') + '\n'; if (data.messages) summary += '消息：' + data.messages.length + '条\n'; if (data.memories && data.memories.memories) summary += '记忆：' + data.memories.memories.length + '条'
      uni.showModal({ title: '确认导入', content: summary + '\n导入将覆盖当前数据', confirmText: '导入', confirmColor: '#E8936A', success: function(r) { if (r.confirm) self.doImport(data) } })
    },

    doImport(data) {
      var self = this
      if (data.messages) uni.setStorageSync('chat_msgs_' + self.aiId, JSON.stringify(data.messages))
      if (data.memories) uni.setStorageSync('chat_memories_' + self.aiId, JSON.stringify(data.memories))
      if (data.settings) uni.setStorageSync('chat_settings_' + self.aiId, JSON.stringify(data.settings))
      if (data.chatBg) uni.setStorageSync('chat_bg_' + self.aiId, JSON.stringify(data.chatBg))
      uni.showToast({ title: '导入成功', icon: 'success' })
      setTimeout(function() { self.loadSettings(); self.loadMemCount(); self.loadBgText() }, 500)
    },

    collectAllData() {
      var self = this
      var rm = uni.getStorageSync('chat_msgs_' + self.aiId); var msgs = []; if (rm) { try { msgs = typeof rm === 'string' ? JSON.parse(rm) : rm } catch(e) {} }
      var rs = uni.getStorageSync('chat_settings_' + self.aiId); var settings = {}; if (rs) { try { settings = typeof rs === 'string' ? JSON.parse(rs) : rs } catch(e) {} }
      var rmem = uni.getStorageSync('chat_memories_' + self.aiId); var mem = { memories: [], times: [] }; if (rmem) { try { mem = typeof rmem === 'string' ? JSON.parse(rmem) : rmem } catch(e) {} }
      var rb = uni.getStorageSync('chat_bg_' + self.aiId); var bg = null; if (rb) { try { bg = typeof rb === 'string' ? JSON.parse(rb) : rb } catch(e) {} }
      return { version: '1.0', app: '落雨AI', exportTime: Date.now(), saveTime: Date.now(), ai: { id: self.aiId, name: self.aiInfo.name, type: self.aiInfo.type, gender: self.aiInfo.gender, prompt: self.aiInfo.prompt, personality: self.aiInfo.personality, background: self.aiInfo.background, habits: self.aiInfo.habits }, messages: msgs, memories: mem, settings: settings, chatBg: bg }
    },

    getDateStr() { var d = new Date(); return d.getFullYear() + '' + String(d.getMonth()+1).padStart(2,'0') + String(d.getDate()).padStart(2,'0') + '_' + String(d.getHours()).padStart(2,'0') + String(d.getMinutes()).padStart(2,'0') },

    clearMsgs() {
      var self = this
      uni.showModal({ title: '清除聊天记录', content: '确定清除所有聊天记录？不可恢复。', confirmText: '清除', confirmColor: '#E85D5D', success: function(r) { if (r.confirm) { uni.removeStorageSync('chat_msgs_' + self.aiId); uni.showToast({ title: '已清除', icon: 'none' }) } } })
    }
  }
}
</script>

<style>
page { background: #F2F2F2; }
.page { width: 100%; min-height: 100vh; background: #F2F2F2; }
.scroll { width: 100%; min-height: 100vh; }

.ai-card { margin: 12px; background: #FFF; border-radius: 14px; padding: 16px; display: flex; align-items: center; }
.ai-card:active { background: #F8F8F8; }
.card-avatar { width: 56px; height: 56px; border-radius: 50%; overflow: hidden; display: flex; align-items: center; justify-content: center; flex-shrink: 0; }
.card-avatar.bg1 { background: linear-gradient(135deg, #FFE5D0, #FFD4B8); }
.card-avatar.bg2 { background: linear-gradient(135deg, #D0E8FF, #B8D8FF); }
.card-avatar.bg3 { background: linear-gradient(135deg, #D5F5E3, #B8E8D0); }
.card-avatar.bg4 { background: linear-gradient(135deg, #F5D0FF, #E8B8FF); }
.card-avatar.bg5 { background: linear-gradient(135deg, #FFF3D0, #FFE8B8); }
.card-avatar.bg6 { background: linear-gradient(135deg, #FFD0D0, #FFB8B8); }
.card-avatar.bg7 { background: linear-gradient(135deg, #D0F0FF, #B8E8FF); }
.card-avatar-img { width: 56px; height: 56px; }
.card-info { flex: 1; margin-left: 12px; overflow: hidden; }
.card-name-row { display: flex; align-items: center; gap: 6px; }
.card-name { font-size: 17px; font-weight: 700; color: #2D2A26; }
.type-tag { background: #FFF0E8; border-radius: 4px; padding: 1px 6px; }
.type-tag.friend { background: #E8F5FF; }
.type-tag-text { font-size: 11px; color: #E8936A; font-weight: 600; }
.type-tag.friend .type-tag-text { color: #6BAFE8; }
.card-prompt { font-size: 13px; color: #B0B0B0; margin-top: 4px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }

.section-title { font-size: 13px; font-weight: 500; color: #888; padding: 20px 16px 8px; display: block; }
.group { background: #FFF; margin: 0 12px; border-radius: 14px; overflow: hidden; }
.item { display: flex; align-items: center; justify-content: space-between; padding: 14px 16px; }
.item:active { background: #F8F8F8; }
.item + .item { border-top: 0.5px solid #F0F0F0; }
.sub-item { background: #FAFAFA; padding-left: 28px; }
.item-label-box { display: flex; flex-direction: column; gap: 3px; }
.item-label { font-size: 15px; color: #2D2A26; }
.item-hint { font-size: 12px; color: #B0B0B0; }
.warn-text { color: #E85D5D !important; }
.item-right { display: flex; align-items: center; gap: 4px; }
.item-value { font-size: 13px; color: #B0B0B0; }
.arrow-img { width: 16px; height: 16px; }

.seg-bar { display: flex; background: #F0F0F0; border-radius: 8px; padding: 2px; gap: 2px; }
.seg-item { padding: 6px 14px; border-radius: 6px; transition: all 0.2s ease; }
.seg-item.active { background: #E8936A; }
.seg-item:active { opacity: 0.8; }
.seg-text { font-size: 13px; color: #888; white-space: nowrap; }
.seg-item.active .seg-text { color: #FFF; font-weight: 600; }

/* 神态预览 */
.preview-box { margin: 8px 12px 0; background: #FFF; border-radius: 14px; padding: 14px 16px; }
.preview-title { font-size: 12px; color: #B0B0B0; margin-bottom: 10px; display: block; }
.preview-row { display: flex; flex-direction: column; gap: 4px; align-items: flex-start; }
.preview-action { margin-bottom: 2px; }
.preview-action text { font-size: 12px; color: #B0A8A0; font-style: italic; }
.preview-bubble { background: #F5F5F5; border-radius: 4px 14px 14px 14px; padding: 10px 14px; }
.preview-bubble text { font-size: 14px; color: #2D2A26; line-height: 1.6; }
</style>
