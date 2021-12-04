# VueSwiper 轮播图

- 安装依赖包

  ```bash
  npm install swiper@5.3.8 vue-awesome-swiper@4.1.1 --save-dev
  ```

- SwiperDemo.vue

  ```html
  <template>
    <swiper class="swiper-demo" ref="mySwiper" :options="swiperOptions">
      <swiper-slide>
        <img src="/imgs/slide-1.jpg" alt />
      </swiper-slide>
      <swiper-slide>
        <img src="/imgs/slide-2.jpg" alt />
      </swiper-slide>
      <swiper-slide>
        <img src="/imgs/slide-3.jpg" alt />
      </swiper-slide>
      <!-- 分页器 -->
      <div class="swiper-pagination" slot="pagination"></div>
      <!-- 切换按钮 -->
      <div class="swiper-button-prev" slot="button-prev"></div>
      <div class="swiper-button-next" slot="button-next"></div>
    </swiper>
  </template>
  <script>
    // 导入轮播组件和样式
    import { Swiper, SwiperSlide } from "vue-awesome-swiper";
    import "swiper/css/swiper.css";
    export default {
      name: "swiper-demo",
      components: {
        Swiper,
        SwiperSlide,
      },
      data() {
        return {
          swiperOptions: {
            autoplay: {
              // 自动播放
              delay: 1000, //1秒切换一次
            },
            loop: true, // 循环播放
            effect: "cube", // 切换效果 立方体
            cubeEffect: {
              slideShadows: true, // 开启slide阴影
              shadow: true, // 开启投影
              shadowOffset: 100, // 投影距离
              shadowScale: 0.6, // 投影缩放比例
            },
            pagination: {
              el: ".swiper-pagination", // 绑定分页器
              clickable: true, // 点击分页器切换
            },
            navigation: {
              nextEl: ".swiper-button-next", // 绑定下一页
              prevEl: ".swiper-button-prev", // 绑定上一页
            },
          },
        };
      },
      computed: {
        swiper() {
          return this.$refs.mySwiper.$swiper;
        },
      },
      mounted() {
        // 鼠标覆盖停止自动切换
        this.swiper.el.onmouseover = () => {
          this.swiper.autoplay.stop();
        };
        //鼠标离开开始自动切换
        this.swiper.el.onmouseout = () => {
          this.swiper.autoplay.start();
        };
      },
    };
  </script>
  <style lang="scss" scoped>
    .swiper-demo {
      width: 1100px;
      height: 400px;
      img {
        height: 100%;
        width: 100%;
      }
    }
  </style>
  ```
