<template>
  <view class="page">
    <scroll-view class="scroll" scroll-y>

      <!-- 添加按钮 -->
      <view class="add-bar" @click="showAddPanel">
        <text class="add-icon">+</text>
        <text class="add-text">添加新记忆</text>
      </view>

      <!-- 记忆列表 -->
      <view class="group" v-if="memories.length > 0">
        <view class="item" v-for="(mem, idx) in memories" :key="idx" @click="editMemory(idx)">
          <view class="item-left">
            <text class="mem-tag">{{ getTag(mem) }}</text>
            <text class="mem-value">{{ getValue(mem) }}</text>
          </view>
          <view class="item-right" @click.stop="deleteMemory(idx)">
            <text class="del-btn">删除</text>
          </view>
        </view>
      </view>

      <!-- 空状态 -->
      <view class="empty-box" v-if="memories.length === 0">
        <text class="empty-icon"></text>
        <text class="empty-text">暂无记忆</text>
        <text class="empty-hint">AI会自动记住你提到的信息，你也可以手动添加。</text>
      </view>

      <view class="count-bar" v-if="memories.length > 0">
        <text class="count-text">{{ memories.length }} / 50 条记忆</text>
      </view>

      <view style="height: 40px;"></view>
    </scroll-view>

    <!-- 添加/编辑弹窗 -->
    <view class="mask" v-if="showPanel" @click="closePanel"></view>
    <view class="panel" :class="{ show: showPanel }">
      <view class="panel-header">
        <text class="panel-title">{{ isEdit ? '编辑记忆' : '添加记忆' }}</text>
        <view class="panel-close" @click="closePanel">
          <text class="panel-close-text">✕</text>
        </view>
      </view>
      <view class="panel-body">
        <view class="panel-field">
          <text class="field-label">分类标签</text>
          <input class="field-input" v-model="editTag" placeholder="如：名字、喜好、工作..." />
        </view>
        <view class="panel-field">
          <text class="field-label">具体内容</text>
          <input class="field-input" v-model="editValue" placeholder="如：喜欢吃火锅、叫小明..." />
        </view>
      </view>
      <view class="panel-footer">
        <view class="panel-btn cancel" @click="closePanel">
          <text class="panel-btn-text">取消</text>
        </view>
        <view class="panel-btn confirm" @click="saveMemory">
          <text class="panel-btn-text">保存</text>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      aiId: '',
      memories: [],
      memTimes: [],
      showPanel: false,
      isEdit: false,
      editIdx: -1,
      editTag: '',
      editValue: ''
    }
  },

  onLoad(opts) {
    this.aiId = opts.id || ''
    this.loadMemories()
  },

  onShow() {
    this.loadMemories()
  },

  methods: {

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

    saveToStorage() {
      uni.setStorageSync('chat_memories_' + this.aiId, JSON.stringify({
        memories: this.memories,
        times: this.memTimes || []
      }))
    },

    getTag(mem) {
      if (!mem) return ''
      var idx = mem.indexOf('：')
      if (idx !== -1) return mem.substring(0, idx)
      return '其他'
    },

    getValue(mem) {
      if (!mem) return ''
      var idx = mem.indexOf('：')
      if (idx !== -1) return mem.substring(idx + 1)
      return mem
    },

    showAddPanel() {
      this.isEdit = false
      this.editIdx = -1
      this.editTag = ''
      this.editValue = ''
      this.showPanel = true
    },

    editMemory(idx) {
      this.isEdit = true
      this.editIdx = idx
      this.editTag = this.getTag(this.memories[idx])
      this.editValue = this.getValue(this.memories[idx])
      this.showPanel = true
    },

    closePanel() {
      this.showPanel = false
    },

    saveMemory() {
      var tag = this.editTag.trim()
      var value = this.editValue.trim()
      if (!value) {
        uni.showToast({ title: '请输入内容', icon: 'none' })
        return
      }

      var memory = tag ? (tag + '：' + value) : value

      if (this.isEdit && this.editIdx >= 0) {
        // 编辑
        this.memories[this.editIdx] = memory
        this.memTimes[this.editIdx] = Date.now()
      } else {
        // 新增
        if (this.memories.length >= 50) {
          uni.showToast({ title: '最多50条记忆', icon: 'none' })
          return
        }
        this.memories.push(memory)
        this.memTimes.push(Date.now())
      }

      this.saveToStorage()
      this.closePanel()
      uni.showToast({ title: this.isEdit ? '已更新' : '已添加', icon: 'success' })
    },

    deleteMemory(idx) {
      var self = this
      uni.showModal({
        title: '删除记忆',
        content: '确定删除这条记忆？',
        confirmText: '删除',
        confirmColor: '#E85D5D',
        success: function(r) {
          if (r.confirm) {
            self.memories.splice(idx, 1)
            if (self.memTimes.length > idx) self.memTimes.splice(idx, 1)
            self.saveToStorage()
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
.page { width: 100%; min-height: 100vh; background: #F2F2F2; position: relative; }
.scroll { width: 100%; min-height: 100vh; }

.add-bar {
  margin: 12px 12px 0;
  background: #E8936A;
  border-radius: 12px;
  height: 48px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
}
.add-bar:active { opacity: 0.85; }
.add-icon { font-size: 20px; color: #FFF; font-weight: 700; }
.add-text { font-size: 15px; color: #FFF; font-weight: 600; }

.group { background: #FFF; margin: 12px; border-radius: 14px; overflow: hidden; }
.item { display: flex; align-items: center; justify-content: space-between; padding: 14px 16px; }
.item:active { background: #F8F8F8; }
.item + .item { border-top: 0.5px solid #F0F0F0; }
.item-left { flex: 1; display: flex; align-items: center; gap: 8px; overflow: hidden; }
.mem-tag {
  font-size: 11px;
  color: #E8936A;
  background: #FFF0E8;
  padding: 2px 8px;
  border-radius: 4px;
  flex-shrink: 0;
}
.mem-value { font-size: 14px; color: #2D2A26; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.item-right { flex-shrink: 0; margin-left: 12px; }
.del-btn { font-size: 13px; color: #E85D5D; }
.del-btn:active { opacity: 0.6; }

.empty-box { display: flex; flex-direction: column; align-items: center; padding: 80px 30px 0; }
.empty-icon { font-size: 40px; }
.empty-text { font-size: 16px; font-weight: 600; color: #CCC; margin-top: 12px; }
.empty-hint { font-size: 13px; color: #DDD; margin-top: 8px; text-align: center; line-height: 1.6; }

.count-bar { text-align: center; padding: 16px 0 0; }
.count-text { font-size: 12px; color: #CCC; }

/* 弹窗遮罩 */
.mask {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(0,0,0,0.4);
  z-index: 100;
  animation: fadeIn 0.2s ease;
}
@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

/* 弹窗面板 */
.panel {
  position: fixed;
  left: 0; right: 0; bottom: 0;
  background: #FFF;
  border-radius: 16px 16px 0 0;
  z-index: 101;
  transform: translateY(100%);
  transition: transform 0.3s ease;
}
.panel.show {
  transform: translateY(0);
}

.panel-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 20px 12px;
  border-bottom: 0.5px solid #F0F0F0;
}
.panel-title { font-size: 17px; font-weight: 700; color: #2D2A26; }
.panel-close { width: 32px; height: 32px; display: flex; align-items: center; justify-content: center; }
.panel-close:active { opacity: 0.5; }
.panel-close-text { font-size: 18px; color: #999; }

.panel-body { padding: 16px 20px; }
.panel-field { margin-bottom: 16px; }
.field-label { font-size: 13px; color: #888; margin-bottom: 8px; display: block; }
.field-input {
  width: 100%;
  height: 44px;
  background: #F5F5F5;
  border: 1px solid #E8E8E8;
  border-radius: 10px;
  padding: 0 14px;
  font-size: 15px;
  color: #2D2A26;
  box-sizing: border-box;
}

.panel-footer {
  display: flex;
  gap: 12px;
  padding: 0 20px 30px;
}
.panel-btn {
  flex: 1;
  height: 44px;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
}
.panel-btn:active { opacity: 0.8; }
.panel-btn.cancel { background: #F0F0F0; }
.panel-btn.confirm { background: #E8936A; }
.panel-btn.cancel .panel-btn-text { color: #666; font-size: 15px; font-weight: 600; }
.panel-btn.confirm .panel-btn-text { color: #FFF; font-size: 15px; font-weight: 600; }
</style>
