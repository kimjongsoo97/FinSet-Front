<template>
  <HeaderNormal navbarTitle="단어장" />

  <div class="container">
    <!-- <div class="card-container"> -->
      <draggable
        v-if="items.length > 0" 
        group="items"
        @end="onEnd"
        class="card-container"
      >
  
      <div v-for="(item, index) in items" :key="item.dwno" class="card">
        <div class="card-header">
          <i 
            class="fa-solid fa-star icon" 
            :class="{ active: isStarActive(index) }" 
            @click="toggleStar(index)"
          ></i>
          <span class="title" @click="toggleDescription(index)">{{ item.word }}</span>
          <i class="fa-solid fa-caret-down arrow" :class="{ active: isActive(index) }" @click="toggleDescription(index)"></i>
        </div>
        <div v-if="isActive(index)">
          <p class="description">{{ item.content }}</p>
          <div class="memo">
            <div class="memo-header">
              <p class="memo-text">Memo</p>
              <div class="memo-buttons">
                <button v-if="!item.isEditing" @click="editItem(index)" class="edit-btn">
                  <i class="fa-solid fa-gear"></i> 수정
                </button>
                <button v-if="item.isEditing" @click="saveItem(index)" class="save-btn">
                  <i class="fa-solid fa-check"></i> 저장
                </button>
              </div>
            </div>
            <textarea v-model="item.memo" :readonly="!item.isEditing" />
          </div>
        </div>
      </div>
    <!-- </div> -->
    </draggable>
    

    <div v-if="showEditCompleteDialog" class="confirm-overlay">
      <div class="confirm-dialog">
        <span>수정이 완료되었습니다.</span>
        <div class="dialog-button-group">
          <button @click="closeEditCompleteDialog" class="confirm-button">확인</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import HeaderNormal from '@/components/common/HeaderNormal.vue';
import dictWishApi from '@/api/dictWishApi'; // API 호출 파일
import dictApi from '@/api/dictApi'; // 추가된 API 호출 파일
import { ref } from 'vue';
import { VueDraggableNext } from 'vue-draggable-next'


export default {
  components: {
    HeaderNormal,
    draggable: VueDraggableNext,
  },
  data() {
    return {
      items: [], // 아코디언 데이터
      activeIndices: [], // active 상태를 저장하는 배열
      starStates: [], // 각 리스트 항목별로 false로 초기화
      showEditCompleteDialog: false, // 수정 완료 다이얼로그 표시 여부
      uno: null, // uno 값을 초기화합니다.
      listStarStates: [], // 별 상태를 저장할 배열
      wishItems: [], // 위시 아이템 목록
    };
  },
  mounted() {
    const authData = JSON.parse(localStorage.getItem('auth'));
    this.uno = authData.uno; // uno 값 가져오기
    console.log('UNO:', this.uno); // uno 값 확인
    this.fetchItems() 
  },

  methods: {
    async fetchItems() {
      try {
        const response = await dictWishApi.getAll(this.uno); // uno를 API 호출에 전달합니다.
        if (response && Array.isArray(response)) {
          this.items = response; // API를 통해 단어장 항목을 가져옵니다.
          this.starStates = Array(this.items.length).fill(true); // 별 상태 초기화
          console.log('Fetched items:', this.items);
          
          // 각 item의 dino에 대해 추가 정보를 가져옵니다.
          for (let item of this.items) {
            const data = await dictApi.getByDino(item.dino); // dino로 사전 항목 조회
            item.word = data.word;
            item.content = data.content;
          }
        } else {
          console.error('API 응답 형식이 올바르지 않습니다.', response);
        }
      } catch (error) {
        console.error('Error fetching items:', error.message || error); // 오류 메시지 로그
        if (error.response) {
          console.error('Response data:', error.response.data);
          console.error('Response status:', error.response.status);
        } else if (error.request) {
          console.error('No response received:', error.request);
        } else {
          console.error('Error setting up request:', error.message);
        }
      }
    },

    isActive(index) {
      return this.activeIndices.includes(index); // 특정 인덱스가 active 상태인지 확인
    },
    toggleDescription(index) {
      if (this.isActive(index)) {
        this.activeIndices = this.activeIndices.filter(i => i !== index); // active 상태 해제
      } else {
        this.activeIndices.push(index); // active 상태 추가
      }
    },
    async toggleStar(index) {
      const currentState = this.starStates[index];
      this.starStates[index] = !currentState; // 해당 인덱스의 별 상태 토글

      console.log(this.items[index]);

      const dino = this.items[index].dino; // 현재 아이템의 dino 값 가져오기
      console.log('dino:', dino);

      // 새로운 상태에 따라 dict 객체 생성
      const dict = { ...this.wishItems[index], status: this.starStates[index] ? 0 : 1 };
      console.log('dict:', dict);

      try {
        // API 호출하여 상태 업데이트
        await dictApi.updateStatus(dino, dict);
        console.log(`${this.starStates[index] ? 'Activated' : 'Deactivated'} star for item with dino: ${dino}`);
      } catch (error) {
        console.error('Error updating wish item:', error);
      }
    },

    isStarActive(index) {
      return this.starStates[index]; // 해당 인덱스의 별이 활성화된 상태인지 확인
    },
    editItem(index) {
      const item = this.items[index];
      item.isEditing = true; // 수정 모드 활성화
    },
    async saveItem(index) {
      const item = this.items[index];
      item.isEditing = false; // 수정 모드 비활성화
      
      try {
        await dictWishApi.updateMemo(item.dwno, { memo: item.memo }); // API 호출로 메모 업데이트
        this.showEditCompleteDialog = true; // 수정 완료 다이얼로그 표시
      } catch (error) {
        console.error('Error updating memo:', error); // 오류 로그
      }
    },
    closeEditCompleteDialog() {
      this.showEditCompleteDialog = false; // 수정 완료 다이얼로그 닫기
    },
    
    onEnd(event) {
      if (event && event.newIndex !== event.oldIndex) {
        // 드래그된 아이템의 새 인덱스와 기존 인덱스 확인
        const movedItem = this.items[event.oldIndex]; // 드래그된 아이템
        this.items.splice(event.oldIndex, 1); // 기존 인덱스에서 아이템 제거
        this.items.splice(event.newIndex, 0, movedItem); // 새 인덱스에 아이템 추가

        // dwno 값을 업데이트
        this.updateDwnoValues();
      } else {
        console.error('드래그 이벤트에 문제가 있습니다.');
      }
    },

    async updateDwnoValues() {
      try {
        const updatedItems = this.items.map((item, index) => ({
          dwno: item.dwno, // 기존 dwno 유지
          newDwno: index + 1 // 새 dwno 값 설정 (1부터 시작)
        }));

        console.log('Updated Items:', updatedItems);

        // API 호출로 dwno 값 업데이트
        await dictWishApi.updateOrder({ dictWishOrders: updatedItems }); // 이 부분은 API 엔드포인트에 따라 조정 필요
        console.log('dwno values updated successfully');
      } catch (error) {
        console.error('Error updating dwno values:', error);
      }
    },


  },
};
</script>


