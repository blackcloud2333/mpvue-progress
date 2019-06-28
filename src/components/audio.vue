<template>
  <div class="">
    <progress
      :width=750
      :height=30
      :hasPercent=true
      :percent=50
      :direction="'ss'"
      :bgcColor="'green'"
      :activeColor="'red'"
    >
    </progress>
  </div>
</template>

<script>
import progress from '@/components/progress.vue';
export default {
  name: 'audio',
  components: {
    progress
  },
  props: {
    contentTitle: {
      type: String,
      default: ''
    },
    audioUrl: {
      type: String,
      default: ''
    },
    delayTime: {
      type: Number,
      default: 15
    },
    // 达到百分之x触发事件
    ruleProcess: {
      type: Number,
      default: 0.8
    },
    actionCode: {
      type: String,
      default: ''
    }
  },
  onLoad () {
    this.innerAudioContext = wx.getBackgroundAudioManager()
    this.innerAudioContext.title = this.contentTitle
    this.innerAudioContext.src = this.audioUrl
    this.innerAudioContext.autoplay = true
    let query = wx.createSelectorQuery()
    let self = this
    if (this.areaWidth === undefined || this.areaWidth === null || this.viewWidth === undefined || this.viewWidth === null) {
      // 获取movable的宽度，计算改变进度使用
      setTimeout(() => {
        query.select('#movable-area').boundingClientRect(function (rect) {
          self.areaWidth = rect.width
        }).exec()
        query.select('#movable-view').boundingClientRect(function (rect) {
          self.viewWidth = rect.width // 节点的宽度
        }).exec()
      }, 300)
    },
    // 播放
      this.innerAudioContext.onPlay(() => {
        console.log('onPlay load')
        this.voice.tip = 'Playing'
      })
      this.innerAudioContext.onStop(() => {
        console.log('onStop')
        this.voice.tip = 'Stop'
      })
      this.innerAudioContext.onPause(() => {
        console.log('Pause')
        this.voice.tip = 'Pause'
      })
      // 播放进度
      this.innerAudioContext.onTimeUpdate(() => {
        this.voice.progress = Math.round(100 * this.innerAudioContext.currentTime / this.innerAudioContext.duration)
        this.voice.time = this.dateformat(Math.round(this.innerAudioContext.currentTime))
        this.currentProcess = `${this.voice.time.min}:${this.voice.time.sec}`
        let total = this.dateformat(Math.round(this.innerAudioContext.duration))
        this.totalProcess = `${total.min}:${total.sec}`
        if (!this.isChanging) {
          this.voice.margin = Math.round((this.areaWidth - this.viewWidth) * (this.innerAudioContext.currentTime / this.innerAudioContext.duration)) // 计算当前滑块margin-left
        }
        // setInterval(() => {
        //   this.spTime = this.dateformat(Math.round(this.innerAudioContext.currentTime))
        // }, 1000)
        console.log('=====不断更新的时间', this.voice.time.sec)
        this.judgePlayProcess()
      })
      // 播放结束
      this.innerAudioContext.onEnded(() => {
        console.log('onEnded')
        this.voice.progress = 100
        this.voice.tip = 'End Playing'
        this.voice.time = this.dateformat(Math.round(this.innerAudioContext.duration))
        this.spTime = this.dateformat(Math.round(this.innerAudioContext.duration))
        this.voice.margin = Math.round(this.areaWidth - this.viewWidth)
      })
      // 播放错误
      this.innerAudioContext.onError((res) => {
        this.voice.tip = 'Error Playing'
      })
  },
      methods: {
      // 当开始滑动，分离currentTime
      seekTouchStart (e) {
        this.isChanging = true
        console.log('seekTouchStart isChanging', this.isChanging)
        console.log(e)
      },
      seekTouchEnd (e) {
        this.isChanging = false
        var that = this
        that.innerAudioContext.seek(that.innerAudioContext.duration * (that.voice.progress / 100))
        that.innerAudioContext.play()
        that.voice.time = that.dateformat(Math.round(that.innerAudioContext.duration * (that.voice.progress / 100)))
        that.spTime = that.dateformat(Math.round(that.innerAudioContext.duration * (that.voice.progress / 100)))
        console.log('滑动之后的进度', this.voice.time)
      },
      // 移动音频滑块，此处不能设置moveable-view 的x值，会有冲突延迟
      voiceSeekMove (e) {
        var progress = Math.round(e.mp.detail.x / (this.areaWidth - this.viewWidth) * 100)
        this.voice.progress = progress
        this.voice.margin = e.mp.detail.x
        this.voice.time = this.dateformat(Math.round(this.innerAudioContext.duration * (this.voice.progress / 100)))
        this.spTime = this.dateformat(Math.round(this.innerAudioContext.duration * (this.voice.progress / 100)))
      },
      // 点击播放、暂停
      voiceClick () {
        const paused = this.innerAudioContext.paused
        setTimeout(() => {
          paused
            ? this.innerAudioContext.play()
            : this.innerAudioContext.pause()
        }, 300)
      },
      dateformat (second) {
        // 天
        var day = Math.floor(second / (3600 * 24))
        // 小时位
        var hour = Math.floor((second - day * 3600 * 24) / 3600)
        // 分钟位
        var min = Math.floor((second - day * 3600 * 24 - hour * 3600) / 60)
        // 秒位
        var sec = Math.round(second - day * 3600 * 24 - hour * 3600 - min * 60) // equal to => var sec = second % 60;

        return {
          'day': day,
          'hour': this.p(hour),
          'min': this.p(min),
          'sec': this.p(sec)
        }
      },
      p (s) {
        return s < 10 ? '0' + s : s
      },
      finishCourse () {
        this.$api.actionFinish({
          activityCode: this.activityCode,
          actionCode: this.actionCode,
          actionType: this.actionType,
          actionName: this.actionName,
          cycle: this.cycle
        }).then(res => {
          this.gold = res.data.totalAmt
          this.reachSignal = false
          this.$dialog.toast(`恭喜您获得${this.gold}个金币`)
        })
      },
      jumpToMore () {
        console.log(this.innerAudioContext.currentTime)
        if (this.innerAudioContext.currentTime + this.delayTime <= this.innerAudioContext.duration) {
          this.aimTime = this.innerAudioContext.currentTime + this.delayTime
        } else {
          this.aimTime = this.innerAudioContext.duration
        }
        this.innerAudioContext.seek(this.aimTime)
        this.voice.time = this.dateformat(Math.round(this.innerAudioContext.currentTime))
      },
      jumpToLess () {
        if (this.innerAudioContext.currentTime - this.delayTime > 0) {
          this.aimTime = this.innerAudioContext.currentTime - this.delayTime
        } else {
          this.aimTime = 0
        }
        this.innerAudioContext.seek(this.aimTime)
        // this.innerAudioContext.play()
      },
      judgePlayProcess () {
        this.spProcess = (this.innerAudioContext.currentTime / this.innerAudioContext.duration)
        if (this.spProcess >= this.ruleProcess && !this.reachSignal && !this.hasReachSignal) {
          this.reachSignal = true
          this.hasReachSignal = true
          this.$sa.track('page_view', {
            page_name: '浏览家庭健康服务行动详情完成弹窗',
            goods_name: this.goodsName,
            item_name: this.actionName,
            page_type: this.contentName
          })
        }
      },
      closeShowGroup () {
        this.reachSignal = false
        this.finishCourse()
      },
      onLoading () {
        this.$dialog.toast('...加载中')
      }
    }
}
</script>

<style>
.card {
  padding: 10px;
}
</style>
