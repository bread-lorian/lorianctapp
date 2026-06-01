<template>
  <view class="page">
    <scroll-view class="scroll" scroll-y @scrolltolower="loadMore" lower-threshold="100">

      <view class="post-card" v-for="post in posts" :key="post._id">
        <view class="post-header">
          <view class="post-avatar">
            <text class="post-avatar-text">{{ post.author_name ? post.author_name[0] : '?' }}</text>
          </view>
          <view class="post-info">
            <text class="post-author">{{ post.author_name || '匿名' }}</text>
            <text class="post-time">{{ formatTime(post.created_at) }}</text>
          </view>
        </view>
        <text class="post-content">{{ post.content }}</text>
        <view class="post-images" v-if="post.images && post.images.length > 0">
          <image v-for="(img, idx) in post.images" :key="idx" :src="img" mode="aspectFill" class="post-img" @click="previewImage(post.images, idx)" />
        </view>
        <view class="post-footer">
          <view class="post-action" @click="likePost(post)">
            <text class="post-action-icon">&#x2764;</text>
            <text class="post-action-text">{{ post.likes || 0 }}</text>
          </view>
        </view>
      </view>

      <view class="load-more" v-if="loading">
        <text class="load-text">加载中...</text>
      </view>
      <view class="load-more" v-if="noMore && posts.length > 0">
        <text class="load-text">没有更多了</text>
      </view>
      <view class="empty" v-if="!loading && posts.length === 0">
        <text class="empty-text">还没有动态</text>
      </view>

      <view style="height: 20px;"></view>
    </scroll-view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      posts: [],
      page: 1,
      loading: false,
      noMore: false
    }
  },

  onLoad() {
    this.page = 1
    this.posts = []
    this.noMore = false
    this.loadPosts()
  },

  onShow() {
    this.page = 1
    this.posts = []
    this.noMore = false
    this.loadPosts()
  },

  methods: {

    loadPosts() {
      var self = this
      if (self.loading || self.noMore) return
      self.loading = true

      uniCloud.callFunction({
        name: 'admin-api',
        data: {
          action: 'getWorldPosts',
          data: { page: self.page }
        }
      }).then(function(res) {
        if (res.result.code === 0) {
          var list = res.result.data || []
          if (list.length < 20) self.noMore = true
          if (self.page === 1) {
            self.posts = list
          } else {
            self.posts = self.posts.concat(list)
          }
          self.page++
        }
      }).catch(function(err) {
        console.error('加载失败', err)
      }).finally(function() {
        self.loading = false
      })
    },

    loadMore() {
      this.loadPosts()
    },

    likePost(post) {
      var self = this
      post.likes = (post.likes || 0) + 1

      uniCloud.callFunction({
        name: 'admin-api',
        data: {
          action: 'likePost',
          data: { postId: post._id }
        }
      })
    },

    previewImage(urls, idx) {
      uni.previewImage({ urls: urls, current: idx })
    },

    formatTime(ts) {
      if (!ts) return ''
      var d = new Date(ts)
      var now = new Date()
      var h = String(d.getHours()).padStart(2, '0')
      var m = String(d.getMinutes()).padStart(2, '0')
      if (d.toDateString() === now.toDateString()) return '今天 ' + h + ':' + m
      var yesterday = new Date(now)
      yesterday.setDate(yesterday.getDate() - 1)
      if (d.toDateString() === yesterday.toDateString()) return '昨天 ' + h + ':' + m
      return (d.getMonth() + 1) + '/' + d.getDate() + ' ' + h + ':' + m
    }
  }
}
</script>

<style>
page { background: #F2F2F2; }
.page { width: 100%; min-height: 100vh; background: #F2F2F2; }
.scroll { width: 100%; min-height: 100vh; }
.post-card { background: #FFF; margin: 8px 12px; border-radius: 12px; padding: 16px; }
.post-header { display: flex; align-items: center; margin-bottom: 10px; }
.post-avatar { width: 40px; height: 40px; border-radius: 50%; background: linear-gradient(135deg, #FFE5D0, #FFD4B8); display: flex; align-items: center; justify-content: center; flex-shrink: 0; }
.post-avatar-text { font-size: 18px; font-weight: 600; color: #E8936A; }
.post-info { margin-left: 10px; flex: 1; }
.post-author { font-size: 15px; font-weight: 600; color: #2D2A26; display: block; }
.post-time { font-size: 12px; color: #CCC; margin-top: 2px; display: block; }
.post-content { font-size: 15px; color: #333; line-height: 1.7; display: block; }
.post-images { display: flex; flex-wrap: wrap; gap: 6px; margin-top: 10px; }
.post-img { width: 100px; height: 100px; border-radius: 8px; }
.post-footer { display: flex; align-items: center; gap: 20px; margin-top: 12px; padding-top: 10px; border-top: 0.5px solid #F0F0F0; }
.post-action { display: flex; align-items: center; gap: 4px; }
.post-action:active { opacity: 0.6; }
.post-action-icon { font-size: 16px; color: #CCC; }
.post-action-text { font-size: 13px; color: #999; }
.empty { text-align: center; padding: 80px 30px; }
.empty-text { font-size: 14px; color: #CCC; }
.load-more { text-align: center; padding: 16px; }
.load-text { font-size: 13px; color: #CCC; }
</style>
