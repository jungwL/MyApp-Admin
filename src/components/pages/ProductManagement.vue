<script setup>
import { ref, onMounted, computed } from 'vue';

// --- 상태 변수 정의 ---
// ref()로 감싸진 변수들은 모두 '마법 상자'가 됩니다.
const breadData = ref(null);    // 빵 데이터 전체를 담을 상자
const isLoading = ref(true);     // 로딩 중인지 기억하는 상자 (true/false)
const activeTab = ref('cake');   // 지금 어떤 탭이 선택됐는지 기억하는 상자
const error = ref(null);

// --- 상품 추가 모달을 위한 상태 변수 ---
const isAddModalVisible = ref(false);
const isSubmitting = ref(false);
const newProduct = ref({
  name: '',
  ingredients: '',
  price: 0,
  category: 'cake'
});
const newProductImage = ref(null);
const imagePreviewUrl = ref('');

const categories = ref([
  { key: 'cake', name: '케이크' },
  { key: 'baguette', name: '바게트' },
  { key: 'bread', name: '빵' },
  { key: 'sand', name: '샌드위치' },
]);

// --- 오늘의빵 리스트 요청API ---
async function fetchBreadData() {
  isLoading.value = true;
  try {
    const response = await fetch('http://localhost:8080/api/admin/todayBreadList/all');
    if (!response.ok) throw new Error(`서버 에러: ${response.status}`);
    breadData.value = await response.json(); // 빵정보 JSON데이터를 breadData에 담는다.
  } catch (e) {
    error.value = '상품 데이터를 불러오는 데 실패했습니다.';
  } finally {
    isLoading.value = false;
  }
}

// --- 활성화된 탭에 맞는 상품 목록 ---
// computed는 ref나 reactive 같은 반응형 데이터에 의존하는 '계산된 속성'을 만듭니다.
// 의존하는 데이터(여기서는 breadData, activeTab)가 변경되면,
// computed는 자동으로 재계산되어 항상 최신 상태를 유지합니다.
const activeBreadList = computed(() => {
  // 이 함수는 activeTab이나 breadData가 변경될 때마다 자동으로 실행됩니다.
  // --- 삼항 연산자를 사용한 안전한 데이터 접근 ---
  // (조건) ? 참일_때_값 : 거짓일_때_값
  return (
      // [조건]
      // 1. breadData.value: 서버로부터 받은 전체 데이터가 존재하는지 확인합니다. (null이 아닌지)
      //    데이터가 아직 로드되지 않았을 때 발생할 수 있는 에러를 방지합니다.
      // 2. breadData.value[activeTab.value]: 전체 데이터 안에 현재 활성화된 탭(예: 'cake')에 해당하는 키와 값이 존재하는지 확인합니다.
      breadData.value && breadData.value[activeTab.value]
  )
      ? // [참일 때]
        // 조건이 모두 참이면, breadData 객체에서 현재 활성화된 탭 이름(예: 'cake')을 키로 사용하여
        // 해당하는 빵 목록 배열(예: 케이크 목록)을 반환합니다.
      breadData.value[activeTab.value]
      : // [거짓일 때]
        // 조건 중 하나라도 거짓이면 (데이터가 아직 없거나, 해당 카테고리가 없으면)
        // 에러를 발생시키는 대신, 안전하게 빈 배열([])을 반환하여 화면에 아무것도 표시되지 않도록 합니다.
      [];
});
// --- 상품 삭제 호출 함수 ---
async function deleteProduct(breadId, categoryKey) {
  if (!confirm(`정말로 이 상품(ID: ${breadId})을 삭제하시겠습니까?`)) {
    return;
  }
  try {
    const response = await fetch(`http://localhost:8080/api/admin/todayBreadDel/${breadId}`, {
      method: 'DELETE',
    });
    if (!response.ok) {
      throw new Error('상품 삭제에 실패했습니다.');
    }
    const index = breadData.value[categoryKey].findIndex(b => b.id === breadId);
    if (index > -1) {
      breadData.value[categoryKey].splice(index, 1);
    }
    alert('상품이 성공적으로 삭제되었습니다.');
  } catch (e) {
    console.error('상품 삭제 중 에러 발생:', e);
    alert(e.message);
  }
}

