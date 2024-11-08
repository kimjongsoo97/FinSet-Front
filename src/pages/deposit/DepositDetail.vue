<template>
  <HeaderNormal navbarTitle="예금 상세" />

  <div class="container">
    <div class="wish-item d-flex align-items-center mt-2">
      <div style="width: 300px;">

        <img :src="deposit.imgUrl" alt="Thumbnail" class="rounded-circle me-3 thumbnail">
        <h5 class="mb-0">{{ deposit.depositName }}</h5> <!-- 예금 이름 -->
        <p class="mb-0 text-muted">{{ deposit.depositBank }}</p> <!-- 은행 이름 -->
      </div>
      <div class="deposit-icon" @click="toggleFavorite(deposit)"> <!-- 클래스명 변경 -->
        <i :class="deposit.favorite ? 'fas fa-heart' : 'far fa-heart'"
           :style="{ color: deposit.favorite ? '#FAB809' : '#888' }"></i>
      </div>

    </div>

    <div class="join-info d-flex mt-2">
      <p class="join-text join-bg">방문없이 가입</p>
      <p class="join-text join-bg">누구나 가입</p>
    </div>

    <div class="details mt-3">
      <div class="rate-info">
        <div>
          <span>최고</span>
          <br />
          <span class="rate-highlight">연 {{ deposit.depositMaxRate }}%</span> <!-- 최고 금리 -->
        </div>

        <div class="separator"></div>

        <div>
          <span>기본</span>
          <br />
          <span class="rate-highlight">연 {{ deposit.depositNormalRate }}%</span> <!-- 기본 금리 -->
          <span class="small-text" style="color: #838687;">(12개월, 세전)</span>
        </div>
      </div>

      <div class="centered-info">
        <h5>상품 안내</h5>
        <hr style="height: 4px;">
      </div>

      <div class="fixed-info">
        <div class="info-item">
          <span class="info-label" style="margin-right: 80px;">기간</span>
          <span>{{ deposit.depositJoin }}</span> <!-- 가입 기간 -->
        </div>
        <div class="info-item">
          <span class="info-label" style="margin-right: 80px;">금액</span>
          <span>{{ Number(deposit.depositAmount).toLocaleString() }}</span>
        </div>
        <div class="info-item">
          <span class="info-label" style="margin-right: 50px;">가입 방법</span>
          <span>{{ deposit.depositWay }}</span> <!-- 가입 방법 -->
        </div>
        <div class="info-item">
          <span class="info-label" style="margin-right: 80px;">대상</span>
          <span>{{ deposit.depositTarget }}</span> <!-- 대상 -->
        </div>
      </div>
    </div>

    <div class="button-container">
      <a :href="deposit.depositLink" target="_blank" rel="noopener noreferrer" class="join-button">지금 바로 가입하기</a> <!-- 가입 링크 -->
    </div>
  </div>
</template>

<script>
import { useRoute } from 'vue-router';
import depositApi from '@/api/depositApi'; // 예금 API 가져오기
import HeaderNormal from '@/components/common/HeaderNormal.vue';
import wishApi from "@/api/wishApi.js";

export default {
  components: {
    HeaderNormal,
  },
  data() {
    return {
      deposit: {}, // 예금 정보 초기화
      imgUrl: '', // 이미지 URL 초기화
      deposits: [], // 예금 목록 데이터 초기화
      wishes: [], // 사용자 관심 목록 데이터 초기화
      uno: null,
    };
  },
  mounted() {
    const route = useRoute();
    const dno = route.query.dno; // 쿼리에서 dno 가져오기
    console.log('dno:', dno); // dno 값을 로그로 출력

    if (dno) {
      depositApi.fetchDepositById(dno).then((data) => {
        this.deposit = data; // 예금 정보 가져오기
        this.imgUrl = data.imgUrl;
        console.log(this.deposit); // 로그로 출력
      });
    }
    this.fetchDeposits();

  },

  methods: {
    async fetchDeposits() {
      try {
        const authData = JSON.parse(localStorage.getItem('auth')); // 로컬 스토리지에서 auth 데이터 가져오기
        this.uno = authData.uno; // uno 값 가져오기

        // 사용자 관심 목록 불러오기 (try-catch로 감싸서 오류 방지)
        try {
          this.wishes = await wishApi.fetchAllWishes(); // 관심 목록 전체 조회
          console.log("Wishes:", this.wishes); // 관심 목록 로그 출력
        } catch (error) {
          console.warn("No wishes found or error fetching wishes:", error); // 관심 목록이 없을 경우 로그 출력
          this.wishes = []; // 관심 목록이 없으면 빈 배열로 처리
        }

        // 관심 목록에서 tno가 1인 상품의 pno를 사용해 deposit.favorite 설정
        const favoritePnos = this.wishes
          .filter(wish => wish.tno === 1)
          .map(wish => wish.pno);

        this.deposit.favorite = favoritePnos.includes(this.deposit.dno);


      } catch (error) {
        console.error("Error fetching deposits:", error); // 오류 처리
      }
    },

    async toggleFavorite(deposit) {
      try {
        deposit.favorite = !deposit.favorite; // favorite 상태 토글

        // API 호출하여 즐겨찾기 추가/삭제 처리
        if (deposit.favorite) {
          await wishApi.addWish({ tno: 1, uno: this.uno, pno: deposit.dno }); // tno와 pno 추가
        } else {
          await wishApi.deleteWish({ tno: 1, uno: this.uno, pno: deposit.dno }); // tno와 pno 삭제
        }
      } catch (error) {
        console.error("Error toggling favorite:", error);
        deposit.favorite = !deposit.favorite; // 원래 상태로 복구
      }
    },
    getImg(imgUrl) {
      return imgUrl ? `src${imgUrl}` : ''; // 절대 URL로 수정
    },
  },

};
</script>


