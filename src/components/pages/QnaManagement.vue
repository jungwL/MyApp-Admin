<script setup>
import { ref, onMounted, computed } from 'vue';

// --- 데이터 저장을 위한 반응형 변수들 ---
const qnaList = ref([]);
const isLoading = ref(true);
const error = ref(null);
const searchQuery = ref('');

// --- 답글 기능을 위한 상태 변수들 ---
const isModalVisible = ref(false);
const selectedQna = ref(null);
const replyContent = ref('');
const isSubmitting = ref(false);

// --- 검색 필터 기능 ---
const filteredQnaList = computed(() => {
  if (!qnaList.value) return [];
  if (!searchQuery.value) return qnaList.value;
  const lowerCaseQuery = searchQuery.value.toLowerCase();
  return qnaList.value.filter(qna => {
    const nameMatch = qna.name && qna.name.toLowerCase().includes(lowerCaseQuery);
    const phoneMatch = qna.phone && qna.phone.includes(lowerCaseQuery);
    return nameMatch || phoneMatch;
  });
});

// --- 서버에서 QnA 목록을 가져오는 함수 ---
async function fetchQnaData() {
  isLoading.value = true;
  error.value = null;
  try {
    const response = await fetch('http://localhost:8080/api/admin/qna/all');
    if (!response.ok) throw new Error('데이터를 불러오는데 실패했습니다.');
    qnaList.value = await response.json();
  } catch (e) {
    error.value = e.message;
  } finally {
    isLoading.value = false;
  }
}

// --- 답글 모달을 여는 함수 ---
function openReplyModal(qna) {
  selectedQna.value = qna;
  // 🌟 새 답글을 작성하는 것이므로 입력창은 항상 비워둡니다.
  replyContent.value = '';
  isModalVisible.value = true;
}

// --- 답글을 서버로 전송하는 함수 ---
async function submitReply() {
  if (!replyContent.value.trim() || !selectedQna.value) {
    alert('답글 내용을 입력해주세요.');
    return;
  }
  isSubmitting.value = true;
  try {
    const response = await fetch(`http://localhost:8080/api/admin/qna/${selectedQna.value.id}/reply`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ reply: replyContent.value }),
    });
    if (!response.ok) throw new Error('답글 등록에 실패했습니다.');
    await fetchQnaData();
    isModalVisible.value = false;
    alert('답글이 성공적으로 등록되었습니다.');
  } catch (e) {
    alert(e.message);
  } finally {
    isSubmitting.value = false;
  }
}

onMounted(fetchQnaData);
</script>

