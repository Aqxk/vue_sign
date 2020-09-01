<template>
  <div class="sign-wrap">
    <div :class="'sign' + ' ' + signClass">
      <canvas />
    </div>
    <van-button type="info" size="small" style="width: 100px;" @click="clear">清除</van-button>
  </div>
</template>

<script>
import { Button } from 'vant';

export default {
  name: 'Sign',
  components: {
    [Button.name]: Button
  },
  props: [
    'signClass'
  ],
  data() {
    return {
      lineObj: {}
    }
  },
  mounted() {
    this.lineObj = this.lineCanvas()
  },
  methods: {
    lineCanvas () {
      const Line = function (obj) {
        this.lineWidth = 2;
        this.color = '#000';
        this.background = '#fff';
        for (var i in obj) {
          this[i] = obj[i];
        };

        this.canvas = this.el.querySelector('canvas');
        this.canvas.width = this.el.clientWidth;
        this.canvas.height = this.el.clientHeight;
        this.el.prepend(this.canvas);

        this.cxt = this.canvas.getContext('2d');
        this.cxt.fillStyle = this.background;
        this.cxt.fillRect(0, 0, this.canvas.width, this.canvas.height);
        this.cxt.strokeStyle = this.color;
        this.cxt.lineWidth = this.lineWidth;
        this.cxt.lineCap = 'round'; // 线条末端添加圆形线帽，减少线条的生硬感
        this.cxt.lineJoin = 'round'; // 线条交汇时为原型边角
        // 利用阴影，消除锯齿
        this.cxt.shadowBlur = 1;
        this.cxt.shadowColor = '#000';

        // 开始绘制
        this.canvas.addEventListener('touchstart', e => {
          this.canvasBound = this.canvas.getBoundingClientRect();
          this.scrollTop = window.scrollY

          this.cxt.beginPath();
          this.cxt.moveTo(e.changedTouches[0].pageX - this.canvasBound.left, e.changedTouches[0].pageY - this.canvasBound.top - this.scrollTop);
        }, false);

        // 绘制中
        this.canvas.addEventListener('touchmove', e => {
          this.cxt.lineTo(e.changedTouches[0].pageX - this.canvasBound.left, e.changedTouches[0].pageY - this.canvasBound.top - this.scrollTop);
          this.cxt.stroke();

          e.preventDefault();
        }, false);

        // 结束绘制
        this.canvas.addEventListener('touchend', e => {
          this.cxt.closePath();
        }, false);
      }

      Line.prototype.clear = function () {
        this.cxt.clearRect(0, 0, this.canvas.width, this.canvas.height);
      }

      Line.prototype.save = function () {
        var imgBase64 = this.canvas.toDataURL();
        var imgPng = this.canvas.toDataURL('image/png');
        var imgJpg = this.canvas.toDataURL('image/jpeg', 0.8);

        return {
          png: imgPng,
          jpg: imgJpg
        }
      }

      return new Line({
        el: document.querySelector('.' + this.signClass)
      })
    },
    save () {
      return this.lineObj.save()
    },
    clear () {
      this.lineObj.clear()
    }
  }
}
</script>

<style lang="scss" scoped>
  .sign {
    border: solid 1px #eee;
    border-radius: 5px;
    margin-bottom: 10px;
    canvas {
      display: block;
      width: 100%;
      height: 100%;
    }
  }
</style>
