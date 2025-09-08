<template>
  <div>
    <h1>📦 상품 관리</h1>
    <div class="content-box">
      <!-- 탭 메뉴 -->
      <div class="tabs">
        <button
            v-for="category in categories"
            :key="category.key"
            :class="{ active: activeTab === category.key }"
            @click="activeTab = category.key"
            class="tab-button"
        >
          {{ category.name }}
        </button>
      </div>

      <!-- 로딩 중일 때 -->
      <div v-if="isLoading" class="feedback-box">
        <p>🔄 상품 목록을 불러오는 중입니다...</p>
      </div>

      <!-- 에러 발생 시 -->
      <div v-else-if="error" class="feedback-box error">
        <p>⚠️ {{ error }}</p>
      </div>

      <!-- 데이터가 성공적으로 로드되었을 때 -->
      <div v-else class="tab-content">
        <p v-if="activeBreadList.length === 0" class="no-data">
          해당 카테고리에 상품이 없습니다.
        </p>
        <table v-else class="product-table">
          <thead>
          <tr>
            <th>ID</th>
            <th>이미지</th>
            <th>상품명</th>
            <th>가격</th>
            <th>평균 별점</th>
          </tr>
          </thead>
          <tbody>
          <tr v-for="bread in activeBreadList" :key="bread.id">
            <td>{{ bread.id }}</td>
            <td>
              <img :src="getImageUrl(bread.image)" :alt="bread.name" class="product-image" />
            </td>
            <td>{{ bread.name }}</td>
            <td>{{ formatPrice(bread.price) }}</td>
            <td>
              <span class="star-rating">⭐ {{ bread.avgRating.toFixed(1) }}</span>
            </td>
          </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';

// --- 상태 변수 정의 ---
const breadData = ref(null); // 서버에서 받은 전체 카테고리별 데이터
const isLoading = ref(true); // 로딩 상태
const error = ref(null); // 에러 메시지
const activeTab = ref('cake'); // 기본으로 활성화될 탭

// --- 탭 메뉴 정의 ---
const categories = ref([
  { key: 'cake', name: '케이크' },
  { key: 'baguette', name: '바게트' },
  { key: 'bread', name: '빵' },
  { key: 'sand', name: '샌드위치' },
]);

// --- 서버에서 데이터 가져오는 함수 ---
async function fetchBreadData() {
  try {
    const response = await fetch('http://localhost:8080/api/bread/today');
    if (!response.ok) {
      throw new Error(`서버 에러: ${response.status}`);
    }
    breadData.value = await response.json();
  } catch (e) {
    console.error('상품 데이터 로딩 실패:', e);
    error.value = '상품 데이터를 불러오는 데 실패했습니다. 서버 상태를 확인해주세요.';
  } finally {
    isLoading.value = false;
  }
}

// --- 활성화된 탭에 맞는 상품 목록을 계산하는 computed 속성 ---
const activeBreadList = computed(() => {
  if (!breadData.value || !breadData.value[activeTab.value]) {
    return [];
  }
  return breadData.value[activeTab.value];
});

// --- Helper 함수 ---
// 이미지 경로를 완성하는 함수
function getImageUrl(imagePath) {
  // 서버의 정적 리소스 경로에 맞게 URL을 설정합니다.
  return `http://localhost:8080/${imagePath}`;
}
// 가격을 원화 형식으로 변환하는 함수
function formatPrice(price) {
  return `${price.toLocaleString('ko-KR')}원`;
}

// 컴포넌트가 마운트될 때 데이터 로딩 함수를 호출
onMounted(fetchBreadData);
</script>

<style scoped>
/* 기본 컨텐츠 박스 스타일 */
.content-box {
  margin-top: 20px;
  padding: 20px;
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

/* 탭 메뉴 스타일 */
.tabs {
  display: flex;
  border-bottom: 2px solid #e0e0e0;
  margin-bottom: 20px;
}
.tab-button {
  padding: 10px 20px;
  cursor: pointer;
  background-color: transparent;
  border: none;
  font-size: 16px;
  font-weight: 500;
  color: #666;
  position: relative;
  transition: color 0.3s ease;
}
.tab-button::after {
  content: '';
  position: absolute;
  bottom: -2px;
  left: 0;
  width: 100%;
  height: 2px;
  background-color: #8D6E63; /* Brown */
  transform: scaleX(0);
  transition: transform 0.3s ease;
}
.tab-button.active {
  color: #5D4037; /* Darker Brown */
  font-weight: 700;
}
.tab-button.active::after {
  transform: scaleX(1);
}

/* 로딩/에러/데이터 없음 메시지 박스 */
.feedback-box {
  text-align: center;
  padding: 40px;
  font-size: 18px;
  color: #555;
}
.feedback-box.error {
  color: #D32F2F; /* Red */
}
.no-data {
  text-align: center;
  padding: 40px;
  color: #777;
}


/* 상품 테이블 스타일 */
.product-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 10px;
}
.product-table th,
.product-table td {
  padding: 12px 15px;
  text-align: left;
  border-bottom: 1px solid #eee;
}
.product-table th {
  background-color: #f9f9f9;
  font-weight: 600;
  color: #333;
}
.product-table tbody tr:hover {
  background-color: #f5f5f5;
}

/* 테이블 내 이미지 스타일 */
.product-image {
  width: 60px;
  height: 60px;
  object-fit: cover;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.05);
}

/* 별점 스타일 */
.star-rating {
  font-weight: bold;
  color: #FFA000; /* Amber */
}
</style>
