<template>
  <view class="page" :style="bgStyle">

    <scroll-view class="chat-list" scroll-y :scroll-into-view="toId" scroll-with-animation>

      <view v-for="(msg, idx) in msgs" :key="msg.id">
        <view class="time-tag" v-if="showTime(idx)">
          <text>{{ fmtTime(msg.time) }}</text>
        </view>

        <view class="row ai" v-if="msg.role === 'ai'">
          <view class="msg-avatar-box">
            <view class="msg-avatar" :class="aiBg || 'bg1'">
              <image v-if="aiAvatarPath" :src="aiAvatarPath" mode="aspectFill" class="msg-avatar-img" />
              <image v-else src="/static/default-avatar.png" mode="aspectFit" class="msg-avatar-img" />
            </view>
          </view>
          <view class="bubble-box">
            <view class="action" v-if="msg.action && settings.showAction && settings.actionPosition === 'outside'">
              <text>*{{ truncateAction(msg.action) }}*</text>
            </view>
            <view class="bubble ai-bubble">
              <text v-if="msg.action && settings.showAction && settings.actionPosition === 'inside'">（{{ msg.action }}）{{ msg.content }}</text>
              <text v-else>{{ msg.content }}</text>
            </view>
          </view>
        </view>

        <view class="row user" v-if="msg.role === 'user'">
          <view class="bubble-box user-bubble-box">
            <view class="bubble user-bubble" v-if="!msg.type || msg.type === 'text'">
              <text>{{ msg.content }}</text>
            </view>
            <view class="image-wrap" v-else-if="msg.type === 'image'">
              <image :src="msg.imageUrl" mode="widthFix" class="msg-image" @click="previewImage(msg.imageUrl)" />
              <view class="image-caption-box" v-if="msg.content">
                <text class="image-caption">{{ msg.content }}</text>
              </view>
            </view>
            <view class="emoji-wrap" v-else-if="msg.type === 'emoji'">
              <image :src="msg.imageUrl" mode="aspectFit" class="emoji-send-img" />
            </view>
          </view>
          <view class="msg-avatar-box">
            <view class="msg-avatar user-avatar-bg">
              <image v-if="userAvatarPath" :src="userAvatarPath" mode="aspectFill" class="msg-avatar-img" />
              <image v-else src="/static/default-user-avatar.png" mode="aspectFit" class="msg-avatar-img" />
            </view>
          </view>
        </view>
      </view>

      <view class="row ai" v-if="loading">
        <view class="msg-avatar-box">
          <view class="msg-avatar" :class="aiBg || 'bg1'">
            <image v-if="aiAvatarPath" :src="aiAvatarPath" mode="aspectFill" class="msg-avatar-img" />
            <image v-else src="/static/default-avatar.png" mode="aspectFit" class="msg-avatar-img" />
          </view>
        </view>
        <view class="bubble-box">
          <view class="bubble ai-bubble typing">
            <text>正在输入...</text>
          </view>
        </view>
      </view>

      <view id="bottom" style="height: 20px;"></view>
    </scroll-view>

    <view class="memory-toast" v-if="showMemoryToast" :class="{ fadeOut: memoryToastFade }">
      <text class="toast-icon"></text>
      <text class="toast-text">{{ memoryToastText }}</text>
    </view>

    <view class="emoji-panel" v-if="showEmoji">
      <view class="emoji-tabs">
        <view class="emoji-tab" :class="{ active: emojiTab === 'default' }" @click="emojiTab = 'default'">
          <text class="emoji-tab-text">默认表情</text>
        </view>
        <view class="emoji-tab" :class="{ active: emojiTab === 'custom' }" @click="emojiTab = 'custom'">
          <text class="emoji-tab-text">我的表情</text>
        </view>
      </view>
      <scroll-view class="emoji-grid" scroll-y v-if="emojiTab === 'default'">
        <view class="emoji-grid-inner">
          <view class="emoji-item" v-for="e in emojiList" :key="e" @click="insertEmoji(e)">
            <text>{{ e }}</text>
          </view>
        </view>
      </scroll-view>
      <scroll-view class="emoji-grid" scroll-y v-if="emojiTab === 'custom'">
        <view class="emoji-grid-inner">
          <!-- 添加按钮固定在最前面 -->
          <view class="emoji-item add-emoji-btn" @click="addCustomEmoji">
            <text class="add-emoji-icon">+</text>
          </view>
          <!-- 已有表情在后面 -->
          <view class="emoji-item custom-emoji-item" v-for="(ce, idx) in customEmoji" :key="'ce_' + idx" @click="sendCustomEmoji(ce)" @longpress="deleteCustomEmoji(idx)">
            <image :src="ce" mode="aspectFit" class="custom-emoji-img" />
          </view>
        </view>
        <view v-if="customEmoji.length === 0" class="empty-emoji-hint">
          <text class="empty-emoji-text">点击+添加你自己的表情包</text>
        </view>
      </scroll-view>
    </view>

    <view class="input-bar">
      <view class="emoji-btn" @click="toggleEmoji">
        <image class="emoji-icon" src="/static/icon-emoji.png" mode="aspectFit" />
      </view>
      <view class="img-btn" @click="chooseAndSendImage">
        <image class="img-icon" src="/static/icon-image.png" mode="aspectFit" />
      </view>
      <input class="input-field" v-model="inputText" placeholder="跟TA想说什么..." :focus="wantFocus" @blur="onBlur" @confirm="send" />
      <view class="send-space"></view>
      <view class="send-btn" :class="{ active: inputText.trim() }" @click="send">
        <text>发送</text>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      aiId: '',
      aiName: '',
      aiAvatar: '',
      aiPrompt: '',
      aiPersonality: '',
      aiGender: '',
      aiBackground: '',
      aiHabits: '',
      aiBg: 'bg1',
      msgs: [],
      inputText: '',
      loading: false,
      toId: '',
      apiKey: '',
      apiUrl: '',
      modelName: '',
      visionApiUrl: '',
      visionModelName: '',
      useVision: false,
      visionSupport: false,
      settings: {
        showAction: true,
        actionPosition: 'outside',
        actionLimit: 20,
        blocked: false,
        autoSave: true,
        autoMessage: true
      },
      memories: [],
      memTimes: [],
      autoTimer: null,
      showEmoji: false,
      emojiTab: 'default',
      emojiList: ['😊','😂','🥰','😍','😘','😋','😜','🤗','😢','😭','😡','🥺','😳','🤔','😏','🙄','😅','🤣','😌','😇','🤩','😚','😃','😄','❤️','💕','💖','💗','💓','💞','💘','💝','👋','👍','👎','✌️','🤞','💪','🙏','👏','🌹','🌸','🌙','⭐','☀️','🌈','🍀','🎵','🐱','🐶','🐰','🦊','🐻','🐼','🐸','🦋'],
      customEmoji: [],
      wantFocus: false,
      isSending: false,
      chatBg: '',
      chatBgType: 'default',
      chatBgColor: '#F5F5F5',
      showMemoryToast: false,
      memoryToastFade: false,
      memoryToastText: '',
      aiAvatarPath: '',
      userAvatarPath: ''
    }
  },

  computed: {
    bgStyle() {
      if (this.chatBgType === 'image' && this.chatBg) {
        return {
          backgroundImage: 'url(' + this.chatBg + ')',
          backgroundSize: 'cover',
          backgroundPosition: 'center'
        }
      }
      if (this.chatBgType === 'color' && this.chatBgColor) {
        return { background: this.chatBgColor }
      }
      return { background: '#F5F5F5' }
    }
  },

  onLoad(opts) {
    var self = this
    self.aiId = opts.id || '1'
    self.aiName = decodeURIComponent(opts.name || 'AI')
    self.aiAvatar = decodeURIComponent(opts.avatar || '')
    self.aiPrompt = decodeURIComponent(opts.prompt || '')
    self.aiPersonality = decodeURIComponent(opts.personality || '')
    self.aiGender = decodeURIComponent(opts.gender || '女')
    self.aiBackground = decodeURIComponent(opts.background || '')
    self.aiHabits = decodeURIComponent(opts.habits || '')
    self.aiBg = decodeURIComponent(opts.bg || 'bg1')
    uni.setNavigationBarTitle({ title: self.aiName })
    // #ifdef APP-PLUS
    self.addAiBadge()
    // #endif
    self.loadApi()
    self.loadMsgs()
    self.loadSettings()
    self.loadMemories()
    self.loadBg()
    self.loadAvatars()
    self.loadCustomEmoji()
    self.startAuto()
    setTimeout(function() {
      if (self.settings.autoSave) self.autoSaveToFile()
    }, 2000)
  },

  onShow() {
    var self = this
    self.loadApi()
    self.loadSettings()
    self.loadMemories()
    self.loadBg()
    self.loadAvatars()
    self.loadCustomEmoji()
    setTimeout(function() {
      if (self.settings.autoSave) self.autoSaveToFile()
    }, 1000)
  },

  onNavigationBarButtonTap(btn) {
    if (btn.index === 0) {
      uni.navigateTo({ url: '/pages/chat-setting/chat-setting?id=' + this.aiId })
    }
  },

  onUnload() {
    if (this.autoTimer) {
      clearInterval(this.autoTimer)
      this.autoTimer = null
    }
    if (this.settings.autoSave) this.autoSaveToFile()
    // #ifdef APP-PLUS
    try {
      var b = plus.nativeObj.View.getViewById('aiBadge')
      if (b) b.close()
    } catch(e) {}
    // #endif
  },

  methods: {

    truncateAction(text) {
      if (!text) return ''
      var limit = this.settings.actionLimit || 20
      return text.length <= limit ? text : text.substring(0, limit) + '...'
    },

    addAiBadge() {
      var self = this
      setTimeout(function() {
        try {
          var old = plus.nativeObj.View.getViewById('aiBadge')
          if (old) old.close()
          var ws = plus.webview.currentWebview()
          var sbH = plus.navigator.getStatusbarHeight()
          var sW = plus.screen.resolutionWidth
          var bX = sW / 2 + self.aiName.length * 17 / 2 + 5
          var bY = sbH + 12
          var bv = new plus.nativeObj.View('aiBadge', {
            top: bY + 'px',
            left: bX + 'px',
            width: '72px',
            height: '20px'
          })
          bv.draw([
            {
              tag: 'rect',
              rectStyles: { color: '#E8F5E9', radius: '4px' },
              position: { top: '0px', left: '0px', width: '72px', height: '20px' }
            },
            {
              tag: 'font',
              text: '由AI生成',
              textStyles: { size: '10px', color: '#4CAF50', align: 'center' },
              position: { top: '0px', left: '0px', width: '72px', height: '20px' }
            }
          ])
          ws.append(bv)
        } catch(e) {}
      }, 800)
    },

    loadAvatars() {
      var self = this
      var raw = uni.getStorageSync('ai_list')
      if (raw) {
        try {
          var list = typeof raw === 'string' ? JSON.parse(raw) : raw
          if (Array.isArray(list)) {
            for (var i = 0; i < list.length; i++) {
              if (list[i].id === self.aiId) {
                self.aiAvatarPath = list[i].avatar || ''
                self.aiBg = list[i].bg || 'bg1'
                break
              }
            }
          }
        } catch(e) {}
      }
      if (self.aiAvatar && !self.aiAvatarPath) self.aiAvatarPath = self.aiAvatar
      var p = uni.getStorageSync('user_profile')
      if (p) {
        try {
          var obj = typeof p === 'string' ? JSON.parse(p) : p
          self.userAvatarPath = obj.avatar || ''
        } catch(e) {}
      }
    },

    onBlur() {
      var self = this
      if (self.isSending) {
        setTimeout(function() { self.wantFocus = true }, 150)
        return
      }
      self.wantFocus = false
    },

    toggleEmoji() {
      if (this.showEmoji) {
        this.showEmoji = false
        this.isSending = true
        var self = this
        setTimeout(function() {
          self.wantFocus = true
          self.isSending = false
        }, 100)
      } else {
        this.showEmoji = true
        this.wantFocus = false
      }
    },

    insertEmoji(e) {
      this.inputText = this.inputText + e
    },

    // ========== 收藏表情管理（持久化存储） ==========
    loadCustomEmoji() {
      var self = this
      var d = uni.getStorageSync('custom_emoji_' + self.aiId)
      if (d) {
        try {
          var arr = typeof d === 'string' ? JSON.parse(d) : d
          if (Array.isArray(arr)) {
            // 过滤掉已经不存在的临时文件路径
            var valid = []
            for (var i = 0; i < arr.length; i++) {
              if (arr[i] && arr[i].length > 0) valid.push(arr[i])
            }
            self.customEmoji = valid
          }
        } catch(e) {}
      }
    },

    saveCustomEmoji() {
      var self = this
      uni.setStorageSync('custom_emoji_' + self.aiId, JSON.stringify(self.customEmoji))
    },

    addCustomEmoji() {
      var self = this
      if (self.customEmoji.length >= 50) {
        uni.showToast({ title: '最多50个', icon: 'none' })
        return
      }
      uni.chooseImage({
        count: 1,
        sizeType: ['compressed'],
        success: function(res) {
          var tempPath = res.tempFilePaths[0]
          // 复制到永久目录，防止临时文件被清理后消失
          self.copyToPermanent(tempPath, function(permanentPath) {
            if (permanentPath) {
              self.customEmoji.push(permanentPath)
            } else {
              self.customEmoji.push(tempPath)
            }
            self.saveCustomEmoji()
            uni.showToast({ title: '已添加', icon: 'none' })
          })
        }
      })
    },

    // 把临时文件复制到永久目录
    copyToPermanent(tempPath, callback) {
      // #ifdef APP-PLUS
      var self = this
      var fileName = 'emoji_' + self.aiId + '_' + Date.now() + '.jpg'
      var dirPath = '_doc/emoji_cache/'
      plus.io.resolveLocalFileSystemURL(dirPath, function(dirEntry) {
        self.doCopy(tempPath, dirEntry, fileName, callback)
      }, function() {
        plus.io.resolveLocalFileSystemURL('_doc/', function(docEntry) {
          docEntry.getDirectory('emoji_cache', { create: true, exclusive: false }, function(dirEntry) {
            self.doCopy(tempPath, dirEntry, fileName, callback)
          }, function() { callback(null) })
        }, function() { callback(null) })
      })
      // #endif
      // #ifndef APP-PLUS
      callback(tempPath)
      // #endif
    },

    doCopy(srcPath, dirEntry, fileName, callback) {
      var srcUrl = srcPath
      if (srcUrl.indexOf('file://') !== 0 && srcUrl.charAt(0) === '/') {
        srcUrl = 'file://' + srcUrl
      }
      plus.io.resolveLocalFileSystemURL(srcUrl, function(srcEntry) {
        srcEntry.copyTo(dirEntry, fileName, function(destEntry) {
          callback(destEntry.fullPath)
        }, function() { callback(null) })
      }, function() { callback(null) })
    },

    deleteCustomEmoji(idx) {
      var self = this
      uni.showActionSheet({
        itemList: ['删除该表情'],
        success: function(res) {
          if (res.tapIndex === 0) {
            var path = self.customEmoji[idx]
            self.customEmoji.splice(idx, 1)
            self.saveCustomEmoji()
            // 删除永久文件
            if (path && path.indexOf('emoji_cache') !== -1) {
              self.deletePermanentFile(path)
            }
            uni.showToast({ title: '已删除', icon: 'none' })
          }
        }
      })
    },

    deletePermanentFile(path) {
      try {
        var url = path
        if (url.indexOf('file://') !== 0 && url.charAt(0) === '/') url = 'file://' + url
        plus.io.resolveLocalFileSystemURL(url, function(entry) {
          entry.remove(function() {}, function() {})
        }, function() {})
      } catch(e) {}
    },

    sendCustomEmoji(path) {
      var self = this
      if (self.settings.blocked) {
        uni.showToast({ title: '已拉黑', icon: 'none' })
        return
      }
      if (!self.apiKey) {
        uni.showToast({ title: '请先配置API Key', icon: 'none' })
        return
      }
      self.showEmoji = false

      self.msgs.push({
        id: 'm_' + Date.now(),
        role: 'user',
        content: '',
        type: 'emoji',
        imageUrl: path,
        time: Date.now()
      })
      self.saveMsgs()
      self.scrollEnd()
      if (self.settings.autoSave) self.autoSaveToFile()

      // 检查视觉支持
      self.loadApi()
      if (!self.useVision || !self.visionSupport) {
        self.msgs.push({
          id: 'm_' + Date.now(),
          role: 'ai',
          action: null,
          content: '当前AI不支持识别表情包，请在API设置中切换支持视觉的提供商。',
          type: 'text',
          time: Date.now()
        })
        self.saveMsgs()
        self.scrollEnd()
        return
      }

      self.compressAndSendEmoji(path)
    },

    compressAndSendEmoji(path) {
      var self = this
      uni.compressImage({
        src: path,
        quality: 50,
        success: function(res) {
          self.imageToBase64(res.tempFilePath, function(b64) {
            if (b64) {
              self.callAI('用户发了一张表情包', b64, true, 'emoji')
            } else {
              self.callAI('用户发了一张表情包', null, false, 'emoji')
            }
          })
        },
        fail: function() {
          self.imageToBase64(path, function(b64) {
            if (b64) {
              self.callAI('用户发了一张表情包', b64, true, 'emoji')
            } else {
              self.callAI('用户发了一张表情包', null, false, 'emoji')
            }
          })
        }
      })
    },

    chooseAndSendImage() {
      var self = this
      if (self.settings.blocked) {
        uni.showToast({ title: '已拉黑', icon: 'none' })
        return
      }
      if (!self.apiKey) {
        uni.showToast({ title: '请先配置API Key', icon: 'none' })
        return
      }
      uni.chooseImage({
        count: 1,
        sizeType: ['compressed'],
        success: function(res) {
          var imgPath = res.tempFilePaths[0]
          var userText = self.inputText.trim()
          self.inputText = ''
          self.showEmoji = false

          self.msgs.push({
            id: 'm_' + Date.now(),
            role: 'user',
            content: userText,
            type: 'image',
            imageUrl: imgPath,
            time: Date.now()
          })
          self.saveMsgs()
          self.scrollEnd()
          if (self.settings.autoSave) self.autoSaveToFile()

          // 检查视觉支持
          self.loadApi()
          if (!self.useVision || !self.visionSupport) {
            self.msgs.push({
              id: 'm_' + Date.now(),
              role: 'ai',
              action: null,
              content: '当前AI不支持识别图片，请在API设置中切换支持视觉的提供商。',
              type: 'text',
              time: Date.now()
            })
            self.saveMsgs()
            self.scrollEnd()
            return
          }

          self.compressAndSendImage(imgPath, userText)
        }
      })
    },

    compressAndSendImage(imgPath, userText) {
      var self = this
      uni.compressImage({
        src: imgPath,
        quality: 50,
        success: function(res) {
          self.imageToBase64(res.tempFilePath, function(b64) {
            self.callAI(userText || '', b64, true, 'image')
          })
        },
        fail: function() {
          self.imageToBase64(imgPath, function(b64) {
            self.callAI(userText || '', b64, true, 'image')
          })
        }
      })
    },

    imageToBase64(path, callback) {
      // #ifdef APP-PLUS
      var url = path
      if (url.indexOf('file://') !== 0 && url.charAt(0) === '/') {
        url = 'file://' + url
      }
      plus.io.resolveLocalFileSystemURL(url, function(entry) {
        entry.file(function(file) {
          var reader = new plus.io.FileReader()
          reader.onloadend = function(e) {
            callback(e.target.result)
          }
          reader.onerror = function() {
            callback(null)
          }
          reader.readAsDataURL(file)
        }, function() {
          callback(null)
        })
      }, function() {
        callback(null)
      })
      // #endif
      // #ifndef APP-PLUS
      callback(null)
      // #endif
    },

    previewImage(url) {
      uni.previewImage({ urls: [url], current: url })
    },

    showToast(text) {
      var self = this
      self.memoryToastText = text
      self.showMemoryToast = true
      self.memoryToastFade = false
      setTimeout(function() {
        self.memoryToastFade = true
        setTimeout(function() {
          self.showMemoryToast = false
          self.memoryToastFade = false
        }, 500)
      }, 1500)
    },

    loadApi() {
      var c = uni.getStorageSync('api_config')
      if (c) {
        try {
          var obj = typeof c === 'string' ? JSON.parse(c) : c
          this.apiKey = obj.apiKey || ''
          this.apiUrl = obj.apiUrl || 'https://api.deepseek.com'
          this.modelName = obj.modelName || 'deepseek-chat'
          this.visionSupport = obj.visionSupport || false
          this.useVision = obj.useVision || false
          this.visionModelName = obj.visionModelName || ''
          this.visionApiUrl = obj.visionApiUrl || obj.apiUrl || 'https://api.deepseek.com'
        } catch(e) {}
      }
    },

    loadMsgs() {
      var d = uni.getStorageSync('chat_msgs_' + this.aiId)
      if (d) {
        try {
          var arr = typeof d === 'string' ? JSON.parse(d) : d
          if (Array.isArray(arr)) this.msgs = arr
        } catch(e) {}
      }
      this.scrollEnd()
    },

    saveMsgs() {
      uni.setStorageSync('chat_msgs_' + this.aiId, JSON.stringify(this.msgs))
    },

    loadSettings() {
      var d = uni.getStorageSync('chat_settings_' + this.aiId)
      if (d) {
        try {
          var obj = typeof d === 'string' ? JSON.parse(d) : d
          this.settings.showAction = obj.showAction !== false
          this.settings.actionPosition = obj.actionPosition || 'outside'
          this.settings.actionLimit = obj.actionLimit || 20
          this.settings.blocked = obj.blocked || false
          this.settings.autoSave = obj.autoSave !== false
          this.settings.autoMessage = obj.autoMessage !== false
        } catch(e) {}
      }
    },

    loadMemories() {
      var d = uni.getStorageSync('chat_memories_' + this.aiId)
      if (d) {
        try {
          var obj = typeof d === 'string' ? JSON.parse(d) : d
          if (obj && obj.memories) {
            this.memories = obj.memories
            this.memTimes = obj.times || []
          } else if (Array.isArray(obj)) {
            this.memories = obj
            this.memTimes = []
          }
        } catch(e) {
          this.memories = []
          this.memTimes = []
        }
      } else {
        this.memories = []
        this.memTimes = []
      }
    },

    saveMemories() {
      uni.setStorageSync('chat_memories_' + this.aiId, JSON.stringify({
        memories: this.memories,
        times: this.memTimes || []
      }))
    },

    loadBg() {
      var d = uni.getStorageSync('chat_bg_' + this.aiId)
      if (d) {
        try {
          var bg = typeof d === 'string' ? JSON.parse(d) : d
          if (bg.type === 'color') {
            this.chatBgType = 'color'
            this.chatBgColor = bg.value
            this.chatBg = ''
          } else if (bg.type === 'image') {
            this.chatBgType = 'image'
            this.chatBg = bg.value
          } else {
            this.chatBgType = 'default'
          }
        } catch(e) {
          this.chatBgType = 'default'
        }
      } else {
        this.chatBgType = 'default'
        this.chatBg = ''
        this.chatBgColor = '#F5F5F5'
      }
    },

    scrollEnd() {
      var self = this
      self.toId = ''
      setTimeout(function() { self.toId = 'bottom' }, 50)
    },

    send() {
      var self = this
      var text = self.inputText.trim()
      if (!text) return
      if (self.settings.blocked) {
        uni.showToast({ title: '已拉黑', icon: 'none' })
        return
      }
      if (!self.apiKey) {
        uni.showToast({ title: '请先配置API Key', icon: 'none' })
        return
      }
      self.showEmoji = false
      self.msgs.push({
        id: 'm_' + Date.now(),
        role: 'user',
        content: text,
        type: 'text',
        time: Date.now()
      })
      self.inputText = ''
      self.saveMsgs()
      self.scrollEnd()
      self.extractMemory(text)
      self.callAI(text, null, false, 'text')
      if (self.settings.autoSave) self.autoSaveToFile()
      self.isSending = true
      self.wantFocus = false
      setTimeout(function() {
        self.wantFocus = true
        setTimeout(function() { self.isSending = false }, 200)
      }, 50)
    },

    callAI(userMsg, imageBase64, isImage, contentType) {
      var self = this

      self.loadSettings()
      self.loadApi()

      contentType = contentType || 'text'
      self.loading = true
      self.scrollEnd()

      var useApi = self.apiUrl
      var useModel = self.modelName
      if (isImage && self.useVision && self.visionSupport && self.visionModelName) {
        useApi = self.visionApiUrl || self.apiUrl
        useModel = self.visionModelName
      }

      var ctxLimit = isImage ? 10 : 20
      var ctx = []
      var start = Math.max(0, self.msgs.length - ctxLimit)
      for (var i = start; i < self.msgs.length; i++) {
        var m = self.msgs[i]
        if (m.role === 'user') {
          if (m.type === 'emoji') ctx.push({ role: 'user', content: '[表情]' })
          else if (m.type === 'image') ctx.push({ role: 'user', content: '[图片] ' + (m.content || '') })
          else ctx.push({ role: 'user', content: m.content })
        } else {
          ctx.push({ role: 'assistant', content: m.content })
        }
      }

      var userMessage
      if (imageBase64) {
        userMessage = {
          role: 'user',
          content: [
            { type: 'image_url', image_url: { url: imageBase64 } },
            { type: 'text', text: userMsg || '请描述你看到的内容' }
          ]
        }
      } else {
        userMessage = { role: 'user', content: userMsg }
      }

      var allMsgs = [{ role: 'system', content: self.buildPrompt() }].concat(ctx).concat([userMessage])

      uni.request({
        url: useApi + '/v1/chat/completions',
        method: 'POST',
        header: {
          'Authorization': 'Bearer ' + self.apiKey,
          'Content-Type': 'application/json'
        },
        data: {
          model: useModel,
          messages: allMsgs,
          temperature: 0.8,
          max_tokens: 1000
        },
        success: function(res) {
          if (res.statusCode === 200 && res.data && res.data.choices) {
            var reply = res.data.choices[0].message.content
            var parsed = self.parseReply(reply)
            self.msgs.push({
              id: 'm_' + Date.now(),
              role: 'ai',
              action: parsed.action,
              content: parsed.content,
              type: 'text',
              time: Date.now()
            })
            self.saveMsgs()
            if (self.settings.autoSave) self.autoSaveToFile()
          } else if (isImage && (res.statusCode === 400 || res.statusCode === 415)) {
            var errMsg = '该模型不支持识别图片，请在API设置中配置视觉模型。'
            if (contentType === 'emoji') {
              errMsg = '该模型不支持识别表情包，请在API设置中配置视觉模型。'
            }
            self.msgs.push({
              id: 'm_' + Date.now(),
              role: 'ai',
              action: null,
              content: errMsg,
              type: 'text',
              time: Date.now()
            })
            self.saveMsgs()
          } else {
            var errMsg2 = 'AI回复失败'
            if (res.data && res.data.error && res.data.error.message) {
              errMsg2 = res.data.error.message
            }
            uni.showToast({ title: errMsg2, icon: 'none' })
          }
        },
        fail: function() {
          uni.showToast({ title: '网络错误', icon: 'none' })
        },
        complete: function() {
          self.loading = false
          self.scrollEnd()
        }
      })
    },

    buildPrompt() {
      var p = '你是' + this.aiName + '，' + this.aiPersonality + '。\n'
      p += '性别：' + this.aiGender + '\n'
      p += '人设：' + this.aiPrompt + '\n'
      if (this.aiBackground) p += '背景：' + this.aiBackground + '\n'
      if (this.aiHabits) p += '习惯用语：' + this.aiHabits + '\n'
      if (this.memories.length > 0) {
        p += '\n你记得的关于用户的重要信息：\n'
        for (var i = 0; i < this.memories.length; i++) {
          p += '- ' + this.memories[i] + '\n'
        }
      }
      if (this.settings.showAction) {
        if (this.settings.actionPosition === 'outside') {
          p += '\n你必须按照此格式回答！'
          p += '\n回复格式：先写神态动作用*星号*包裹（不超过' + (this.settings.actionLimit || 20) + '个字），再写对话内容。'
          p += '\n示例：*微微低头，脸红了* 我也很想你呀'
          p += '\n任何动作描写、第三人称描写都必须在星号内！绝不能逾越！'
        } else {
          p += '\n回复格式：在对话内容开头用（）包裹神态动作，然后写对话内容。'
          p += '\n示例：（微微低头，脸红了）我也很想你呀'
        }
      } else {
        p += '\n注意：直接回复对话内容即可，不要使用*星号*或（）包裹任何神态动作描述，不要输出任何动作描写。'
      }
      return p
    },

    parseReply(text) {
      if (!this.settings.showAction) {
        return { action: null, content: text.trim() }
      }
      if (this.settings.actionPosition === 'inside') {
        var m = text.match(/（(.+?)）/)
        if (m) return { action: m[1], content: text.replace(/（.+?）/, '').trim() }
      }
      var m2 = text.match(/\*(.+?)\*/)
      if (m2) return { action: m2[1], content: text.replace(/\*.+?\*/, '').trim() }
      return { action: null, content: text.trim() }
    },

    extractMemory(userMsg) {
      var self = this
      if (self.memories.length >= 50) return
      var msg = userMsg.trim()
      if (!msg || msg.length < 3) return
      var rules = [
        { regex: /我叫(.{1,10})/, tag: '名字' },
        { regex: /我的名字[是叫](.{1,10})/, tag: '名字' },
        { regex: /你可以叫我(.{1,10})/, tag: '名字' },
        { regex: /我今年(.{1,3})岁/, tag: '年龄' },
        { regex: /我的生日[是]?(.+)/, tag: '生日' },
        { regex: /我生于(.+)/, tag: '生日' },
        { regex: /我住[在]?(.{2,15})/, tag: '居住地' },
        { regex: /我在(.{2,10})工作/, tag: '工作' },
        { regex: /我的工作[是]?(.+)/, tag: '工作' },
        { regex: /我是做(.+)的/, tag: '工作' },
        { regex: /我在(.{2,10})上学/, tag: '学校' },
        { regex: /我[上读]的是(.+)/, tag: '学校' },
        { regex: /我喜欢(.{1,15})/, tag: '喜好' },
        { regex: /我最爱(.{1,15})/, tag: '喜好' },
        { regex: /我讨厌(.{1,15})/, tag: '讨厌' },
        { regex: /我不喜欢(.{1,15})/, tag: '讨厌' },
        { regex: /我的爱好[是]?(.+)/, tag: '爱好' },
        { regex: /我喜欢吃(.{1,10})/, tag: '食物' },
        { regex: /我喜欢听(.{1,10})/, tag: '音乐' },
        { regex: /我喜欢看(.{1,10})/, tag: '影视' },
        { regex: /我养了(.{1,10})/, tag: '宠物' },
        { regex: /我家(.{1,5})叫(.{1,10})/, tag: '宠物' },
        { regex: /我现在很(.{1,6})/, tag: '心情' },
        { regex: /我最近很(.{1,6})/, tag: '心情' },
        { regex: /我今天(.{1,10})/, tag: '今天' },
        { regex: /我明天[要]?(.+)/, tag: '计划' },
        { regex: /我下周[要]?(.+)/, tag: '计划' },
        { regex: /我打算(.+)/, tag: '计划' },
        { regex: /我准备(.+)/, tag: '计划' },
        { regex: /我刚(.{1,15})/, tag: '经历' },
        { regex: /我害怕(.{1,10})/, tag: '恐惧' },
        { regex: /我担心(.{1,10})/, tag: '担忧' },
        { regex: /我想要(.{1,15})/, tag: '愿望' },
        { regex: /我需要(.{1,15})/, tag: '需求' },
        { regex: /我有一个(.{1,10})/, tag: '拥有' },
        { regex: /我[是当](.{1,8})的/, tag: '身份' },
        { regex: /我是(.{1,6})/, tag: '身份' }
      ]
      var saved = false
      var memText = ''
      for (var i = 0; i < rules.length; i++) {
        var match = msg.match(rules[i].regex)
        if (match) {
          var value = match[1].trim()
          if (value.length > 0 && value.length < 20) {
            var memory = rules[i].tag + '：' + value
            var duplicate = false
            for (var j = 0; j < self.memories.length; j++) {
              if (self.memories[j] === memory) {
                duplicate = true
                break
              }
            }
            if (duplicate) break
            var updated = false
            for (var k = 0; k < self.memories.length; k++) {
              if (self.memories[k].indexOf(rules[i].tag + '：') !== -1) {
                self.memories[k] = memory
                self.memTimes[k] = Date.now()
                updated = true
                break
              }
            }
            if (!updated) {
              self.memories.push(memory)
              self.memTimes.push(Date.now())
            }
            saved = true
            memText = memory
            break
          }
        }
      }
      if (!saved && (msg.indexOf('记住') !== -1 || msg.indexOf('记着') !== -1 || msg.indexOf('别忘了') !== -1)) {
        var exists = false
        for (var x = 0; x < self.memories.length; x++) {
          if (self.memories[x] === msg) {
            exists = true
            break
          }
        }
        if (!exists) {
          self.memories.push(msg)
          self.memTimes.push(Date.now())
          saved = true
          memText = msg
        }
      }
      if (saved) {
        self.saveMemories()
        self.showToast('已记住：' + (memText.length > 12 ? memText.substring(0, 12) + '...' : memText))
      }
    },

    autoSaveToFile() {
      // #ifdef APP-PLUS
      var self = this
      if (!self.aiId || !self.aiName) return
      var rm = uni.getStorageSync('chat_msgs_' + self.aiId)
      var msgs = []
      if (rm) { try { msgs = typeof rm === 'string' ? JSON.parse(rm) : rm } catch(e) {} }
      if (!msgs || msgs.length === 0) return
      var rs = uni.getStorageSync('chat_settings_' + self.aiId)
      var settings = {}
      if (rs) { try { settings = typeof rs === 'string' ? JSON.parse(rs) : rs } catch(e) {} }
      var rmem = uni.getStorageSync('chat_memories_' + self.aiId)
      var memData = { memories: [], times: [] }
      if (rmem) { try { memData = typeof rmem === 'string' ? JSON.parse(rmem) : rmem } catch(e) {} }
      var rb = uni.getStorageSync('chat_bg_' + self.aiId)
      var bgData = null
      if (rb) { try { bgData = typeof rb === 'string' ? JSON.parse(rb) : rb } catch(e) {} }
      var saveData = JSON.stringify({
        version: '1.0',
        app: '落雨AI',
        saveTime: Date.now(),
        ai: {
          id: self.aiId,
          name: self.aiName,
          gender: self.aiGender,
          prompt: self.aiPrompt,
          personality: self.aiPersonality,
          background: self.aiBackground,
          habits: self.aiHabits
        },
        messages: msgs,
        memories: memData,
        settings: settings,
        chatBg: bgData
      })
      plus.android.requestPermissions(
        ['android.permission.WRITE_EXTERNAL_STORAGE', 'android.permission.READ_EXTERNAL_STORAGE'],
        function() { self.doSaveFile(saveData) },
        function() {}
      )
      // #endif
    },

    doSaveFile(saveData) {
      var self = this
      var fileName = self.aiName + '_' + self.aiId + '.json'
      var dirName = '落雨备份'
      function write(dirEntry) {
        dirEntry.getFile(fileName, { create: true }, function(fileEntry) {
          fileEntry.createWriter(function(writer) {
            writer.onwrite = function() {}
            writer.write(saveData)
          })
        })
      }
      plus.io.resolveLocalFileSystemURL('file:///storage/emulated/0/' + dirName, function(dirEntry) {
        write(dirEntry)
      }, function() {
        plus.io.resolveLocalFileSystemURL('file:///storage/emulated/0/', function(rootEntry) {
          rootEntry.getDirectory(dirName, { create: true, exclusive: false }, function(dirEntry) {
            write(dirEntry)
          })
        })
      })
    },

    startAuto() {
      var self = this
      self.autoTimer = setInterval(function() {
        self.loadSettings()
        if (!self.settings.autoMessage || self.settings.blocked) return
        var rm = uni.getStorageSync('chat_msgs_' + self.aiId)
        var msgs = []
        if (rm) { try { msgs = typeof rm === 'string' ? JSON.parse(rm) : rm } catch(e) {} }
        if (!msgs || msgs.length === 0) return
        var last = msgs[msgs.length - 1]
        if (last.role === 'ai') return
        if ((Date.now() - last.time) / 3600000 < 3) return
        var recentStart = Math.max(0, msgs.length - 10)
        var recent = []
        for (var i = recentStart; i < msgs.length; i++) {
          var role = msgs[i].role === 'user' ? '用户' : self.aiName
          var c = msgs[i].type === 'image' ? '[图片]' : msgs[i].type === 'emoji' ? '[表情]' : msgs[i].content
          recent.push(role + '：' + c)
        }
        var prompt = '你是' + self.aiName + '，' + self.aiPersonality + '。\n用户很久没回复了，最近聊天：\n\n' + recent.join('\n') + '\n\n请结合内容发一条关心的消息，不超过40字。'
        if (self.settings.showAction) {
          if (self.settings.actionPosition === 'outside') prompt += '用*星号*包裹神态动作。'
          else prompt += '用（）包裹神态动作。'
        } else {
          prompt += '直接回复对话内容，不要用星号或括号描写动作。'
        }
        uni.request({
          url: self.apiUrl + '/v1/chat/completions',
          method: 'POST',
          header: { 'Authorization': 'Bearer ' + self.apiKey, 'Content-Type': 'application/json' },
          data: { model: self.modelName, messages: [{ role: 'user', content: prompt }], max_tokens: 150 },
          success: function(res) {
            if (res.statusCode === 200 && res.data && res.data.choices) {
              var reply = res.data.choices[0].message.content
              var parsed = self.parseReply(reply)
              var raw = uni.getStorageSync('chat_msgs_' + self.aiId)
              var latest = []
              if (raw) { try { latest = typeof raw === 'string' ? JSON.parse(raw) : raw } catch(e) {} }
              latest.push({
                id: 'm_' + Date.now(),
                role: 'ai',
                action: parsed.action,
                content: parsed.content,
                type: 'text',
                time: Date.now(),
                auto: true
              })
              uni.setStorageSync('chat_msgs_' + self.aiId, JSON.stringify(latest))
              self.msgs = latest
              if (self.settings.autoSave) self.autoSaveToFile()
              uni.showToast({ title: self.aiName + '发来一条消息', icon: 'none' })
            }
          }
        })
      }, 30 * 60 * 1000)
    },

    showTime(idx) {
      if (idx === 0) return true
      var prev = this.msgs[idx - 1]
      var curr = this.msgs[idx]
      if (!prev || !curr) return false
      return (curr.time - prev.time) > 3 * 60 * 1000
    },

    fmtTime(ts) {
      var d = new Date(ts)
      var now = new Date()
      var h = String(d.getHours()).padStart(2, '0')
      var m = String(d.getMinutes()).padStart(2, '0')
      var timeStr = h + ':' + m
      if (d.toDateString() === now.toDateString()) return '今天 ' + timeStr
      var yesterday = new Date(now)
      yesterday.setDate(yesterday.getDate() - 1)
      if (d.toDateString() === yesterday.toDateString()) return '昨天 ' + timeStr
      return (d.getMonth() + 1) + '/' + d.getDate() + ' ' + timeStr
    }
  }
}
</script>

