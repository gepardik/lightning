<template>
  <div
    v-cloak
    class="swiper-parent"
    :style="{ height: slideHeight + 'px' }"
    v-if="articles.length"
  >
    <section
      :class="{
        'swiper-slide': true,
        wide: wideScreen,
        active: activeSlide === idx + 1,
      }"
      v-for="(article, idx) in articles"
      :key="idx"
      :data-id="article.id"
      :id="'swiper-article-' + (idx + 1)"
      :style="{
        background: `radial-gradient(circle at 20% center, rgba(0, 0, 0, 0), rgba(0, 0, 0, .3), rgba(0, 0, 0, .8)),
          url('${article.image[orientation]}') no-repeat`,
        backgroundColor: !article.image[orientation] && '#0063FC',
      }"
    >
      <a
        class="article-rubric"
        @click="
          $emit(
            'openUrl',
            'https://' +
              article.section.domain +
              '/section/' +
              article.section.id
          )
        "
        v-if="article.section"
      >
        {{ article.section.name }}
      </a>
      <a
        class="lightning-headline"
        @click="
          $emit('openUrl', 'https://' + rootSection.domain + '/' + article.id)
        "
      >
        <span class="blue-font article-exclamation" v-if="article.exclamation">
          {{ article.exclamation }} <span>&rang;</span>
        </span>
        {{ article.headline }}
        <span class="blue-font" v-if="article.commentCount > 0"
          >({{ article.commentCount }})</span
        >
      </a>
      <div
        :class="['article-shadow', { invisible: activeSlide === idx + 1 }]"
      ></div>
    </section>
    <section
      :class="[
        'swiper-slide',
        { wide: wideScreen, active: activeSlide === articles.length + 1 },
        'final',
      ]"
      :key="articles.length + 1"
      :data-id="0"
      :id="'swiper-article-' + (articles.length + 1)"
      style="background: #000;"
    >
      <span>Check if there are any news</span>
      <button @click="$emit('reloadNews')" class="reload-news-button">
        Refresh
        <img src="/v5/img/icons/lightning/icon_uuenda.svg" alt="reload icon" />
      </button>
    </section>
  </div>
</template>

<script>
export default {
  name: 'lightning-swiper',
  props: ['articles'],
  data: () => ({
    orientation: 'portrait',
    slideHeight: window.innerHeight,
    activeSlide: 1,
    wideScreen: false,
    timer: null,
  }),
  created() {
    if (window.location.hash) {
      window.location.hash
        .substr(1)
        .split('&')
        .forEach(hash => {
          hash = hash.split('=');
          if (hash[0] === 'article' && !isNaN(parseInt(hash[1]))) {
            let activeArticleId = +hash[1];
            let currentArticle = this.articles.filter(
              article => article.id === activeArticleId
            )[0];
            this.activeSlide =
              (currentArticle && this.articles.indexOf(currentArticle) + 1) ||
              1;
          }
        });
    }
  },
  mounted() {
    let startY;
    let endY;
    const screenWidth = window.innerWidth;
    const screenHeight = window.innerHeight;
    this.orientation = this.getOrientation();
    this.wideScreen =
      Math.max(screenWidth, screenHeight) /
        Math.min(screenWidth, screenHeight) >
      1.8;
    window.addEventListener(
      'touchstart',
      evt => {
        startY = evt.touches[0].clientY;
      },
      { passive: false }
    );

    window.addEventListener('touchend', evt => {
      endY = evt.changedTouches[0].clientY;
      handleTouch(evt);
    });

    const handleTouch = () => {
      if (endY - startY < -10) {
        if (this.activeSlide <= this.articles.length) {
          this.scrollSlides('up', this.activeSlide + 1);
        }
      } else if (endY - startY > 10) {
        if (this.activeSlide > 1) {
          this.scrollSlides('down', this.activeSlide - 1);
        }
      }
    };

    if (this.activeSlide > 1) {
      this.scrollSlides('none', this.activeSlide);
    }

    window.addEventListener('resize', () => {
      this.slideHeight = window.innerHeight;
      this.orientation = this.getOrientation();
      this.wideScreen =
        Math.max(window.innerWidth, window.innerHeight) /
          Math.min(window.innerWidth, window.innerHeight) >
        1.8;
      const slides = document.querySelectorAll('.swiper-slide');
      slides.forEach(slide => {
        slide.style.transform = `translateY(-${this.slideHeight *
          (this.activeSlide - 1)}px)`;
      });
    });
  },
  methods: {
    scrollSlides(direction, targetSlide) {
      const targetSlideElement = document.querySelector(
        '#swiper-article-' + targetSlide
      );
      let offset = targetSlideElement.offsetTop;
      const slides = document.querySelectorAll('.swiper-slide');
      slides.forEach(slide => {
        slide.style.transform = `translateY(-${offset}px)`;
      });

      if (targetSlide !== this.activeSlide) {
        direction === 'down' ? this.activeSlide-- : this.activeSlide++;
      }
    },

    getOrientation() {
      if (screen.orientation) {
        return screen.orientation.type.split('-')[0];
      } else {
        return window.innerHeight >= window.innerWidth
          ? 'portrait'
          : 'landscape';
      }
    },
  },
  watch: {
    activeSlide() {
      const activeSlideElement = document.querySelector(
        '#swiper-article-' + this.activeSlide
      );
      const activeArticleId = activeSlideElement.dataset.id;
      this.$emit('slideChange', activeArticleId);
    },
  },
};
</script>

