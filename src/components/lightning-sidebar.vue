<template>
  <div class="right-buttons">
    <a
      @click="
        $emit('openUrl', 'https://www.postimees.ee/' + currentArticle)
      "
      class="read-button"
    >
      Loe
    </a>
    <a v-if="isSharePossible" @click="shareLink" class="share-button">
      Share
    </a>
  </div>
</template>

<script>

export default {
  name: 'lightning-sidebar',
  props: ['currentArticle'],
  data: () => ({
    isSharePossible: false,
  }),
  created() {
    if (navigator.share) {
      this.isSharePossible = true;
    }
  },
  methods: {
    shareLink() {
      navigator.share({
        url: `https://www.postimees.ee/${this.currentArticle}`,
      });
    },
  },
};
</script>

<style scoped>
.right-buttons {
  position: fixed;
  z-index: 5;
  height: 110px;
  right: 15px;
  top: 50%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

@media screen and (orientation: portrait) {
  .right-buttons {
    right: 15px;
    right: calc(15px + constant(safe-area-inset-right));
    right: calc(15px + env(safe-area-inset-right));
  }
}

@media screen and (orientation: landscape) {
  .right-buttons {
    right: 15px;
    right: calc(15px + constant(safe-area-inset-top));
    right: calc(15px + env(safe-area-inset-top));
  }
}

.right-buttons > a {
  display: block;
  font-size: 11px;
  line-height: 13px;
  text-transform: capitalize;
  color: #fff;
  width: 100%;
  height: 33px;
  padding-top: 26px;
  text-align: center;
  box-sizing: border-box;
}

.read-button {
  background: url('/v5/img/icons/lightning/icon_loe.svg') top center no-repeat;
}

.share-button {
  background: url('/v5/img/icons/lightning/icon_share.svg') top center no-repeat;
  padding-top: 33px !important;
}
</style>