<style>
page { background: #F5F5F5; }
.page { width: 100%; height: 100vh; display: flex; flex-direction: column; overflow: hidden; position: relative; }
.chat-list { width: 100%; height: 0; flex: 1; padding: 0 12px; box-sizing: border-box; }
.time-tag { text-align: center; padding: 12px 0; }
.time-tag text { font-size: 11px; color: #B0B0B0; background: rgba(0,0,0,0.04); padding: 3px 10px; border-radius: 8px; }
.row { display: flex; margin-bottom: 16px; align-items: flex-start; }
.row.ai { justify-content: flex-start; }
.row.user { justify-content: flex-end; }
.msg-avatar-box { flex-shrink: 0; }
.msg-avatar { width: 38px; height: 38px; border-radius: 50%; overflow: hidden; display: flex; align-items: center; justify-content: center; }
.msg-avatar.bg1 { background: linear-gradient(135deg, #FFE5D0, #FFD4B8); }
.msg-avatar.bg2 { background: linear-gradient(135deg, #D0E8FF, #B8D8FF); }
.msg-avatar.bg3 { background: linear-gradient(135deg, #D5F5E3, #B8E8D0); }
.msg-avatar.bg4 { background: linear-gradient(135deg, #F5D0FF, #E8B8FF); }
.msg-avatar.bg5 { background: linear-gradient(135deg, #FFF3D0, #FFE8B8); }
.msg-avatar.bg6 { background: linear-gradient(135deg, #FFD0D0, #FFB8B8); }
.msg-avatar.bg7 { background: linear-gradient(135deg, #D0F0FF, #B8E8FF); }
.msg-avatar.user-avatar-bg { background: linear-gradient(135deg, #E8E8E8, #D0D0D0); }
.msg-avatar-img { width: 38px; height: 38px; }
.bubble-box { max-width: 65%; margin-left: 8px; }
.user-bubble-box { margin-left: 0; margin-right: 8px; }
.action { margin-bottom: 3px; }
.action text { font-size: 12px; color: #B0A8A0; font-style: italic; }
.bubble { padding: 10px 14px; border-radius: 14px; }
.bubble text { font-size: 14px; line-height: 1.6; }
.ai-bubble { background: #FFF; border-radius: 4px 14px 14px 14px; box-shadow: 0 1px 3px rgba(0,0,0,0.04); }
.ai-bubble text { color: #2D2A26; }
.user-bubble { background: #2D2A26; border-radius: 14px 4px 14px 14px; }
.user-bubble text { color: #FFF; }
.typing text { color: #B0B0B0; }

/* 图片消息：无黑框，无深色背景 */
.image-wrap {
  max-width: 200px;
  border-radius: 12px;
  overflow: hidden;
}
.msg-image {
  width: 200px;
  border-radius: 12px;
  display: block;
}
.image-caption-box {
  background: #2D2A26;
  padding: 6px 10px;
}
.image-caption {
  font-size: 13px;
  color: #FFF;
  display: block;
}

/* 表情包消息 */
.emoji-wrap {
  width: 120px;
  height: 120px;
}
.emoji-send-img {
  width: 120px;
  height: 120px;
  display: block;
}

.memory-toast { position: absolute; top: 12px; left: 50%; transform: translateX(-50%); background: rgba(0,0,0,0.7); border-radius: 20px; padding: 8px 16px; display: flex; align-items: center; gap: 6px; z-index: 50; animation: fadeInToast 0.3s ease; }
.memory-toast.fadeOut { animation: fadeOutToast 0.5s ease forwards; }
.toast-icon { font-size: 14px; }
.toast-text { font-size: 12px; color: #FFF; white-space: nowrap; }
@keyframes fadeInToast { from { opacity: 0; transform: translateX(-50%) translateY(-8px); } to { opacity: 1; transform: translateX(-50%) translateY(0); } }
@keyframes fadeOutToast { from { opacity: 1; } to { opacity: 0; } }
.emoji-panel { width: 100%; height: 250px; background: #FFF; border-top: 0.5px solid #E8E8E8; display: flex; flex-direction: column; flex-shrink: 0; }
.emoji-tabs { display: flex; height: 38px; border-bottom: 0.5px solid #F0F0F0; flex-shrink: 0; }
.emoji-tab { flex: 1; display: flex; align-items: center; justify-content: center; position: relative; }
.emoji-tab-text { font-size: 13px; color: #999; }
.emoji-tab.active .emoji-tab-text { color: #E8936A; font-weight: 600; }
.emoji-tab.active::after { content: ''; position: absolute; bottom: 0; left: 25%; right: 25%; height: 2px; background: #E8936A; border-radius: 1px; }
.emoji-grid { flex: 1; overflow: hidden; }
.emoji-grid-inner { display: flex; flex-wrap: wrap; padding: 8px 10px; box-sizing: border-box; align-content: flex-start; }
.emoji-item { width: 12.5%; height: 44px; display: flex; align-items: center; justify-content: center; }
.emoji-item text { font-size: 22px; }
.emoji-item:active { background: #F0F0F0; border-radius: 8px; }
.custom-emoji-item { position: relative; }
.custom-emoji-img { width: 38px; height: 38px; border-radius: 6px; }
.add-emoji-btn { border: 2px dashed #E0E0E0; border-radius: 8px; margin: 3px; box-sizing: border-box; background: #FAFAFA; }
.add-emoji-btn:active { background: #F0F0F0; }
.add-emoji-icon { font-size: 24px; color: #CCC; }
.empty-emoji-hint { text-align: center; padding: 20px; }
.empty-emoji-text { font-size: 13px; color: #CCC; }
.input-bar { width: 100%; display: flex; align-items: center; padding: 8px 12px; background: rgba(237,237,237,0.95); border-top: 0.5px solid #D8D8D8; box-sizing: border-box; flex-shrink: 0; }
.emoji-btn { width: 36px; height: 36px; display: flex; align-items: center; justify-content: center; flex-shrink: 0; }
.emoji-btn:active { opacity: 0.6; }
.emoji-icon { width: 24px; height: 24px; }
.img-btn { width: 36px; height: 36px; display: flex; align-items: center; justify-content: center; flex-shrink: 0; margin-left: 4px; }
.img-btn:active { opacity: 0.6; }
.img-icon { width: 24px; height: 24px; }
.input-field { flex: 1; height: 38px; background: #FFF; border: 1px solid #E0E0E0; border-radius: 19px; padding: 0 14px; font-size: 14px; box-sizing: border-box; margin-left: 8px; }
.send-space { width: 8px; flex-shrink: 0; }
.send-btn { padding: 9px 16px; border-radius: 19px; background: #D0D0D0; flex-shrink: 0; }
.send-btn.active { background: #E8936A; }
.send-btn text { font-size: 14px; color: #FFF; }
</style>
