<template>
  <div>
    <div class="header">
      <div class="intro">
        <i class="fa fa-university" aria-hidden="true"></i>
        이러닝(e-Learning) 자료조회 서비스
      </div>
      <div class="search">
        <span class="search-title">죽전,천안 교수통합검색</span>
        <span>
          <input
            type="text"
            @keyup.enter="courseSearch"
            v-model="search_word"
            placeholder="교수님을 검색하세요!"
            class="search-box"
          />
          <button class="search-button" @click="courseSearch">
            <i class="fa fa-search" aria-hidden="true"></i>
          </button>
        </span>
        <br />
        <span class="warning">{{this.warningMsg}}</span>
      </div>
      <div class="prof" v-show="isActive && !isEmpty">
        <i class="fa fa-graduation-cap" aria-hidden="true"></i>
        {{this.prof_name + "교수님의 강의자료"}}
      </div>
    </div>
    <div class="body" v-show="isActive && !isEmpty">
      <div class="body-left">
        <span class="title">수강과목 자료조회</span>
        <div class="course-list">
          <li
            v-for="(course, idx) in course.course_name"
            :key="idx"
            @click="detailSearch(idx)"
          >{{ course }}</li>
        </div>
      </div>
      <div>
        <span class="body-right">
          <span class="title">담당학과 및 년도별 이러닝 자료조회</span>
          <div class="content">
            <div class="subtitle" v-show="isActiveDetail">담당학과</div>
            <li v-for="(majors, id) in major" :key="id+10">{{majors}}</li>
            <div class="subtitle" v-show="isActiveDetail">년도별 자료조회(클릭시 조회할 수 있습니다!)</div>
            <li
              v-for="(course, idx) in course_info"
              :key="idx"
              @click="routerToDKU(idx)"
              class="semester"
            >{{course.year + "년 " + course.semester + "학기 " + course.class_no + "분반"}}</li>
            <p v-show="!isActiveDetail">좌측 수강과목을 클릭해주세요!</p>
          </div>
        </span>
      </div>
    </div>
    <div v-show="isEmpty">
      <p class="empty">{{this.prof_name}}의 검색결과가 없습니다!</p>
    </div>
    <div v-show="!isActive" class="pending-page">
      <div class="title">서비스 소개</div>
      <div class="subtitle">이러닝 자료조회 서비스란?</div>
      <p class="pending-explain">
        우리들은 수강신청을 할때, 이 과목이 정확히 어떤 내용을 배우는지 알지 못하고 신청하는 경우가 많습니다. 또한, 과제에 대한 어려움도 있습니다. 이 어려움과 호기심을
        해소하는 방법은 '이러닝'입니다. 우리들은 '이러닝'이라는 사이트를 과제, 수업자료를 받기위해 사용합니다.
        그렇다면, 1년전 혹은 2년전 같은 강의의 이러닝을 들어가볼 수 있다면 어떨까요?
        <br />바로 여기! 교수님의 성함만으로도
        흔적을 찾아 최대 3년전까지의 이러닝자료실을 찾아볼 수 있습니다.
      </p>
      <p></p>
      <br />
      <br />
      <div class="title">서비스 이용방법</div>
      <div class="subtitle">1. 단쿠키 회원가입</div>
      <p
        class="pending-explain"
      >해당 서비스는 단쿠키에서 제공하는 강의데이터를 기반으로 제작된 서비스입니다. 간단한 회원가입후 로그인하셔서 이용해주시면 감사하겠습니다!</p>
      <div class="subtitle">2. 단국대학교 이러닝로그인</div>
      <p class="pending-explain">
        <em>해당 서비스는 '단국대학교 이러닝'에 로그인이 되어있는 상태셔야 정상적 사용이 가능합니다.</em> 아래의 버튼을 누르시면 '단국대학교 이러닝 로그인'창을 띄워드립니다!
        <strong>로그인 후 창을 종료하지마세요!</strong>
      </p>
      <p></p>
      <div class="subtitle">3. 교수님 검색 및 자료조회</div>
      <p class="pending-explain">
        단국대학교 이러닝 로그인을 완료하셨으면, 상단의 검색바에 찾고자하는 교수님을 검색하세요!
        그리고 원하시는 과목의 이러닝자료실, 공지사항의 정보를 확인하세요! 감사합니다:)
      </p>
      <button @click="routerToDKUeleaning">단국대학교 이러닝 로그인</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      course: {
        course_name: [],
        course_code: [],
      },
      course_info: [],
      major: [],
      prof_name: '',
      search_word: '',
      current_code: '',
      isActive: false,
      isActiveDetail: false,
      isEmpty: false,
      warningMsg: '',
    };
  },
  methods: {
    courseSearch() {
      if (this.search_word === '') return;
      this.warningMsg = '';
      this.isActive = true;
      this.isActiveDetail = false;
      this.major = [];
      this.prof_name = this.search_word;
      this.axios
        .get('/elearning', { params: this.search_word })
        .then(res => {
          this.course.course_name = [];
          this.course.course_code = [];
          this.course_info = [];
          res.data.list.map((v, i) => {
            if (v.class.length) {
              this.course.course_name.push(v.name);
              this.course.course_code.push(v.code);
            }
          });
          if (!this.course.course_name.length) {
            // 조회결과 없는경우
            this.isEmpty = true;
          } else {
            this.isEmpty = false;
          }
        })
        .catch(err => {
          if (err.response.status === 401) {
            this.isActive = false;
            this.warningMsg = '단쿠키로그인이 필요한 서비스입니다!';
            this.isActiveDetail = true;
          }
        });
      this.search_word = '';
    },
    detailSearch(idx) {
      this.isActiveDetail = true;
      this.current_code = this.course.course_code[idx];
      this.axios
        .get('/elearning/courseinfo', {
          params: [this.course.course_name[idx], this.prof_name],
        })
        .then(res => {
          this.course_info = [];
          this.major = [];
          res.data[0].major.map((v, i) => {
            this.major.push(v);
          });
          res.data[0].class.map((v, i) => {
            this.course_info.push({
              class_no: v.class_no,
              year: v.opened_at.year,
              semester: v.opened_at.semester,
            });
          });
        });
    },
    routerToDKU(idx) {
      let year = this.course_info[idx].year;
      let semester = this.course_info[idx].semester;
      let code = this.current_code;
      let class_no = this.course_info[idx].class_no;

      let qs = (year + semester + code + class_no).toString();

      let url =
        'http://lms.dankook.ac.kr/Course.do?cmd=viewStudyHome&boardInfoDTO.boardInfoGubun=study_home&courseDTO.courseId=' +
        qs;
      window.open(
        url,
        '_blank',
        'top=10, right=10,location=no, width=1400, height=1200, directories=no, status=no, menubar=no, toolbar=no, resizable=yes',
      );
    },
    routerToDKUeleaning() {
      let url =
        'https://webinfo.dankook.ac.kr/member/logon.do?returnurl=http://lms.dankook.ac.kr&sso=ok';
      window.open(
        url,
        '_blank',
        'top=10, right=10,location=no, width=800, height=800, directories=no, status=no, menubar=no, toolbar=no, resizable=yes',
      );
    },
  },
};
</script>

