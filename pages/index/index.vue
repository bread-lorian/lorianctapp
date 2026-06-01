<template>
  <view class="page">

    <scroll-view class="scroll" scroll-y>

      <view v-if="aiList.length > 0">
        <view class="ai-card" v-for="ai in aiList" :key="ai.id" @click="goChat(ai)" @longpress="onLongPress(ai)">
          <view class="ai-card-avatar">
            <view class="avatar" :class="ai.bg || 'bg1'">
              <image v-if="ai.avatar" :src="ai.avatar" mode="aspectFill" class="avatar-img" />
              <image v-else src="/static/default-avatar.png" mode="aspectFit" class="avatar-img" />
            </view>
          </view>
          <view class="ai-card-info">
            <view class="info-top">
              <text class="ai-name">{{ ai.name }}</text>
              <text class="ai-time">{{ getLastTime(ai.id) }}</text>
            </view>
            <text class="ai-preview">{{ getLastMsg(ai.id) }}</text>
          </view>
        </view>
      </view>

      <view class="empty" v-if="aiList.length === 0">
        <text class="empty-text">还没有AI，点击右上角 + 创建一个吧</text>
      </view>

      <view style="height: 20px;"></view>
    </scroll-view>

    <view class="popup-mask" v-if="showAdd" @click="showAdd = false"></view>
    <view class="add-popup" v-if="showAdd" :style="{ top: popupTop + 'px' }">
      <view class="popup-arrow"></view>
      <view class="popup-menu">
        <view class="popup-item" @click="addPartner">
          <text class="popup-icon"></text>
          <text class="popup-text">添加AI伴侣</text>
        </view>
        <view class="popup-item" @click="addFriend">
          <text class="popup-icon"></text>
          <text class="popup-text">添加AI朋友</text>
        </view>
      </view>
    </view>

    <view class="alert-mask" v-if="showAlert" @click="showAlert = false"></view>
    <view class="alert-panel" v-if="showAlert">
      <view class="alert-body">
        <text class="alert-title">提示</text>
        <text class="alert-msg">你已经有一个AI伴侣「{{ partnerName }}」了，AI伴侣只能添加一个。</text>
        <text class="alert-msg">是否将「{{ partnerName }}」转为朋友，然后添加新的AI伴侣？</text>
      </view>
      <view class="alert-footer">
        <view class="alert-btn cancel" @click="showAlert = false">
          <text class="alert-btn-text">取消</text>
        </view>
        <view class="alert-btn confirm" @click="confirmReplace">
          <text class="alert-btn-text">确定</text>
        </view>
      </view>
    </view>

    <view class="action-mask" v-if="showAction" @click="closeAction"></view>
    <view class="action-panel" v-if="showAction">
      <view class="action-header">
        <text class="action-title">{{ actionAI.name }}</text>
        <text class="action-type">{{ actionAI.type === 'partner' ? '当前：AI伴侣' : '当前：AI朋友' }}</text>
      </view>
      <view class="action-item" @click="goChatSetting">
        <text class="action-text">聊天设置</text>
      </view>
      <view class="action-item" @click="editAI">
        <text class="action-text">编辑信息</text>
      </view>
      <view class="action-item" v-if="actionAI.type === 'friend'" @click="becomePartner">
        <text class="action-text">转为AI伴侣</text>
      </view>
      <view class="action-item" v-if="actionAI.type === 'partner'" @click="becomeFriend">
        <text class="action-text">转为AI朋友</text>
      </view>
      <view class="action-item danger" @click="deleteAI">
        <text class="action-text">删除</text>
      </view>
      <view class="action-cancel" @click="closeAction">
        <text class="action-cancel-text">取消</text>
      </view>
    </view>

  </view>
</template>

