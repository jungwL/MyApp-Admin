<script setup>
import { ref } from 'vue';

// 부모 컴포넌트(App.vue)로 이벤트를 보내기 위해 defineEmits를 사용
const emit = defineEmits(['menu-changed']);

const activeMenuId = ref('dashboard');
const menuItems = ref([
  { id: 'dashboard', name: '대시보드', icon: '📊' },
  { id: 'users', name: '사용자 관리', icon: '👥' },
  { id: 'products', name: '상품 관리', icon: '📦' },
  { id: 'qna', name: 'QnA 관리', icon: '📝' },
  { id: 'settings', name: '설정', icon: '⚙️' }
]);

function changeMenu(menuItem) {
  activeMenuId.value = menuItem.id;
  // 'menu-changed'라는 이름으로 menuItem 객체를 부모에게 보냄
  emit('menu-changed', menuItem);
}
</script>

<template>
  <aside class="sidebar">
    <h2>Admin</h2>
    <nav>
      <ul>
        <li v-for="item in menuItems" :key="item.id">
          <a @click="changeMenu(item)" :class="{ active: item.id === activeMenuId }">
            {{ item.icon }} {{ item.name }}
          </a>
        </li>
      </ul>
    </nav>
  </aside>
</template>

<style scoped>
/* 이 스타일은 Sidebar.vue 컴포넌트 안에서만 적용됩니다. */
.sidebar {
  width: 240px;
  background-color: #2c3e50;
  color: #ecf0f1;
  height: 100vh;
  padding: 20px;
}
.sidebar h2 {
  text-align: center;
  margin-bottom: 30px;
  font-size: 1.8em;
}
.sidebar ul {
  list-style: none;
}
.sidebar ul li a {
  display: block;
  color: #ecf0f1;
  text-decoration: none;
  padding: 15px 20px;
  border-radius: 5px;
  margin-bottom: 10px;
  transition: background-color 0.3s;
  cursor: pointer;
}
.sidebar ul li a:hover,
.sidebar ul li a.active {
  background-color: #34495e;
}
</style>