<style scoped>
/* header */
.header {
  padding: 20px;
  height: 200px;
  display: block;
  margin: 0 auto;
  background-color: #fff;
  border-bottom: 2.5px solid #e2c100;
}
.header .intro {
  text-align: center;
  font-size: 40px;
  letter-spacing: 3px;
  padding: 15px;
  color: #2c2c2c;
  font-weight: 430;
}
.header .search {
  text-align: center;
  margin: 30px;
  padding: 10px;
}
.header .search .search-title {
  text-align: center;
  font-size: 17px;
  color: #191919;
  margin: 30px;
  border: 2px solid #e2c100;
  background-color: #fff;
  padding: 13px;
  font-weight: 350;
}
.header .search .search-box {
  width: 380px;
  height: 50px;
  font-size: 25px;
  text-align: center;
  letter-spacing: 3px;
  margin-right: 0px;
  border: 2px solid #e2c100;
  border-right: 0px;
}
.header .search .search-button {
  margin-left: 0px;
  width: 70px;
  height: 56px;
  font-size: 28px;
  letter-spacing: 3px;
  border: 2px solid #e2c100;
  background-color: #e2c100;
  color: #fff;
}
.header .search .search-button:hover {
  background-color: #ffe342;
  border: 2px solid #ffe342;
}
.header .prof {
  font-size: 20px;
  text-align: center;
  /* display: block; */
  margin-top: 0px;
  padding-top: 30px;
  letter-spacing: 1px;
  margin-right: 30px;
  display: block;
  width: 270px;
  margin: 0 auto;
}
/* !header */
/* body */
.body {
  margin: 0 auto;
  padding-top: 70px;
  width: 1000px;
  height: 700px;
  /* border: 1px solid darkcyan; */
}
.body .body-left {
  margin: 0px;
  float: left;
}
.body .body-right {
  float: right;
  width: auto;
}
.body .title {
  font-size: 20px;
  margin: 70px;
  letter-spacing: 2px;
  padding: 30px;
  line-height: 60px;
  /* border: 1px solid orange; */
  padding-bottom: 5px;
  border-bottom: 2px solid #e2c100;
}
.body-left .course-list {
  list-style: none;
  font-size: 19px;
  padding-left: 10px;
  letter-spacing: 1.5px;
  /* border: 1px solid blueviolet; */
  padding-top: 5px;
  /* text-align: center; */
}
.body-left li {
  border: 1px solid #e2c100;
  padding: 5px;
  margin: 5px;
  width: 350px;
  background-color: #fff;
  border-radius: 5px;
  color: #191919;
  font-weight: 400;
}
.body-left li:hover {
  background-color: #eccc16;
  border: 1px solid #eccc16;
  color: #fff;
}
.body-right .content {
  padding-top: 10px;
  /* border: 1px solid black; */
}
.body-right .content p {
  font-size: 24px;
  margin-left: 100px;
  font-weight: 300;
}
.body-right li {
  border: 1px solid #e2c100;
  background-color: #fff;
  list-style: none;
  padding: 5px;
  margin: 5px;
  width: 500px;
  font-size: 18px;
  color: #191919;
  font-weight: 400;
}
.body-right .semester:hover {
  background-color: #eccc16;
  border: 1px solid #eccc16;
  color: #fff;
}

