<template>

  <HeaderSignUp />

  <div class="container">
      <h1> 반갑습니다!</h1>

      <br>
      <br>
      <div class="col-12">
          <label for="id" class="form-label">*이메일</label>
          <div class="input-group">
              <input type="text" class="form-control" id="id" placeholder="hongildong@kb.com" v-model="member.id">
              <button class="btn btn-custom" type="button" @click="checkId">인증하기</button>
          </div>
          <span :class="disableSubmit.value ? 'text-primary' : 'text-danger'" style="margin-top: 10px; display: block;">{{ checkError }}</span>
          <div class="invalid-feedback">
              이메일을 입력해주세요.
          </div>
      </div>

      <br>

      <div class="col-12">
  <label for="password" class="form-label">*비밀번호</label>
  <input
      type="password"
      class="form-control"
      id="password"
      placeholder="비밀번호 입력"
      v-model="member.password"
      required
  >
  <div class="invalid-feedback">
    비밀번호를 입력해주세요.
  </div>
</div>

<br>

<div class="col-12">
  <label for="password2" class="form-label">*비밀번호 확인</label>
  <span v-if="passwordMismatch" class="text-danger ms-2">비밀번호가 일치하지 않습니다.</span>
  <div class="d-flex align-items-center">
    <input
        type="password"
        class="form-control"
        id="password2"
        placeholder="비밀번호 재입력"
        v-model="member.password2"
        required
    >
  </div>
  <div class="invalid-feedback">
    비밀번호를 확인해주세요.
  </div>
</div>

      <br>

      <div class="col-12">
          <label for="name" class="form-label">*닉네임</label>
          <div class="input-group">
              <input type="text" class="form-control" id="name" placeholder="키키핑" v-model="member.name" required>
              <button class="btn btn-custom" type="button" @click="checkName">중복 확인</button>
          </div>
          <span :class="disableSubmitName.value ? 'text-primary' : 'text-danger'">{{ checkErrorName }}</span>
          <div class="invalid-feedback">
              닉네임을 입력해주세요.
          </div>
      </div>

      <br>
      <br>

      <button class="btn btn-signup" type="button" @click="join">회원가입</button>

  </div>
</template>

<script setup>
import { reactive, ref, computed } from 'vue';
import HeaderSignUp from '@/components/common/HeaderSignUp.vue';
import loginApi from "@/api/loginApi";
import { useRouter } from 'vue-router';

const router = useRouter();
const checkError = ref('');
const checkErrorName = ref('');
const passwordMismatch = ref(false);

const member = reactive({
  id: '',
  password: '',
  password2: '',
  name: '',
  kakaoId:'',
});

const disableSubmit = ref(true);
const disableSubmitName = ref(true);
const isEmailValid = computed(() => {
  return member.id.includes('@');
});

const checkId = async () => {
  if (!member.id) {
      return alert('사용자 ID를 입력하세요');
  }
  if (!isEmailValid.value) {
      return alert('유효한 이메일을 입력해주세요');
  }
  disableSubmit.value = await loginApi.checkId(member.id);
  checkError.value = disableSubmit.value ? "이미 사용중인 아이디 입니다" : "사용가능한 아이디 입니다.";
}

const join = async () => {
  if (member.password !== member.password2) {
      return alert('비밀번호가 일치하지 않습니다');
  }
  try {
      await loginApi.create(member);
      // 회원가입 성공 후 완료 페이지로 이동
      router.push({ name: 'SignUpComplete' });  // 완료 페이지로 이동
  } catch (e) {
      console.error(e);
      alert('회원가입 중 오류가 발생했습니다. 다시 시도해주세요.');
  }
};

const checkName = async () => {
  if (!member.name) {
      return alert('사용자 닉네임을 입력해주셔야됩니다.');
  } else {
      disableSubmitName.value = await loginApi.checkName(member.name);
      checkErrorName.value = disableSubmitName.value ? "이미 사용중인 닉네임 입니다" : "사용가능한 닉네임 입니다.";
  }
};
</script>

<style scoped>
.container {
  margin-top: -70px;
}

.btn-custom {
  background-color: #FFBF0A; /* 버튼 배경색 */
  color: white; /* 버튼 글씨색 */
  border: none; /* 테두리 없음 */
  border-radius: 5px; /* 모서리 둥글게 */
}

.btn-signup {
  background-color: #FFBF0A; /* 버튼 배경색 */
  color: white; /* 버튼 글씨색 */
  border: none; /* 테두리 없음 */
  border-radius: 30px; /* 모서리 둥글게 */
  width: 100%; /* 버튼 너비를 100%로 설정 */
  padding: 10px; /* 패딩 추가 */
  margin-top: 20px; /* 위쪽 여백 추가 */
}

.input-group {
  display: flex; /* 입력 필드와 버튼을 수평으로 정렬 */
}

.input-group .form-control {
  border-right: none; /* 입력 필드와 버튼 사이의 테두리 제거 */
}

.input-group .btn {
  border-left: none; /* 입력 필드와 버튼 사이의 테두리 제거 */
}

.text-danger {
  color: red; /* 오류 메시지 색상 */
}

.form-label {
  color: gray; /* 라벨 색상을 회색으로 변경 */
}
</style>