// --- 상품 추가 모달을 여는 함수 ---
function openAddProductModal() {
  newProduct.value = { name: '', ingredients: '', price: 0, category: 'cake' }; //select 박스
  newProductImage.value = null;
  imagePreviewUrl.value = '';
  isAddModalVisible.value = true;
}

// --- 이미지 파일 선택 시 처리하는 함수 ---
function handleImageUpload(event) {
  const file = event.target.files[0];
  if (file) {
    newProductImage.value = file;
    const reader = new FileReader();
    reader.onload = (e) => {
      imagePreviewUrl.value = e.target.result;
    };
    reader.readAsDataURL(file);
  }
}
// --- 새로운 상품을 서버로 전송하는 함수 ---
async function submitNewProduct() {
  isSubmitting.value = true;
  try {
    const response = await fetch('http://localhost:8080/api/admin/todayBreadSub', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(newProduct.value),
    });
    if (!response.ok) {
      throw new Error('상품 등록에 실패했습니다.');
    }
    alert('상품이 성공적으로 등록되었습니다.');
    isAddModalVisible.value = false;
    await fetchBreadData();
  } catch (e) {
    alert(e.message);
  } finally {
    isSubmitting.value = false;
  }
}

// --- Helper 함수 ---
function getImageUrl(imagePath) {
  return `http://localhost:8081/uploads/${imagePath}`;
}
function formatPrice(price) {
  return `${price.toLocaleString('ko-KR')}원`;
}

onMounted(fetchBreadData);
</script>

<template>
  <div>
    <h1>📦 상품 관리</h1>
    <div class="content-box">
      <!-- 탭 메뉴 -->
      <div class="tabs">
        <button v-for="category in categories" :key="category.key" :class="{ active: activeTab === category.key }" @click="activeTab = category.key" class="tab-button">{{ category.name }}</button>
      </div>

      <div v-if="isLoading" class="feedback-box"><p>🔄 상품 목록을 불러오는 중입니다...</p></div> <!--로딩중일경우 -->
      <div v-else-if="error" class="feedback-box error"><p>⚠️ {{ error }}</p></div><!--에러난 경우 -->

      <div v-else class="tab-content">
        <table v-if="activeBreadList.length > 0" class="product-table">
          <thead>
          <tr>
            <th>ID</th>
            <th>이미지</th>
            <th>상품명</th>
            <th>가격</th>
            <th>평균 별점</th>
            <th>삭제</th>
          </tr>
          </thead>
          <tbody>
          <tr v-for="bread in activeBreadList" :key="bread.id">
            <td>{{ bread.id }}</td>
            <td><img :src="getImageUrl(bread.image)" :alt="bread.name" class="product-image" /></td>
            <td>{{ bread.name }}</td>
            <td>{{ formatPrice(bread.price) }}</td>
            <td><span class="star-rating">⭐ {{ bread.avgRating.toFixed(1) }}</span></td>
            <td>
              <button @click.stop="deleteProduct(bread.id, activeTab)" class="delete-button" title="삭제">🗑️</button>
            </td>
          </tr>
          </tbody>
        </table>
        <p v-else class="no-data">해당 카테고리에 상품이 없습니다.</p>

        <!-- 상품 추가 버튼 -->
        <div class="actions-container">
          <button @click="openAddProductModal" class="add-button">+ 상품 추가</button>
        </div>
      </div>
    </div>

    <!-- 상품 추가 슬라이드 팝업(모달) -->
    <div v-if="isAddModalVisible" class="modal-overlay" @click.self="isAddModalVisible = false">
      <div class="modal-slide-panel">
        <h2>새 상품 등록</h2>
        <form @submit.prevent="submitNewProduct" class="add-product-form">
          <div class="form-group">
            <label for="category">카테고리</label>
            <select id="category" v-model="newProduct.category">
              <option v-for="cat in categories" :key="cat.key" :value="cat.key">{{ cat.name }}</option>
            </select>
          </div>
          <div class="form-group">
            <label for="name">상품명</label>
            <input type="text" id="name" v-model="newProduct.name" required />
          </div>
          <div class="form-group">
            <label for="price">가격</label>
            <input type="number" id="price" v-model.number="newProduct.price" required min="0" />
          </div>
          <div class="form-group">
            <label for="ingredients">재료/설명</label>
            <textarea id="ingredients" v-model="newProduct.ingredients" rows="4"></textarea>
          </div>
         <!-- <div class="form-group">
            <label for="image">상품 이미지</label>
            <input type="file" id="image" @change="handleImageUpload" accept="image/*" required />
            <div v-if="imagePreviewUrl" class="image-preview">
              <img :src="imagePreviewUrl" alt="이미지 미리보기" />
            </div>
          </div>-->
          <div class="form-group">
            <label for="image">상품 이미지 경로</label>
            <input
                type="text"
                id="image"
                v-model="newProduct.image"
                placeholder="예: images/cake1.jpg"
                required
            />
          </div>
          <div class="form-actions">
            <button type="button" @click="isAddModalVisible = false" class="btn btn-secondary">취소</button>
            <button type="submit" :disabled="isSubmitting" class="btn btn-primary">{{ isSubmitting ? '저장 중...' : '저장' }}</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

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
  vertical-align: middle;
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

