<template>
  <div class="md-splash">
    <!-- <header class="header">
      <h2 class="title">启动页</h2>
    </header> -->
    <swiper class="md-splash__swiper" indicator-dots>
      <swiper-item class="md-splash__item" v-for="(item, index) in movies" :for-index="index" :key="item.id">
        <image :src="item.images.large" class="md-splash__image" mode="aspectFit"/>
        <button class="md-splash__start" @click="handleStart" v-if="index === movies.length - 1">立即体验</button>
      </swiper-item>
    </swiper>
  </div>
</template>

<script>
/* eslint-disable */
import { getStorage, setStorage } from '@/utils/wechat';
import { getBoardData } from '@/utils/api';
const LAST_SPLASH_DATA = 'LAST_SPLASH_DATA';
export default {
  data() {
    return {
      movies: []
    };
  },
  methods: {
    async getCache() {
      try {
        const res = await getStorage(LAST_SPLASH_DATA);
        const { movies, expires } = res.data;

        // 有缓存，判断是否过期
        if (movies && expires > Date.now()) {
          return res.data;
        }
        // 已经过期或者没有缓存
        console.log('uncached');
        return null;
      } catch (error) {
        return null;
      }
    },
    handleStart() {
      console.log('click start');

      // TODO: 访问历史的问题
      wx.switchTab({
        url: '../board/main'
      });
    },
    async getInitData() {
      const cache = await this.getCache();
      if (cache) {
        this.movies = cache.movies;
        return;
      }
      const data = await getBoardData({
        board: 'coming_soon',
        page: 1,
        count: 3
      });

      this.movies = data.subjects;
      await setStorage(LAST_SPLASH_DATA, {
        movies: data.subjects,
        expires: Date.now() + 1 * 24 * 60 * 60 * 1000 // 缓存一天
      });
    }
  },
  mounted() {
    this.getInitData();
  }
};
/* eslint-enable */
</script>

<style lang="scss">
@import 'node_modules/sass-bem/bem';
$bem-component-namespace: 'md';

page {
  height: 100%;
}

@include c('splash') {
  height: 100%;

  @include e('swiper') {
    height: 100%;
  }

  @include e('item') {
    flex: 1;
  }

  @include e('image') {
    position: absolute;
    height: 100%;
    width: 100%;
    opacity: 0.9;
  }

  @include e('start') {
    position: absolute;
    bottom: 200rpx;
    left: 50%;
    width: 300rpx;
    margin-left: -150rpx;
    background-color: rgba(64, 88, 109, 0.4);
    color: #fff;
    border: 1rpx solid rgba(64, 88, 109, 0.8);
    border-radius: 200rpx;
    font-size: 40rpx;
  }
}
</style>
