<script setup>
import { ref, onMounted, computed } from 'vue';

const userList = ref([]);
const isLoading = ref(true);
const error = ref(null);
const searchQuery = ref('');

// --- 추가된 상태 변수 ---
// 현재 수정 중인 사용자의 ID를 저장합니다. null이면 아무도 선택되지 않은 상태입니다.
const selectedUserId = ref(null);
// 수정 폼에 바인딩될 사용자 데이터의 '복사본'입니다.
const editingUser = ref(null);

// --- computed 속성: 이름(userName)과 연락처(phoneNumber)로 필터링 ---
const filteredUserList = computed(() => {
  if (!userList.value) return [];
  if (!searchQuery.value) return userList.value;

  const lowerCaseQuery = searchQuery.value.toLowerCase();

  return userList.value.filter(user => {
    const nameMatch = user.userName && user.userName.toLowerCase().includes(lowerCaseQuery);
    const phoneMatch = user.phoneNumber && user.phoneNumber.includes(lowerCaseQuery);
    return nameMatch || phoneMatch;
  });
});

// --- 서버에서 사용자 목록을 가져오는 함수 ---
async function fetchUserData() {
  try {
    const response = await fetch('http://localhost:8080/api/admin/users/all');
    if (!response.ok) {
      throw new Error('사용자 데이터를 불러오는데 실패했습니다.');
    }
    const data = await response.json();
    userList.value = data;
  } catch (e) {
    error.value = e.message;
    console.error("사용자 데이터 로딩 중 에러 발생:", e);
  } finally {
    isLoading.value = false;
  }
}

// --- 행 클릭 시 수정 폼을 토글하는 함수 ---
function selectUserForEdit(user) {
  // 이미 선택된 사용자를 다시 클릭하면 폼을 닫습니다.
  if (selectedUserId.value === user.userId) {
    selectedUserId.value = null;
    editingUser.value = null;
  } else {
    // 새로운 사용자를 선택하면 해당 사용자 ID를 저장하고,
    // 원본 데이터가 바로 변경되지 않도록 객체를 복사해서 수정용 데이터로 사용합니다.
    selectedUserId.value = user.userId;
    editingUser.value = { ...user };
  }
}