<script>
export default {
  data() {
    return {
      aiList: [],
      showAdd: false,
      showAlert: false,
      partnerName: '',
      showAction: false,
      actionAI: {},
      navHeight: 44,
      popupTop: 34
    }
  },

  onLoad() {
    var self = this
    // #ifdef APP-PLUS
    var webView = self.$scope.$getAppWebview()
    webView.setTitleNViewButtonStyle(0, { text: '+' })
    var currentWebview = plus.webview.currentWebview()
    var style = currentWebview.getStyle()
    if (style && style.titleNView && style.titleNView.height) {
      self.navHeight = style.titleNView.height
      self.popupTop = self.navHeight - 8
    }
    // #endif
  },

  onShow() {
    this.loadList()
  },

  onNavigationBarButtonTap(btn) {
    if (btn.index === 0) {
      this.showAdd = !this.showAdd
    }
  },

  methods: {

    loadList() {
      var self = this
      var raw = uni.getStorageSync('ai_list')
      var list = []
      if (raw) {
        try {
          list = typeof raw === 'string' ? JSON.parse(raw) : raw
          if (!Array.isArray(list)) list = []
        } catch(e) { list = [] }
      }
      list.sort(function(a, b) {
        if (a.pinned && !b.pinned) return -1
        if (!a.pinned && b.pinned) return 1
        var ta = self.getLastMsgTime(a.id)
        var tb = self.getLastMsgTime(b.id)
        return tb - ta
      })
      self.aiList = list
    },

    saveList(list) {
      uni.setStorageSync('ai_list', JSON.stringify(list))
    },

    getPartner() {
      for (var i = 0; i < this.aiList.length; i++) {
        if (this.aiList[i].type === 'partner') return this.aiList[i]
      }
      return null
    },

    getLastMsgTime(aiId) {
      var raw = uni.getStorageSync('chat_msgs_' + aiId)
      if (raw) {
        try {
          var arr = typeof raw === 'string' ? JSON.parse(raw) : raw
          if (Array.isArray(arr) && arr.length > 0) return arr[arr.length - 1].time || 0
        } catch(e) {}
      }
      return 0
    },

    getLastTime(aiId) {
      var t = this.getLastMsgTime(aiId)
      if (!t) return ''
      var d = new Date(t)
      var now = new Date()
      var h = String(d.getHours()).padStart(2, '0')
      var m = String(d.getMinutes()).padStart(2, '0')
      if (d.toDateString() === now.toDateString()) return h + ':' + m
      var yesterday = new Date(now)
      yesterday.setDate(yesterday.getDate() - 1)
      if (d.toDateString() === yesterday.toDateString()) return '昨天'
      return (d.getMonth() + 1) + '/' + d.getDate()
    },

    getLastMsg(aiId) {
      var raw = uni.getStorageSync('chat_msgs_' + aiId)
      if (raw) {
        try {
          var arr = typeof raw === 'string' ? JSON.parse(raw) : raw
          if (Array.isArray(arr) && arr.length > 0) {
            var last = arr[arr.length - 1]
            if (last.type === 'emoji') return '[表情]'
            if (last.type === 'image') return '[图片]'
            return last.content || ''
          }
        } catch(e) {}
      }
      return '暂无消息'
    },

    addPartner() {
      var self = this
      self.showAdd = false
      var existing = self.getPartner()
      if (existing) {
        self.partnerName = existing.name
        self.showAlert = true
        return
      }
      uni.navigateTo({ url: '/pages/ai-add/ai-add?type=partner' })
    },

    confirmReplace() {
      var self = this
      self.showAlert = false
      var oldPartner = self.getPartner()
      if (oldPartner) {
        for (var i = 0; i < self.aiList.length; i++) {
          if (self.aiList[i].id === oldPartner.id) {
            self.aiList[i].type = 'friend'
            break
          }
        }
        self.saveList(self.aiList)
      }
      uni.navigateTo({ url: '/pages/ai-add/ai-add?type=partner' })
    },

    addFriend() {
      this.showAdd = false
      uni.navigateTo({ url: '/pages/ai-add/ai-add?type=friend' })
    },

    goChat(ai) {
      uni.navigateTo({
        url: '/pages/chat/chat?id=' + ai.id +
          '&name=' + encodeURIComponent(ai.name) +
          '&avatar=' + encodeURIComponent(ai.avatar || '') +
          '&prompt=' + encodeURIComponent(ai.prompt || '') +
          '&personality=' + encodeURIComponent(ai.personality || '') +
          '&gender=' + encodeURIComponent(ai.gender || '女') +
          '&background=' + encodeURIComponent(ai.background || '') +
          '&habits=' + encodeURIComponent(ai.habits || '') +
          '&bg=' + encodeURIComponent(ai.bg || 'bg1')
      })
    },

    onLongPress(ai) {
      this.actionAI = ai
      this.showAction = true
    },

    closeAction() {
      this.showAction = false
    },

    goChatSetting() {
      var self = this
      self.showAction = false
      uni.navigateTo({ url: '/pages/chat-setting/chat-setting?id=' + self.actionAI.id })
    },

    editAI() {
      var self = this
      self.showAction = false
      uni.navigateTo({ url: '/pages/ai-add/ai-add?id=' + self.actionAI.id })
    },

    becomePartner() {
      var self = this
      self.showAction = false
      var existing = self.getPartner()
      if (existing) {
        for (var i = 0; i < self.aiList.length; i++) {
          if (self.aiList[i].id === existing.id) {
            self.aiList[i].type = 'friend'
            break
          }
        }
      }
      for (var j = 0; j < self.aiList.length; j++) {
        if (self.aiList[j].id === self.actionAI.id) {
          self.aiList[j].type = 'partner'
          break
        }
      }
      self.saveList(self.aiList)
      self.loadList()
      uni.showToast({ title: '已转为AI伴侣', icon: 'none' })
    },

    becomeFriend() {
      var self = this
      self.showAction = false
      for (var i = 0; i < self.aiList.length; i++) {
        if (self.aiList[i].id === self.actionAI.id) {
          self.aiList[i].type = 'friend'
          break
        }
      }
      self.saveList(self.aiList)
      self.loadList()
      uni.showToast({ title: '已转为AI朋友', icon: 'none' })
    },

    deleteAI() {
      var self = this
      uni.showModal({
        title: '删除AI',
        content: '确定删除「' + self.actionAI.name + '」？所有聊天记录将被清除。',
        confirmText: '删除',
        confirmColor: '#E85D5D',
        success: function(r) {
          if (r.confirm) {
            var newList = []
            for (var i = 0; i < self.aiList.length; i++) {
              if (self.aiList[i].id !== self.actionAI.id) newList.push(self.aiList[i])
            }
            self.saveList(newList)
            uni.removeStorageSync('chat_msgs_' + self.actionAI.id)
            uni.removeStorageSync('chat_settings_' + self.actionAI.id)
            uni.removeStorageSync('chat_memories_' + self.actionAI.id)
            uni.removeStorageSync('chat_bg_' + self.actionAI.id)
            uni.removeStorageSync('custom_emoji_' + self.actionAI.id)
            self.showAction = false
            self.loadList()
            uni.showToast({ title: '已删除', icon: 'none' })
          }
        }
      })
    }
  }
}
</script>

