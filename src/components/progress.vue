<template>
  <div class="_wrapper">
    <div
      v-show="hasPercent"
      :style="
    'font-size:'+height+'rpx;'
    "
    >
      {{0}}
    </div>
    <div
      class="progress progress__bg"
      id="progress"
      :style="
    'width:'+width+'rpx;height:'+height+'rpx;background-color:'+bgcColor+''
    "
    >
      <div
        class="progress__line"
        :style="
    'width:'+activeWidth+'rpx;height:'+height+'rpx;background-color:'+activeColor+''
    "
      ></div>
    </div>
    <div
      v-show="hasPercent"
      :style="
    'font-size:'+height+'rpx;'
    "
    >
      {{percent}}
    </div>
  </div>
</template>

<script>
export default {
  name: 'progress',
  data () {
    return {
      activeWidth: ''
    }
  },
  props: {
    //  进度条的宽度
    hasPercent: {
      type: Boolean,
      value: 10
    },
    height: {
      type: Number,
      value: 10
    },
    //  进度条长度
    width: {
      type: Number,
      value: 750
    },
    // normal , sp,默认normal
    direction: {
      type: String,
      value: 'normal'
    },
    // 背景色
    bgcColor: {
      type: String,
      value: 'green'
    },
    // 完成的进度条的颜色
    activeColor: {
      type: String,
      value: 'red'
    },
    // 进度完成百分比
    percent: {
      type: [Number, String],
      value: 50
    }
  },
  created () {
    console.log('==========')
    console.log(this.width)
    console.log(this.height)
    console.log(this.bgcColor)
    console.log(this.direction)
    console.log(this.percent)
    this.activeWidth = this.width * this.percent / 100
    console.log('', this.activeWidth)
    console.log('==========')
  },
  methods: {
    initBase () {
    }
  }
}
</script>

<style scoped lang="scss">
._wrapper {
  display: flex;
}
.progress {
  background: "red";
}
.progress__bg {
  position: relative;
}
.progress__line {
  position: absolute;
}
</style>