// --- 수정 내용을 저장하는 함수 ---
async function handleUpdate(user) {
  if (!editingUser.value) return;

  // API 호출을 위한 try-catch 블록
  try {
    // 1. 서버로 보낼 데이터 준비 (password 같은 불필요한 정보는 제외할 수 있습니다)
    const updatedUserData = {
      userName: editingUser.value.userName,
      userPoint: editingUser.value.userPoint,
      phoneNumber: editingUser.value.phoneNumber,
      pinNo: editingUser.value.pinNo,
      userAddress: editingUser.value.userAddress,
      userId: editingUser.value.userId
    };
    console.log(`${user.userId}`)
    // 2. fetch를 사용하여 서버에 PUT 요청 보내기
    // URL의 마지막 부분에 있는 id는 사용자의 고유 식별자여야 합니다.
    const response = await fetch(`http://localhost:8080/api/admin/users/${user.userId}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(updatedUserData),
    });

    // 3. 서버 응답 확인
    if (!response.ok) {
      // 서버가 4xx 또는 5xx 상태 코드를 반환하면 에러를 발생시킴
      throw new Error('서버로부터 응답을 받는데 실패했습니다. 상태 코드: ' + response.status);
    }

    // 4. API 호출 성공 시 로직
    console.log("성공적으로 수정됨:", updatedUserData);
    alert(`${editingUser.value.userName}의 정보가 성공적으로 수정되었습니다.`);

    // 4-1. 화면의 userList에서 원본 데이터를 업데이트합니다.
    const index = userList.value.findIndex(u => u.userId === editingUser.value.userId);
    if (index !== -1) {
      userList.value[index] = { ...editingUser.value }; // 복사본을 할당하여 반응성을 유지
    }

    // 4-2. 선택 상태를 초기화하여 수정 폼을 닫습니다.
    selectedUserId.value = null;
    editingUser.value = null;

  } catch (error) {
    // 5. API 호출 실패 시 에러 처리
    console.error("사용자 정보 수정 중 에러 발생:", error);
    alert('정보 수정에 실패했습니다. 문제가 지속되면 관리자에게 문의하세요.');
  }
}

// --- 수정을 취소하는 함수 ---
function cancelEdit() {
  // 선택 상태를 초기화하여 수정 폼을 닫습니다.
  selectedUserId.value = null;
  editingUser.value = null;
}

onMounted(() => {
  fetchUserData();
});
</script>

<template>
  <div>
    <h1>👥 사용자 관리</h1>
    <div class="content-box">
      <div class="filter-container">
        <input
            type="text"
            v-model="searchQuery"
            placeholder="사용자 이름 또는 연락처로 검색하세요"
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
          <th>아이디</th>
          <th>이름</th>
          <th>연락처</th>
          <th>주소</th>
          <th>포인트</th>
          <th>핀번호</th>
        </tr>
        </thead>
        <template v-for="user in filteredUserList" :key="user.userId">
          <tr @click="selectUserForEdit(user)" class="user-row">
            <td>{{ user.userId }}</td>
            <td>{{ user.userName }}</td>
            <td>{{ user.phoneNumber }}</td>
            <td>{{ user.userAddress }}</td>
            <td>{{ user.userPoint }}</td>
            <td>{{ user.pinNo}}</td>
          </tr>

          <transition name="slide-fade">
            <tr v-if="selectedUserId === user.userId" class="edit-row">
              <td colspan="6">
                <div class="edit-form-container">
                  <h3>✍️ {{ user.userName }}님 정보 수정</h3>
                  <div class="form-grid">
                    <div class="form-group">
                      <label>이름</label>
                      <input type="text" v-model="editingUser.userName" />
                    </div>
                    <div class="form-group">
                      <label>연락처</label>
                      <input type="text" v-model="editingUser.phoneNumber" />
                    </div>
                    <div class="form-group full-width">
                      <label>주소</label>
                      <input type="text" v-model="editingUser.userAddress" />
                    </div>
                    <div class="form-group">
                      <label>포인트</label>
                      <input type="number" v-model="editingUser.userPoint" />
                    </div>
                    <div class="form-group">
                    <label>핀번호</label>
                    <input type="text" v-model="editingUser.pinNo" />
                  </div>
                  </div>
                  <div class="button-group">
                    <button @click="handleUpdate(user)" class="btn-save">저장</button>
                    <button @click="cancelEdit" class="btn-cancel">취소</button>
                  </div>
                </div>
              </td>
            </tr>
          </transition>
        </template>

        <tr v-if="filteredUserList.length === 0">
          <td colspan="5" style="text-align: center;">검색 결과가 없습니다.</td>
        </tr>
      </table>
    </div>
  </div>
</template>

<style scoped>
/* 기존 스타일 */
.filter-container { margin-bottom: 20px; }
.search-input { width: 100%; padding: 10px 15px; border: 1px solid #ddd; border-radius: 5px; font-size: 1em; }
.content-box { margin-top: 20px; padding: 20px; background-color: #fff; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
.data-table { width: 100%; border-collapse: collapse; margin-top: 20px; }
.data-table th, .data-table td { padding: 12px 15px; border: 1px solid #ddd; text-align: left; vertical-align: middle; }
.data-table th { background-color: #e9ecef; font-weight: 600; }

/* --- 추가된 스타일 --- */

/* 클릭 가능한 행에 마우스 커서 변경 및 호버 효과 */
.user-row:hover {
  background-color: #f8f9fa;
  cursor: pointer;
}

/* 수정 폼 행의 기본 스타일 제거 */
.edit-row, .edit-row:hover {
  background-color: #fdfdfd; /* 살짝 다른 배경색으로 구분 */
}
.edit-row td {
  padding: 0;
  border: 1px solid #ddd;
  border-top: 2px solid #007bff; /* 상단에 구분선 강조 */
}

/* 수정 폼 컨테이너 스타일 */
.edit-form-container {
  padding: 20px;
  background-color: #fdfdfd;
}
.edit-form-container h3 {
  margin-top: 0;
  margin-bottom: 20px;
  font-weight: 600;
}

/* 수정 폼 레이아웃 */
.form-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
  margin-bottom: 20px;
}
.form-group {
  display: flex;
  flex-direction: column;
}
.form-group.full-width {
  grid-column: 1 / -1; /* 한 줄 전체를 차지 */
}
.form-group label {
  margin-bottom: 5px;
  font-weight: 500;
  font-size: 0.9em;
}
.form-group input {
  padding: 8px 12px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

/* 버튼 그룹 스타일 */
.button-group {
  text-align: right;
  margin-top: 10px;
}
.button-group button {
  padding: 8px 16px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-weight: 500;
  margin-left: 10px;
  transition: background-color 0.2s;
}
.btn-save {
  background-color: #007bff;
  color: white;
}
.btn-save:hover {
  background-color: #0056b3;
}
.btn-cancel {
  background-color: #6c757d;
  color: white;
}
.btn-cancel:hover {
  background-color: #5a6268;
}

/* 슬라이드 다운/업 애니메이션 */
.slide-fade-enter-active {
  transition: all 0.3s ease-out;
}
.slide-fade-leave-active {
  transition: all 0.3s cubic-bezier(1, 0.5, 0.8, 1);
}
.slide-fade-enter-from,
.slide-fade-leave-to {
  transform: translateY(-10px);
  opacity: 0;
}
</style>