<style scoped>
[v-cloak] {
  display: none;
}

.swiper-parent {
  box-sizing: border-box;
  overflow: hidden;
  background-color: #000;
}

section.swiper-slide {
  height: 100%;
  z-index: 1;
  position: relative;
  font-size: 18px;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  align-items: center;
  background-position: top center !important;
  transition: background 0.4s ease 0.2s, transform 0.5s ease;
}

section.swiper-slide.final {
  padding: 35px;
  color: #fff;
  font-size: 20px;
  line-height: 26px;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.article-shadow {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: 4;
  width: 100%;
  height: 100%;
  background: #000;
  opacity: 1;
  transition: opacity 0.4s;
}

.article-shadow.invisible {
  opacity: 0;
  z-index: -1;
}

section.swiper-slide.final > button {
  background-color: #009ee2;
  color: #fff;
  font-size: 18px;
  font-weight: 500;
  line-height: 26px;
  border: none;
  padding: 8px 32px;
  border-radius: 22px;
  margin-top: 35px;
  display: flex;
  align-items: center;
}

section.swiper-slide.final > button img {
  margin-left: 10px;
}

.lightning-headline {
  display: block;
  text-decoration: none;
  color: #fff;
  font-size: 20px;
  line-height: 26px;
  font-weight: 800;
  padding-left: 15px;
  padding-left: calc(15px + constant(safe-area-inset-left));
  padding-left: calc(15px + env(safe-area-inset-left));
  padding-right: 40px;
  padding-right: calc(40px + constant(safe-area-inset-right));
  padding-right: calc(40px + env(safe-area-inset-right));
  padding-bottom: 20px;
  padding-bottom: calc(20px + constant(safe-area-inset-bottom));
  padding-bottom: calc(20px + env(safe-area-inset-bottom));
  max-width: calc(100% - 55px);
  overflow: hidden;
  word-break: break-word;
}

@media screen and (min-width: 414px) {
  .lightning-headline {
    font-size: 26px;
    line-height: 32px;
  }
}

@media screen and (orientation: portrait) {
  .swiper-slide {
    background-size: 150% auto !important;
  }

  .swiper-slide.active {
    background-size: 100% auto !important;
  }

  .swiper-slide.wide {
    background-size: auto 150% !important;
  }

  .swiper-slide.wide.active {
    background-size: auto 100% !important;
  }
}

@media screen and (orientation: landscape) {
  .swiper-slide {
    background-size: auto 150% !important;
  }

  .swiper-slide.active {
    background-size: auto 100% !important;
  }

  .swiper-slide.wide {
    background-size: 150% auto !important;
  }

  .swiper-slide.wide.active {
    background-size: 100% auto !important;
  }
}

.blue-font {
  color: #0063FC;
}

.article-rubric {
  display: block;
  text-decoration: none;
  color: #fff;
  background-color: #0063FC;
  font-size: 11px;
  line-height: 15px;
  text-transform: uppercase;
  padding: 5px 10px 3px 10px;
  border-radius: 11px;
  height: 21px;
  margin-left: 15px;
  margin-left: calc(15px + constant(safe-area-inset-left));
  margin-left: calc(15px + env(safe-area-inset-left));
  margin-bottom: 10px;
}

.article-exclamation {
  font-weight: 700;
  text-transform: uppercase;
}

.article-exclamation > span {
  font-weight: 100;
}
</style>
