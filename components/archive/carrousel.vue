<template>
  <div :class="{ mobile: isMobile }" class="carrousel">
    <transition name="module" mode="out-in">
      <empty-box v-if="!articleList.length" key="empty" class="article-empty-box">
        <slot>{{ $i18n.text.article.empty }}</slot>
      </empty-box>
      <div
        v-swiper:swiper="swiperOption"
        v-else-if="renderSwiper"
        key="swiper"
        class="swiper index"
        @transitionStart="handleSwiperTransitionStart"
        @transitionEnd="handleSwiperTransitionEnd"
      >
        <div class="swiper-wrapper">
          <div
            v-for="(article, index) in articleList.slice(0, 9)"
            :key="index"
            class="swiper-slide slide-item"
          >
            <div
              :class="{ 'motion-blur-horizontal': transitioning }"
              class="content filter"
            >
              <template v-if="article.ad">
                <a
                  :href="article.url"
                  target="_blank"
                  rel="external nofollow noopener"
                  class="link"
                >
                  <img :src="article.src" :alt="article.title" loading="lazy" >
                  <span class="title">{{ article.title }}</span>
                </a>
              </template>
              <template v-else>
                <nuxt-link :to="`/article/${article.id}`" class="link">
                  <img :src="humanizeThumb(article.thumb)" :alt="article.title" style="height: 210px;">
                  <span class="title">{{ article.title }}</span>
                </nuxt-link>
              </template>
            </div>
          </div>
        </div>
        <div class="swiper-pagination swiper-pagination-clickable swiper-pagination-bullets"/>
      </div>
    </transition>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import adConfig from '~/config/ad.config'
export default {
  name: 'IndexCarrousel',
  props: {
    article: {
      type: Object,
      default: null
    }
  },
  data() {
    return {
      renderSwiper: true,
      transitioning: false,
      swiperOption: {
        autoplay: {
          delay: 3500,
          disableOnInteraction: false
        },
        pagination: {
          clickable: true,
          el: '.swiper-pagination'
        },
        setWrapperSize: true,
        mousewheel: true,
        observeParents: true,
        // 禁用 PC 拖动手指样式
        grabCursor: false,
        // 警用 PC 拖动
        simulateTouch: false,
        preloadImages: false,
        lazy: true
      }
    }
  },
  computed: {
    ...mapState('global', ['imageExt', 'isMobile']),
    articleList() {
      const articles = [...this.article.data.records].slice(0, 9)
      articles.length && articles.splice(2, 0, {
        ad: true,
        ...adConfig.pc.carrousel
      })
      return articles
    }
  },
  activated() {
    this.renderSwiper = true
    this.handleSwiperTransitionEnd()
  },
  deactivated() {
    this.renderSwiper = false
  },
  methods: {
    humanizeThumb(thumb) {
      if (thumb) {
        return thumb
      } else {
        return `${this.cdnUrl}/images/${this.isMobile ? 'mobile-' : ''}thumb-carrousel.jpg`
      }
    },
    handleSwiperTransitionStart() {
      this.transitioning = true
    },
    handleSwiperTransitionEnd() {
      this.transitioning = false
    }
  }
}
</script>

<style lang="scss">
  .index.swiper {
    .swiper-pagination {
      .swiper-pagination-bullet {
        &.swiper-pagination-bullet-active {
          width: 2rem;
          border-radius: 10px;
        }
      }
    }
  }
</style>

<style lang="scss" scoped>
  $pc-carrousel-height: 15em;
  $mobile-carrousel-height: calc((100vw - 2rem) * .35);
  .carrousel {
    height: $pc-carrousel-height;
    margin-bottom: 1em;
    position: relative;
    overflow: hidden;
    background-color: $module-bg;
    > .swiper {
      .slide-item {
        > .content {
          width: 100%;
          height: $pc-carrousel-height;
          position: relative;
          overflow: hidden;
          > .link {
            display: block;
            width: 100%;
            height: 100%;
          }
          img {
            width: 100%;
            @include css3-prefix(transform, scale(1));
            @include css3-prefix(transition, transform .88s);
            &:hover {
              @include css3-prefix(transform, scale(1.06));
            }
          }
          .title {
            position: absolute;
            margin: 0;
            top: 1.5rem;
            right: 2rem;
            color: $link-color;
            padding-right: .6em;
            padding-left: 1em;
            height: 2em;
            line-height: 2em;
            font-size: 1em;
            font-weight: bold;
            border-radius: 1px;
            letter-spacing: .3px;
            max-width: 75%;
            @include text-overflow;
            -webkit-background-clip: text;
            // background-color: $module-hover-bg-opacity-9;
            background: linear-gradient(90deg, transparent 0%, $module-bg 2em, $module-bg-opacity-9, $white);
            &:hover {
              color: $text-darken;
              // background: none;
              padding-left: .6em;
              background-color: $module-bg;
            }
          }
        }
      }
    }
    &.mobile {
      height: $mobile-carrousel-height;
      > .swiper {
        .slide-item {
          > .content {
            height: $mobile-carrousel-height;
            > .title {
              right: 1.7rem;
              max-width: 70%;
            }
          }
        }
      }
    }
  }
</style>
