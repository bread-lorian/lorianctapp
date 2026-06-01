<script>
export default {
  onLaunch() {
    var self = this

    // 获取设备信息
    var sysInfo = uni.getSystemInfoSync()

    // 生成设备ID
    var deviceId = uni.getStorageSync('device_id')
    if (!deviceId) {
      deviceId = 'dev_' + Date.now() + '_' + Math.random().toString(36).substr(2, 8)
      uni.setStorageSync('device_id', deviceId)
    }

    // 设备上报
    uniCloud.callFunction({
      name: 'admin-api',
      data: {
        action: 'reportDevice',
        data: {
          deviceId: deviceId,
          platform: sysInfo.platform || 'unknown',
          version: '1.0.0'
        }
      }
    }).then(function(res) {
      console.log('设备上报成功', res.result)
    }).catch(function(err) {
      console.log('设备上报失败', err)
    })

    // 检查新公告
    var lastRead = uni.getStorageSync('last_notice_read') || 0
    uniCloud.callFunction({
      name: 'admin-api',
      data: {
        action: 'getUnreadAnnouncements',
        data: { lastReadTime: lastRead }
      }
    }).then(function(res) {
      if (res.result.code === 0 && res.result.data && res.result.data.length > 0) {
        var newest = res.result.data[0]
        uni.showModal({
          title: newest.title,
          content: newest.content,
          showCancel: false,
          confirmText: '知道了',
          success: function() {
            uni.setStorageSync('last_notice_read', Date.now())
          }
        })
      }
    }).catch(function(err) {
      console.log('检查公告失败', err)
    })
  },

  onShow() {},
  onHide() {}
}
</script>

<style>
page { background: #F2F2F2; }
</style>