<style scoped>
.container {
  margin-bottom: 150px;
  margin-top: -90px;
}

.card-container {
  display: flex;
  flex-direction: column;
  gap: 10px; /* 카드 간 간격 조정 */
  color: #6e6053; /* 기본 글씨 색상 */
}

.card {
  width: 350px;
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 15px;
  background-color: #6e6053; /* 카드 배경색을 #6E6053로 설정 */
  color: white; /* 카드 내부 텍스트 색상을 흰색으로 설정 */
}

.card-header {
  display: flex;
  align-items: center;
  cursor: pointer; /* 클릭 가능하도록 커서 변경 */
}

.title {
  font-size: 1.2em; /* 제목 크기 조정 */
  margin-left: 10px;
}
.memo {
  margin-top: 10px;
  background-color: white;
  color: black;
  padding: 10px;
  border-radius: 4px;
  position: relative;
}

.memo-header {
  display: flex;
  justify-content: space-between; /* Memo 텍스트와 버튼을 양 끝에 배치 */
  align-items: center; /* 수직으로 중앙 정렬 */
  margin-bottom: 10px; /* 텍스트 영역과 textarea 사이 간격 추가 */
}

.memo-text {
  font-size: 1rem; /* Memo 텍스트 크기 조정 */
}

.memo-buttons {
  display: flex;
}

textarea {
  width: 100%;
  height: 50px;
  border: none;
  background-color: #f0f0f0;
  border-radius: 4px;
  color: black;
}


.icon {
  color: gray; /* 기본 별 색상 - 회색 */
  margin-left: -10px;
}

.icon.active {
  color: #ffbf0a; /* 활성화된 별 색상 - 노란색 */
}

.arrow {
  margin-left: 10px;
  position: absolute;
  transform: translateX(3100%);
}

.save-btn,
.edit-btn{
padding: 4px 8px; /* 버튼 크기 작게 */
border: none;
border-radius: 4px;
cursor: pointer;
font-size: 0.85rem; /* 글씨 크기 작게 */
}

.save-btn{
  background-color: #d9edf7; /* 저장 버튼 배경색 */
  color: #31708f; /* 저장 버튼 글씨색 */
  margin-right: 5px;
}

.edit-btn {
background-color: #edf1f8; /* 수정 버튼 배경색 */
color: #547bc1; /* 수정 버튼 글씨색 */
}


/* 다이얼로그 스타일 */
.confirm-overlay {
  position: fixed; /* 화면에 고정 */
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.6); /* 투명한 검정 배경 */
  z-index: 10; /* 다른 요소들 위에 표시 */
  }
  
  .confirm-dialog {
  position: fixed; /* 화면에 고정 */
  top: 45%; /* 중앙보다 약간 위로 배치 */
  left: 50%; /* 중앙에 배치 */
  transform: translate(-50%, -50%); /* 중앙 정렬 */
  border: 1px solid #60584C;
  background-color: white;
  border-radius: 10px;
  padding: 20px; /* 패딩 추가 */
  text-align: center;
  z-index: 11; /* 오버레이 위에 표시 */
  }
  
  .dialog-button-group {
  display: flex; /* Flexbox를 사용하여 버튼들을 나란히 배치 */
  justify-content: space-between; /* 버튼들 간의 간격을 균등하게 배치 */
  padding-left: 20px;
  padding-right: 20px;
  }

  .confirm-button {
    background-color: #816843; /* 배경을 흰색으로 설정 */
    border: 1px solid #60584C; /* 테두리 색을 #60584C로 설정 */
    color: #ffffff; /* 글씨 색을 #60584C로 설정 */
    font-size: 16px;
    text-align: center;
    padding: 5px 10px; /* 좌우 패딩 추가 */
    border-radius: 5px; /* 모서리를 둥글게 */
    cursor: pointer;
    margin-left: 20px;
    margin-top: 10px;
  }
</style>