<style scoped>
.container{
  position: relative;
  bottom: 85px;
  margin-top: -30px;
}

.details {
  margin-top: 20px;
  
}

.thumbnail {
  position: absolute;
  left: -11px; /* 왼쪽에 배치 */
  width: 50px;
  height: 50px;
  object-fit: cover;
  transform: none; /* 기존의 translateX 제거 */
  top: 34px;
}

.join-info {
  display: flex; /* Flexbox 사용 */
  margin-top: 10px; /* 상단 여백 추가 */
  position: relative;
    left: -15px;
    bottom: 24px;
}

.join-text {
  color: #9E9E9E; /* 글씨색 지정 */
  padding: 1px 4px; /* 패딩 추가 */
  border-radius: 3px; /* 모서리 둥글게 */
  font-size: 9px; /* 글씨 크기 조정 */
}

/* 각각의 배경색 지정 */
.join-bg {
  background-color: #F7F7F7; /* 배경색 지정 */
  margin-right: 5px; /* 간격 제거 */
}


.rate-info {
  display: flex; /* Flexbox 사용 */
  justify-content: space-between; /* 요소를 양쪽 끝에 배치 */
  width: 100%; /* 전체 너비 사용 */
  margin-bottom: 10px; /* 하단 여백 추가 */
  position: relative;
  right: 5px;
    bottom: 27px;
}


.rate-highlight {
  font-size: 20px; /* 글씨 크기 설정 */
  color: #FFBB00; /* 글씨 색상 설정 */
  font-weight: bold;
}

h7{
  color: #838687;
}

.separator {
  width: 1px; /* 선의 너비 */
  background-color: #838687; /* 선의 색상 */
  height: 40px; /* 선의 높이 */
  margin: 0 15px; /* 양쪽 간격 */
}

.small-text {
  font-size: 12px; /* 작게 설정하고 싶은 크기 */
  margin-left: 10px;
}

.centered-info {
  text-align: center; /* 중앙 정렬 */
  margin-top: 10px; /* 위쪽 여백 추가 */
}


.fixed-info {
  display: flex;
  flex-direction: column;
  margin-top: 20px;
  position: relative;
  top:6px;
}

.info-item {
  display: flex; /* flexbox 사용 */
  justify-content: flex-start; /* 왼쪽 정렬 */
  margin-bottom: 10px; /* 각 항목 사이의 공백 조정 */
}

.info-label {
  margin-right: 10px; /* 레이블과 값 사이의 공백 */
  color: #838687;
}

.button-container {
  display: flex; /* flexbox 사용 */
  justify-content: center; /* 가로 중앙 정렬 */
  margin-top: 50px;
}

.join-button {
  background-color: #FFBB00; /* 버튼 배경색 */
  color: white; /* 글씨 색상 */
  border: none; /* 기본 테두리 제거 */
  border-radius: 5px; /* 모서리 둥글게 */
  padding: 3px 30px; /* 버튼 내부 여백 */
  font-size: 20px; /* 글씨 크기 */
  cursor: pointer; /* 마우스 포인터를 손가락으로 변경 */
  transition: background-color 0.3s; /* 배경색 변화 애니메이션 */
  text-decoration: none;
  position: relative;
  top:150px;
}
.deposit-icon { /* 클래스명 변경 */
  font-size: 24px;
  color: #888; /* 기본 색상 */
  margin-top: 80px;
}

.deposit-icon .fas {
  color: #FAB809; /* 하트 아이콘 노란색 */
}
hr{
  width:100%;
}
.join-button:hover {
  background-color: #e6a600; /* 마우스 오버 시 배경색 변화 */
}
h5, p{
  position: relative;
  right: -23px;
  font-weight: bold;
  padding: 5px;
}
img{
  position: relative;
  left: 50px;
}
.fa-heart:before {
    position: relative;
    bottom: 39px;
    font-size: 27px;
}
.centered-info{
  position: relative;
  right: 20px;
}

hr[data-v-f8110eed] {
    width: 100%;
    position: relative;
    left: 20px;
}
</style>