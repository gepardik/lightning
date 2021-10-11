<template>
  <div class="lightning-nav-container">
    <div class="nav-back" onclick="window.history.back()"></div>
    <ul class="lightning-nav">
      <li
        @click="$emit('setCurrentTab', 'editorsChoice')"
        :class="{ 'active-tab': currentTab === 'editorsChoice' }"
      >
        Editors choice
      </li>
      <li
        @click="$emit('setCurrentTab', 'latest')"
        :class="{ 'active-tab': currentTab === 'latest' }"
      >
        Latest
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'navigation',
  props: ['currentTab'],
};
</script>

<style scoped>
.lightning-nav-container {
  z-index: 5;
  position: fixed;
  width: 100%;
  margin-top: 16px;
  margin-top: calc(16px + constant(safe-area-inset-top));
  margin-top: calc(16px + env(safe-area-inset-top));
  display: flex;
  justify-content: center;
}

.nav-back {
  width: 8px;
  height: 14px;
  background: url('/v5/img/icons/lightning/icon_back_arrow.svg') top left /
    cover no-repeat;
  position: absolute;
  left: 15px;
  left: calc(15px + constant(safe-area-inset-left));
  left: calc(15px + env(safe-area-inset-left));
  top: 2px;
}

.lightning-nav {
  text-align: center;
  text-transform: uppercase;
}

.lightning-nav li {
  display: inline-block;
  color: rgba(255, 255, 255, 0.5);
  padding: 0 10px;
  font-size: 11px;
}

.lightning-nav li:nth-child(2) {
  border-left: 1px solid #fff;
}

.lightning-nav li.active-tab {
  color: rgba(255, 255, 255, 1);
}
</style>
