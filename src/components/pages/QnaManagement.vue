<script setup>
import { ref, onMounted, computed } from 'vue';

// --- 데이터 저장을 위한 반응형 변수들 ---
const qnaList = ref([]);
const isLoading = ref(true);
const error = ref(null);

// --- 검색 필터 기능을 위해 추가된 코드 ---
const searchQuery = ref(''); // 사용자가 입력할 검색어를 저장할 변수

// --- Computed 속성: 검색어에 따라 필터링된 QnA 목록을 반환 ---
const filteredQnaList = computed(() => {
  // 원본 데이터가 없으면 빈 배열 반환
  if (!qnaList.value) {
    return [];
  }
  // 검색어가 입력되지 않았으면 원본 목록 전체 반환
  if (!searchQuery.value) {
    return qnaList.value;
  }

  // 검색어를 소문자로 변환 (대소문자 구분 없는 검색을 위함)
  const lowerCaseQuery = searchQuery.value.toLowerCase();

  // 원본 qnaList를 필터링
  return qnaList.value.filter(qna => {
    // 이름(name) 또는 연락처(phone)에 검색어가 포함되어 있는지 확인
    const nameMatch = qna.name && qna.name.toLowerCase().includes(lowerCaseQuery);
    const phoneMatch = qna.phone && qna.phone.includes(lowerCaseQuery);

    // 둘 중 하나라도 일치하면 목록에 포함
    return nameMatch || phoneMatch;
  });
});

// --- 서버에서 데이터를 가져오는 함수 ---
async function fetchQnaData() {
  try {
    const response = await fetch('http://localhost:8080/api/qna/list/all');
    if (!response.ok) {
      throw new Error('데이터를 불러오는데 실패했습니다.');
    }
    const data = await response.json();
    qnaList.value = data;
  } catch (e) {
    error.value = e.message;
    console.error("QnA 데이터 로딩 중 에러 발생:", e);
  } finally {
    isLoading.value = false;
  }
}

// 컴포넌트가 화면에 로드될 때 데이터를 가져옴
onMounted(() => {
  fetchQnaData();
});
</script>

<template>
  <div>
    <h1>📝 QnA 관리</h1>
    <div class="content-box">

      <div class="filter-container">
        <input
            type="text"
            v-model="searchQuery"
            placeholder="이름 또는 연락처로 검색하세요"
            class="search-input"
        />
      </div>

      <div v-if="isLoading">
        <p>데이터를 불러오는 중입니다...</p>
      </div>
      <div v-else-if="error">
        <p style="color: red;">오류 발생: {{ error }}</p>
      </div>

      <table v-else class="data-table">
        <thead>
        <tr>
          <th>문의 번호</th>
          <th>문의 유형</th>
          <th>제목</th>
          <th>내용</th>
          <th>작성자</th>
          <th>연락처</th>
          <th>등록일</th>
        </tr>
        </thead>
        <tbody>
        <tr v-if="filteredQnaList.length === 0">
          <td colspan="7" style="text-align: center;">검색 결과가 없습니다.</td>
        </tr>
        <tr v-for="qna in filteredQnaList" :key="qna.id">
          <td>{{ qna.id }}</td>
          <td>{{ qna.consultType }}</td>
          <td>{{ qna.title }}</td>
          <td>{{ qna.content }}</td>
          <td>{{ qna.name }}</td>
          <td>{{ qna.phone }}</td>
          <td>{{ qna.addTime }}</td>
        </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<style scoped>
/* 검색창 스타일 추가 */
.filter-container {
  margin-bottom: 20px;
}
.search-input {
  width: 100%;
  padding: 10px 15px;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 1em;
}

/* 기존 스타일 */
.content-box { margin-top: 20px; padding: 20px; background-color: #fff; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
.data-table { width: 100%; border-collapse: collapse; margin-top: 20px; }
.data-table th, .data-table td { padding: 12px 15px; border: 1px solid #ddd; text-align: left; }
.data-table th { background-color: #e9ecef; font-weight: 600; }
.data-table tbody tr:hover { background-color: #f8f9fa; }
</style>