<template>
  <div>
    <h1>📝 QnA 관리</h1>
    <div class="content-box">
      <div class="filter-container">
        <input type="text" v-model="searchQuery" placeholder="이름 또는 연락처로 검색하세요" class="search-input" />
      </div>

      <div v-if="isLoading"><p>데이터를 불러오는 중입니다...</p></div>
      <div v-else-if="error"><p style="color: red;">오류 발생: {{ error }}</p></div>

      <table v-else class="data-table">
        <thead>
        <tr>
          <th>문의 번호</th>
          <th>문의 유형</th>
          <th>제목</th>
          <th>작성자</th>
          <th>연락처</th>
          <th>등록일</th>
          <th>답변 상태</th>
        </tr>
        </thead>
        <tbody>
        <tr v-if="filteredQnaList.length === 0">
          <td colspan="7" style="text-align: center;">검색 결과가 없습니다.</td>
        </tr>
        <tr v-for="qna in filteredQnaList" :key="qna.id" @click="openReplyModal(qna)" class="clickable-row">
          <td>{{ qna.id }}</td>
          <td>{{ qna.consultType }}</td>
          <td>{{ qna.title }}</td>
          <td>{{ qna.name }}</td>
          <td>{{ qna.phone }}</td>
          <td>{{ new Date(qna.addTime).toLocaleString() }}</td>
          <td>
            <!-- 🌟 replies 배열의 길이로 답변 유무를 확인합니다. -->
            <span :class="qna.replies && qna.replies.length > 0 ? 'status-replied' : 'status-pending'">
                {{ qna.replies && qna.replies.length > 0 ? '답변 완료' : '대기중' }}
              </span>
          </td>
        </tr>
        </tbody>
      </table>
    </div>

    <!-- 답글 작성을 위한 모달 창 -->
    <div v-if="isModalVisible" class="modal-overlay" @click.self="isModalVisible = false">
      <div class="modal-content">
        <h2>문의 상세 및 답글 달기</h2>
        <div v-if="selectedQna" class="qna-summary">
          <p><strong>작성자:</strong> {{ selectedQna.name }} ({{ selectedQna.phone }})</p>
          <p><strong>제목:</strong> {{ selectedQna.title }}</p>
          <p><strong>문의 내용:</strong></p>
          <p class="qna-content-box">{{ selectedQna.content }}</p>
        </div>

        <!-- 🌟 기존 답변 목록을 보여주는 부분 추가 -->
        <div v-if="selectedQna && selectedQna.replies && selectedQna.replies.length > 0" class="replies-container">
          <h3>등록된 답변</h3>
          <div v-for="reply in selectedQna.replies" :key="reply.id" class="reply-item">
            <p class="reply-content">{{ reply.content }}</p>
            <p class="reply-meta">_ {{ new Date(reply.createdAt).toLocaleString() }}</p>
          </div>
        </div>

        <hr class="divider" />

        <textarea v-model="replyContent" placeholder="새로운 답변을 입력하세요..." class="reply-textarea" rows="5"></textarea>
        <div class="modal-actions">
          <button @click="isModalVisible = false" class="btn btn-secondary">취소</button>
          <button @click="submitReply" :disabled="isSubmitting" class="btn btn-primary">
            {{ isSubmitting ? '등록 중...' : '답글 등록' }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* (기존 스타일은 그대로 유지) */
.filter-container { margin-bottom: 20px; }
.search-input { width: 100%; padding: 10px 15px; border: 1px solid #ddd; border-radius: 5px; font-size: 1em; }
.content-box { margin-top: 20px; padding: 20px; background-color: #fff; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
.data-table { width: 100%; border-collapse: collapse; margin-top: 20px; }
.data-table th, .data-table td { padding: 12px 15px; border: 1px solid #ddd; text-align: left; }
.data-table th { background-color: #e9ecef; font-weight: 600; }
.clickable-row { cursor: pointer; transition: background-color 0.2s ease; }
.clickable-row:hover { background-color: #f8f9fa; }
.status-replied { color: #28a745; font-weight: bold; }
.status-pending { color: #ffc107; font-weight: bold; }
.modal-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background-color: rgba(0, 0, 0, 0.6); display: flex; justify-content: center; align-items: center; z-index: 1000; }
.modal-content { background-color: white; padding: 25px; border-radius: 10px; width: 90%; max-width: 600px; box-shadow: 0 5px 15px rgba(0,0,0,0.3); animation: slide-down 0.3s ease-out; }
.qna-summary p { margin: 5px 0; }
.qna-content-box { background-color: #f8f9fa; border: 1px solid #eee; padding: 10px; border-radius: 5px; margin-top: 5px; max-height: 120px; overflow-y: auto; white-space: pre-wrap; }
.reply-textarea { width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 5px; font-size: 1em; resize: vertical; }
.modal-actions { display: flex; justify-content: flex-end; gap: 10px; margin-top: 20px; }
.btn { padding: 10px 20px; border: none; border-radius: 5px; cursor: pointer; font-weight: bold; transition: background-color 0.2s; }
.btn-primary { background-color: #007bff; color: white; }
.btn-primary:hover { background-color: #0056b3; }
.btn-secondary { background-color: #6c757d; color: white; }
.btn-secondary:hover { background-color: #5a6268; }
.btn:disabled { background-color: #ccc; cursor: not-allowed; }

/* 🌟 답변 목록 스타일 추가 */
.replies-container { margin-top: 20px; }
.replies-container h3 { margin-bottom: 10px; font-size: 1.1em; }
.reply-item {
  background-color: #e9ecef;
  padding: 12px;
  border-radius: 8px;
  margin-bottom: 10px;
}
.reply-content {
  white-space: pre-wrap; /* 줄바꿈 유지 */
}
.reply-meta {
  text-align: right;
  font-size: 0.8em;
  color: #6c757d;
  margin-top: 8px;
}
.divider {
  margin: 20px 0;
  border: none;
  border-top: 1px solid #eee;
}

@keyframes slide-down { from { transform: translateY(-30px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }
</style>

