<script setup>
import { ref, computed } from 'vue';
import Sidebar from './components/Sidebar.vue';

// 각 페이지 컴포넌트들을 import 합니다.
import Dashboard from './components/pages/Dashboard.vue';
import UserManagement from './components/pages/UserManagement.vue';
import ProductManagement from './components/pages/ProductManagement.vue';
import Settings from './components/pages/Settings.vue';
import QnaManagement from './components/pages/QnaManagement.vue';

// 컴포넌트들을 객체에 매핑합니다.
const pages = {
  dashboard: Dashboard,
  users: UserManagement,
  products: ProductManagement,
  qna : QnaManagement,
  settings: Settings,
};

// 현재 활성화된 메뉴의 ID를 저장합니다. 초기값은 'dashboard'.
const activePageId = ref('dashboard');

// activePageId 값에 따라 보여줄 컴포넌트를 동적으로 결정합니다.
const activeComponent = computed(() => {
  return pages[activePageId.value];
});

// Sidebar에서 메뉴가 변경되었다는 이벤트가 오면 activePageId를 업데이트합니다.
function handleMenuChange(menuItem) {
  activePageId.value = menuItem.id;
  console.log(menuItem.name + ' 페이지로 변경');
}
</script>

<template>
  <div class="app-container">
    <Sidebar @menu-changed="handleMenuChange" />

    <main class="main-content">
      <component :is="activeComponent" />
    </main>
  </div>
</template>

<style scoped>
.app-container {
  display: flex;
}
.main-content {
  flex-grow: 1;
  padding: 30px;
}
</style>