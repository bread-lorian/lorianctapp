<template>
  <view class="page">
    <scroll-view class="scroll" scroll-y>
      <view class="av-box" @click="chooseAvatar">
        <view class="av-circle">
          <image v-if="form.avatar" :src="form.avatar" mode="aspectFill" class="av-img" />
          <view v-else class="av-empty">
            <text class="av-plus">+</text>
            <text class="av-hint">选择头像</text>
          </view>
        </view>
      </view>

      <view class="type-show" v-if="form.type">
        <view class="type-badge" :class="form.type === 'partner' ? 'badge-partner' : 'badge-friend'">
          <text class="type-badge-text">{{ form.type === 'partner' ? 'AI伴侣' : 'AI朋友' }}</text>
        </view>
      </view>

      <view class="field">
        <text class="label">名字</text>
        <input class="input" type="text" v-model="form.name" placeholder="给AI取个名字" />
      </view>

      <view class="field">
        <text class="label">性别</text>
        <view class="gr">
          <view class="gb" :class="{ on: form.gender === '女' }" @click="form.gender = '女'"><text>女</text></view>
          <view class="gb" :class="{ on: form.gender === '男' }" @click="form.gender = '男'"><text>男</text></view>
        </view>
      </view>

      <view class="field">
        <text class="label">人设描述（字数不限）</text>
        <textarea class="ta" v-model="form.prompt" placeholder="描述你想要的AI伴侣，例如：温柔体贴的邻家女孩" />
      </view>

      <view class="field">
        <text class="label">性格特点（字数不限）</text>
        <textarea class="ta" v-model="form.personality" placeholder="例如：温柔、体贴、活泼、有点小傲娇" />
      </view>

      <view class="field">
        <text class="label">背景故事（字数不限）</text>
        <textarea class="ta" v-model="form.background" placeholder="AI的背景设定（可选）" />
      </view>

      <view class="field">
        <text class="label">习惯用语（字数不限）</text>
        <textarea class="ta" v-model="form.habits" placeholder="例如：喜欢用~结尾，爱说嘿嘿，说话时带点方言" />
      </view>

      <view class="save-btn" @click="save">
        <text class="save-text">{{ isEdit ? '保存修改' : '添加' }}</text>
      </view>

      <view style="height: 60px;"></view>
    </scroll-view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      isEdit: false,
      editId: '',
      form: {
        avatar: '',
        name: '',
        gender: '女',
        prompt: '',
        personality: '',
        background: '',
        habits: '',
        emoji: '🌸',
        bg: 'bg1',
        type: 'friend',
        relationship: 'friend'
      }
    }
  },
  onLoad(opts) {
    if (opts && opts.type) {
      this.form.type = opts.type
      this.form.relationship = opts.type === 'partner' ? 'lover' : 'friend'
    }

    if (opts && opts.id) {
      this.isEdit = true
      this.editId = opts.id
      var raw = uni.getStorageSync('ai_list')
      var list = []
      if (raw) {
        try { list = typeof raw === 'string' ? JSON.parse(raw) : JSON.parse(JSON.stringify(raw)) } catch(e) { list = [] }
      }
      for (var i = 0; i < list.length; i++) {
        if (list[i].id === opts.id) {
          this.form.avatar = list[i].avatar || ''
          this.form.name = list[i].name || ''
          this.form.gender = list[i].gender || '女'
          this.form.prompt = list[i].prompt || ''
          this.form.personality = list[i].personality || ''
          this.form.background = list[i].background || ''
          this.form.habits = list[i].habits || ''
          this.form.emoji = list[i].emoji || '🌸'
          this.form.bg = list[i].bg || 'bg1'
          this.form.type = list[i].type || 'friend'
          this.form.relationship = list[i].relationship || 'friend'
          break
        }
      }
    }
  },
  methods: {
    chooseAvatar() {
      var self = this
      uni.chooseImage({
        count: 1,
        sizeType: ['compressed'],
        success: function(r) {
          self.form.avatar = r.tempFilePaths[0]
        }
      })
    },
    save() {
      var self = this
      if (!self.form.name || !self.form.name.trim()) {
        uni.showToast({ title: '请输入名字', icon: 'none' })
        return
      }
      if (!self.form.prompt || !self.form.prompt.trim()) {
        uni.showToast({ title: '请输入人设描述', icon: 'none' })
        return
      }

      var bgs = ['bg1', 'bg2', 'bg3', 'bg4', 'bg5', 'bg6', 'bg7']
      var emos = ['🌸', '📚', '⭐', '🎵', '🎮', '🌙', '🧠', '🐱', '🦊', '🐰', '🐻', '🌷']
      if (!self.form.bg) self.form.bg = bgs[Math.floor(Math.random() * bgs.length)]
      if (!self.form.emoji) self.form.emoji = emos[Math.floor(Math.random() * emos.length)]

      // 每次保存都重新读取最新数据
      var raw = uni.getStorageSync('ai_list')
      var list = []
      if (raw) {
        try { list = typeof raw === 'string' ? JSON.parse(raw) : JSON.parse(JSON.stringify(raw)) } catch(e) { list = [] }
      }

      if (self.isEdit) {
        for (var i = 0; i < list.length; i++) {
          if (list[i].id === self.editId) {
            list[i].avatar = self.form.avatar
            list[i].name = self.form.name
            list[i].gender = self.form.gender
            list[i].prompt = self.form.prompt
            list[i].personality = self.form.personality
            list[i].background = self.form.background
            list[i].habits = self.form.habits
            list[i].emoji = self.form.emoji
            list[i].bg = self.form.bg
            break
          }
        }
      } else {
        var newId = 'ai_' + Date.now() + '_' + Math.floor(Math.random() * 10000)
        var newAi = {
          id: newId,
          avatar: self.form.avatar,
          name: self.form.name,
          gender: self.form.gender,
          prompt: self.form.prompt,
          personality: self.form.personality,
          background: self.form.background,
          habits: self.form.habits,
          emoji: self.form.emoji,
          bg: self.form.bg,
          type: self.form.type,
          relationship: self.form.relationship,
          createTime: Date.now(),
          pinned: false
        }
        list.push(newAi)
        uni.setStorageSync('chat_settings_' + newId, JSON.stringify({
          showAction: true,
          blocked: false,
          autoSave: true
        }))
      }

      // 序列化后写入，确保数据完整
      var jsonStr = JSON.stringify(list)
      uni.setStorageSync('ai_list', jsonStr)

      // 验证写入是否成功
      var verify = uni.getStorageSync('ai_list')
      if (!verify) {
        // 写入失败，用同步方式重试
        try {
          uni.setStorageSync('ai_list', jsonStr)
        } catch(e) {
          uni.showToast({ title: '保存失败，请重试', icon: 'none' })
          return
        }
      }

      uni.showToast({ title: self.isEdit ? '已保存' : '添加成功', icon: 'success' })
      setTimeout(function() {
        var pages = getCurrentPages()
        if (pages.length > 1) {
          uni.navigateBack({ delta: 1 })
        } else {
          uni.switchTab({ url: '/pages/index/index' })
        }
      }, 800)
    }
  }
}
</script>