.body .subtitle {
  font-size: 15px;
  font-weight: bold;
  letter-spacing: 1px;
  padding: 5px;
}
.pending-page .title {
  text-align: center;
  font-size: 20px;
  padding: 10px;
  letter-spacing: 2px;
  padding-top: 50px;
  margin: 0 auto;
  width: 160px;
  padding-bottom: 7px;
  border-bottom: 2px #e2c100 solid;
}
.pending-page {
  margin: 10px;
}
.pending-page .subtitle {
  font-size: 18px;
  width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-weight: 400;
}
.pending-page .pending-explain {
  width: 710px;
  height: 70px;
  line-height: 30px;
  margin: 0 auto;
  letter-spacing: 1.5px;
  padding: 5px;
  font-weight: 350;
}
.pending-explain em {
  color: red;
  font-weight: 600;
}
.pending-page button {
  text-align: center;
  margin: 0 auto;
  display: block;
  padding: 17px;
  color: #fff;
  font-size: 20px;
  letter-spacing: 2px;
  background-color: #e2c100;
  border: 1px solid #e2c100;
  margin-top: 15px;
  margin-bottom: 30px;
}
.pending-page button:hover {
  background-color: #eccc16;
  border: 1px solid #eccc16;
}
/* 예외 */
.empty {
  font-size: 30px;
  text-align: center;
}
.warning {
  color: red;
}
</style>