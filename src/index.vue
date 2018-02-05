
<template>
    <div class="wrap-pull" ref="wrap" @scroll="scrollEvent">
            <div :class="pullDownCls" ref="pullDownEl">
                <span class="icon"></span>
                <span :class="pullDownLabelCls">{{pullDownState}}</span>
            </div>
            <slot></slot>
            <div :class="pullUpCls" ref="pullUpEl" v-if="hasMore">
                <span class="icon"></span>
                <span :class="pullUpLabelCls">{{pullUpState}}</span>
            </div>
    </div>
</template>

<script>
const scrollCls = 'scroll',
  pullDownCls = 'pullDown',
  pullDownLabelCls = 'pullDownLabel',
  pullUpCls = 'pullUp',
  pullUpLabelCls = 'pullUpLabel';
const lableUp = {
  initial: '上拉加载更多',
  suspend: '松开加载更多',
  loading: '加载中',
  complete: '加载完成',
};
const lableDown = {
  initial: '下拉刷新',
  suspend: '松开刷新',
  loading: '刷新中',
  complete: '刷新完成',
};
export default {
  name: 'PullToRefresh',
  props: [
    'down',
    'up',
    'pullupOffset',
    'pulldownOffset',
    'addNew',
    'addMore',
    'hasMore',
  ],
  data() {
    return {
      msg: 'PullToRefresh',
      pullDownState: lableDown.initial,
      pullUpState: lableUp.initial,
      pullDownCls,
      pullDownLabelCls,
      pullUpCls,
      pullUpLabelCls,
    };
  },
  components: {},
  methods: {
    scrollEvent(e) {
      this.$emit('scroll', e);
    },
    touchStart(ev) {
      let self = this;
      let touch = ev.touches[0];
      let scrollObj = self.scrollObj;
      self.pullFlag = 0;
      if (self.down) {
        this.pullDownEl.style.webkitTransitionDuration = '0s';
        this.pullDownCls = pullDownCls;
        this.pullDownState = lableDown.initial;
      }
      if (self.up && this.pullUpEl) {
        this.pullUpEl.style.webkitTransitionDuration = '0s';
      }
      self.startY = touch.screenY;
      self.startPageY = scrollObj.scrollTop;
      self.maxY = scrollObj.scrollHeight - scrollObj.clientHeight;
    },
    touchMove(ev) {
      let self = this,
        len = this.pulldownOffset || 80,
        offsetDefault = this.pullupOffset || 20;
      let offsetY, touch;
      touch = ev.touches[0];
      offsetY = (touch.screenY - self.startY) / 2;
      console.log('pulldown', self.down, offsetY, len);
      if (self.down && self.startPageY == 0 && offsetY > 0) {
        ev.preventDefault();
        if (offsetY > len) {
          offsetY = len;
          self.pullDownState = lableDown.suspend;
          self.pullDownCls = 'pullDown flip';
          self.pullFlag = 1;
        }
        self.pullDownEl.style.height = offsetY + 'px';
      } else if (
        self.up &&
        self.startPageY >= self.maxY - offsetDefault &&
        offsetY < 0
      ) {
        ev.preventDefault();
        self.pullFlag = 2;
        self.pullUpCls = 'pullUp flip';
        self.pullUpState = lableUp.suspend;
      }
    },
    touchEnd(ev) {
      let self = this;

      if (self.down && self.pullFlag == 1) {
        self.pullDownCls = 'pullDown loading';
        self.pullDownState = lableDown.loading;
        // console.log('loadData');
        self.addNew().then(function() {
          self.pullDownEl.style.webkitTransitionDuration = '0.5s';
          self.pullDownEl.style.height = 0;
          self.pullDownCls = 'pullDown ok';
          self.pullDownState = lableDown.complete;
        });
      } else if (self.up && self.pullFlag == 2) {
        self.pullUpCls = 'pullUp loading';
        self.pullUpState = lableUp.loading;
        if (self.hasMore) {
          self.addMore().then(function() {
            self.pullUpCls = 'pullUp';
            self.pullUpState = lableUp.initial;
          });
        }
      } else if (self.down) {
        self.pullDownEl.style.webkitTransitionDuration = '0.5s';
        self.pullDownEl.style.height = 0;
      }
      self.pullFlag = 0;
    },
  },

  mounted() {
    this.scrollObj = this.$refs.wrap;
    this.pullDownEl = this.$refs.pullDownEl;

    this.pullUpEl = this.$refs.pullUpEl;

    this.scrollObj.addEventListener('touchstart', this.touchStart.bind(this));
    this.scrollObj.addEventListener('touchmove', this.touchMove.bind(this));
    this.scrollObj.addEventListener('touchend', this.touchEnd.bind(this));
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@import './PullToRefresh.css';
</style>