<style>
page {
  background: #F5F5F5;
}

.page {
  width: 100%;
  min-height: 100vh;
  background: #F5F5F5;
}

.scroll {
  width: 100%;
  min-height: 100vh;
}

.av-box {
  width: 100%;
  display: flex;
  justify-content: center;
  padding-top: 30px;
  padding-bottom: 16px;
}

.av-circle {
  width: 80px;
  height: 80px;
  border-radius: 80px;
  overflow: hidden;
  background: linear-gradient(135deg, #FFE5D0, #FFD4B8);
}

.av-img {
  width: 80px;
  height: 80px;
  display: block;
}

.av-empty {
  width: 80px;
  height: 80px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.av-plus {
  font-size: 28px;
  color: #E8936A;
}

.av-hint {
  font-size: 10px;
  color: #B0A8A0;
  margin-top: 4px;
}

.type-show {
  width: 100%;
  display: flex;
  justify-content: center;
  padding-bottom: 20px;
}

.type-badge {
  border-radius: 20px;
  padding: 5px 16px;
}

.type-badge.badge-partner {
  background: #FFE8E8;
}

.type-badge.badge-friend {
  background: #E8F4FF;
}

.type-badge-text {
  font-size: 13px;
  font-weight: 600;
}

.badge-partner .type-badge-text {
  color: #E85D5D;
}

.badge-friend .type-badge-text {
  color: #5D9CE8;
}

.field {
  width: 100%;
  padding: 0 16px;
  margin-bottom: 20px;
  box-sizing: border-box;
}

.label {
  font-size: 14px;
  font-weight: 600;
  color: #2D2A26;
  margin-bottom: 8px;
  display: block;
}

.input {
  width: 100%;
  height: 44px;
  background: #FFF;
  border: 1px solid #E0E0E0;
  border-radius: 12px;
  padding: 0 16px;
  font-size: 14px;
  color: #2D2A26;
  box-sizing: border-box;
}

.ta {
  width: 100%;
  min-height: 80px;
  max-height: 200px;
  background: #FFF;
  border: 1px solid #E0E0E0;
  border-radius: 12px;
  padding: 12px 16px;
  font-size: 14px;
  color: #2D2A26;
  box-sizing: border-box;
  overflow-y: auto;
}

.gr {
  display: flex;
  width: 100%;
}

.gb {
  flex: 1;
  height: 44px;
  border: 1.5px solid #E0E0E0;
  border-radius: 12px;
  background: #FFF;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 12px;
}

.gb:last-child {
  margin-right: 0;
}

.gb text {
  font-size: 14px;
  color: #999;
}

.gb.on {
  border-color: #E8936A;
  background: #FFF8F2;
}

.gb.on text {
  color: #E8936A;
  font-weight: 600;
}

.save-btn {
  width: calc(100% - 32px);
  margin: 20px 16px 0;
  height: 48px;
  background: #E8936A;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.save-text {
  color: #FFF;
  font-size: 16px;
  font-weight: 600;
}

.save-btn:active {
  opacity: 0.85;
}
</style>