<style>
page { background: #F2F2F2; }
.page { width: 100%; min-height: 100vh; background: #F2F2F2; }
.scroll { width: 100%; min-height: 100vh; }

.ai-card { display: flex; align-items: center; background: #FFF; padding: 12px 16px; }
.ai-card:active { background: #F8F8F8; }
.ai-card + .ai-card { border-top: 0.5px solid #F0F0F0; }
.ai-card-avatar { flex-shrink: 0; }
.avatar { width: 50px; height: 50px; border-radius: 50%; overflow: hidden; display: flex; align-items: center; justify-content: center; }
.avatar.bg1 { background: linear-gradient(135deg, #FFE5D0, #FFD4B8); }
.avatar.bg2 { background: linear-gradient(135deg, #D0E8FF, #B8D8FF); }
.avatar.bg3 { background: linear-gradient(135deg, #D5F5E3, #B8E8D0); }
.avatar.bg4 { background: linear-gradient(135deg, #F5D0FF, #E8B8FF); }
.avatar.bg5 { background: linear-gradient(135deg, #FFF3D0, #FFE8B8); }
.avatar.bg6 { background: linear-gradient(135deg, #FFD0D0, #FFB8B8); }
.avatar.bg7 { background: linear-gradient(135deg, #D0F0FF, #B8E8FF); }
.avatar-img { width: 50px; height: 50px; }
.ai-card-info { flex: 1; margin-left: 12px; overflow: hidden; }
.info-top { display: flex; align-items: center; justify-content: space-between; }
.ai-name { font-size: 16px; font-weight: 600; color: #2D2A26; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; max-width: 200px; }
.ai-time { font-size: 11px; color: #CCC; flex-shrink: 0; margin-left: 8px; }
.ai-preview { font-size: 13px; color: #B0B0B0; margin-top: 4px; display: block; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }

.empty { display: flex; align-items: center; justify-content: center; padding: 120px 30px; }
.empty-text { font-size: 14px; color: #CCC; }

.popup-mask { position: fixed; top: 0; left: 0; right: 0; bottom: 0; background: transparent; z-index: 90; }
.add-popup { position: fixed; right: 8px; z-index: 91; }
.popup-arrow { width: 0; height: 0; border-left: 8px solid transparent; border-right: 8px solid transparent; border-bottom: 8px solid #3A3A3A; margin-left: auto; margin-right: 14px; }
.popup-menu { background: #3A3A3A; border-radius: 8px; overflow: hidden; min-width: 160px; }
.popup-item { display: flex; align-items: center; padding: 14px 18px; gap: 10px; }
.popup-item:active { background: #4A4A4A; }
.popup-item + .popup-item { border-top: 0.5px solid #4A4A4A; }
.popup-icon { font-size: 17px; }
.popup-text { font-size: 15px; color: #FFF; }

.alert-mask { position: fixed; top: 0; left: 0; right: 0; bottom: 0; background: rgba(0,0,0,0.4); z-index: 100; }
.alert-panel { position: fixed; top: 50%; left: 50%; width: 300px; margin-left: -150px; margin-top: -100px; background: #FFF; border-radius: 14px; z-index: 101; }
.alert-body { padding: 24px 20px 16px; text-align: center; }
.alert-title { font-size: 17px; font-weight: 700; color: #2D2A26; display: block; margin-bottom: 10px; }
.alert-msg { font-size: 14px; color: #666; line-height: 1.6; display: block; margin-bottom: 6px; }
.alert-footer { display: flex; border-top: 0.5px solid #F0F0F0; }
.alert-btn { flex: 1; padding: 14px; text-align: center; }
.alert-btn:active { background: #F5F5F5; }
.alert-btn + .alert-btn { border-left: 0.5px solid #F0F0F0; }
.alert-btn.cancel .alert-btn-text { color: #999; font-size: 16px; }
.alert-btn.confirm .alert-btn-text { color: #E8936A; font-size: 16px; font-weight: 600; }

.action-mask { position: fixed; top: 0; left: 0; right: 0; bottom: 0; background: rgba(0,0,0,0.4); z-index: 100; }
.action-panel { position: fixed; left: 0; right: 0; bottom: 0; background: #FFF; border-radius: 16px 16px 0 0; z-index: 101; padding-bottom: env(safe-area-inset-bottom, 0px); }
.action-header { padding: 16px 20px 12px; border-bottom: 0.5px solid #F0F0F0; text-align: center; }
.action-title { font-size: 16px; font-weight: 600; color: #2D2A26; display: block; }
.action-type { font-size: 12px; color: #B0B0B0; margin-top: 4px; display: block; }
.action-item { padding: 16px 20px; text-align: center; }
.action-item:active { background: #F5F5F5; }
.action-item + .action-item { border-top: 0.5px solid #F0F0F0; }
.action-text { font-size: 16px; color: #2D2A26; }
.action-item.danger .action-text { color: #E85D5D; }
.action-cancel { margin: 8px 12px 12px; background: #F0F0F0; border-radius: 12px; padding: 14px; text-align: center; }
.action-cancel:active { background: #E8E8E8; }
.action-cancel-text { font-size: 16px; color: #666; font-weight: 600; }
</style>