/* 추가된 스타일 */
.delete-button {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 20px;
  padding: 5px;
  border-radius: 50%;
  transition: background-color 0.2s;
}
.delete-button:hover {
  background-color: #ffcdd2; /* Light red */
}
.actions-container {
  display: flex;
  justify-content: flex-end;
  margin-top: 20px;
}
.add-button {
  padding: 10px 20px;
  background-color: #4CAF50; /* Green */
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
  font-size: 16px;
  transition: background-color 0.2s;
}
.add-button:hover {
  background-color: #45a049;
}

/* 모달 스타일 추가 */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 1000;
}
.modal-slide-panel {
  position: fixed;
  top: 0;
  right: 0;
  width: 100%;
  max-width: 450px;
  height: 100%;
  background-color: white;
  box-shadow: -5px 0 15px rgba(0,0,0,0.2);
  padding: 30px;
  overflow-y: auto;
  animation: slide-in 0.3s ease-out;
}
.modal-slide-panel h2 {
  margin-top: 0;
  margin-bottom: 30px;
  color: #333;
}
.add-product-form .form-group {
  margin-bottom: 20px;
}
.add-product-form label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #555;
}
.add-product-form input[type="text"],
.add-product-form input[type="number"],
.add-product-form select,
.add-product-form textarea {
  width: 100%;
  padding: 12px;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 1em;
  box-sizing: border-box; /* padding이 width에 포함되도록 설정 */
}
.add-product-form input[type="file"] {
  padding: 8px;
}
.image-preview {
  margin-top: 15px;
}
.image-preview img {
  max-width: 100%;
  max-height: 200px;
  border-radius: 5px;
  border: 1px solid #ddd;
}
.form-actions {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  margin-top: 30px;
}
.btn { padding: 12px 24px; border: none; border-radius: 5px; cursor: pointer; font-weight: bold; }
.btn-primary { background-color: #007bff; color: white; }
.btn-secondary { background-color: #6c757d; color: white; }
.btn:disabled { background-color: #ccc; }

@keyframes slide-in {
  from { transform: translateX(100%); }
  to { transform: translateX(0); }
}
</style>

