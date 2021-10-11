<template>
  <div class="lightning">
    <Loader v-show="loader" />
    <LightningNavigation
      @setCurrentTab="loadCurrentTab"
      :currentTab="currentTab"
    />
    <LightningSidebar
      v-if="!loader && !finalSlide"
      :currentArticle="currentArticleId"
      @openUrl="openUrl"
    />
    <LightningSwiper
      :articles="articles"
      v-if="articles.length"
      @slideChange="setCurrentArticleId"
      @openUrl="openUrl"
      @reloadNews="reloadNews"
    />
  </div>
</template>

<script>
import LightningNavigation from './lightning-navigation.vue';
import LightningSwiper from './lightning-swiper.vue';
import LightningSidebar from './lightning-sidebar.vue';
import Loader from './Loader.vue';
import axios from 'axios'

export default {
  data: () => ({
    currentTab: 'latest',
    articles: [],
    loader: false,
    currentArticleId: undefined,
    finalSlide: false,
  }),
  name: 'lightning',
  components: {
    LightningNavigation,
    LightningSwiper,
    LightningSidebar,
    Loader,
  },
  created: function() {
    if (window.location.hash) {
      window.location.hash
        .substr(1)
        .split('&')
        .forEach(hash => {
          hash = hash.split('=');
          if (
            hash[0] === 'tab' &&
            ['latest', 'editorsChoice'].includes(hash[1])
          ) {
            this.currentTab = hash[1];
          }
        });

      window.addEventListener('pageshow', evt => {
        if (evt.persisted) {
          document.querySelector('.loader-container').style.display = 'flex';
          document.querySelector('.right-buttons').style.display = 'none';
          window.location.reload();
        }
      });
    }
    this.getArticles();
  },
  methods: {
    openUrl(url) {
      window.location.hash = `tab=${this.currentTab}&article=${this.currentArticleId}`;
      window.location.href = url;
    },
    setCurrentArticleId(articleId) {
      this.currentArticleId = articleId;
      if (+this.currentArticleId === 0) {
        this.finalSlide = true;
      } else {
        this.finalSlide = false;
      }
    },
    loadCurrentTab(tab) {
      this.currentTab = tab;
      this.getArticles();
    },
    reloadNews() {
      window.location.hash = `tab=${this.currentTab}`;
      window.location.reload();
    },
    getArticles: async function() {
      this.loader = true;

      this.articles = [];
      let _articles = [];

      axios
          .get('https://services.postimees.ee/rest/v1/sections/81/articles?limit=36')
          .then(response => {
            if (response.data.length) {
              _articles = response.data
              _articles.forEach(article => {
                let { id, headline, thumbnail, meta, sectionBreadcrumb } = article;
                let image = { portrait: '', landscape: '' };
                image.portrait = thumbnail ? thumbnail.sources.portrait.medium : '';
                image.landscape = thumbnail ? thumbnail.sources.landscape.medium : '';

                let section = sectionBreadcrumb.length
                    ? { ...sectionBreadcrumb[sectionBreadcrumb.length - 1] }
                    : '';
                this.articles.push({
                  id,
                  headline,
                  image,
                  section,
                  commentCount: meta.commentCount || 0,
                  exclamation: meta.exclamation || '',
                });
              });

              this.currentArticleId = !this.currentArticleId && this.articles[0].id;

              this.loader = false;
            }
          })
    },
  },
};
</script>
<style scoped>
.lightning {
  position: fixed;
  width: 100%;
  background-color: #000;
  touch-action: pan-down;
}
</style>
