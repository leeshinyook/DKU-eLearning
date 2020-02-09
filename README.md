# DKU-eLearning

> 과거 단국대학교 이러닝 자료조회 해보기! 

## 궁금증

> 대학교는 보통 이러닝이라는 사이트를 학습자료 배포, 공지, 교수님과의 소통의 장으로 사용된다.
>
> 2019년 11월쯤, 나 역시 하루는 이러닝에서 자료를 다운받던 중 아래의 URI를 보고 이상한 호기심과 궁금증이 생겼다.

##### http://lms.dankook.ac.kr/Course.do?cmd=viewStudyHome&boardInfoDTO.boardInfoGubun=study_home&courseDTO.courseId=201924034801

- 물론 클릭시 아무것도 뜨지 않는다. (로그인이 안되어있기 때문)

#### courseDTO.courseid=201924034801

- Dto는 Data Transfer Object를 뜻한다. 즉, 데이터를 전달하는 객체이다.

> 학교 이러닝은 과목아이디 식별값을 가지고 그에 맞는 데이터를 가져온다. GET!

![스크린샷 2020-02-10 오전 3 21 23](https://user-images.githubusercontent.com/55838461/74107527-70fc4980-4bb4-11ea-939e-a55a43eac920.png)

- 그러면 뒤에 courseid의 규칙을 알수있다면 다른 이러닝자료도 ~~훔쳐볼수~~ 찾아볼 수 있지 않을까?
  - 내가 이 과목을 듣는지 확인하는 인증방식이 구현되어있을까?
  - 뒤 숫자는 무엇을 의미하나?

> 위 처럼 이상한 고민이 생겼다. 당시 그 교수님은 굉장히 이러닝에 과제와, 해답을 올려주셔서 내가 만약
>
> 이 교수님의 작년자료도 볼 수 있다면, 나름 굉장한 수확이었으니까 (교수님들은 과제를 매년 똑같이 재탕하는 경우가 많다.)
>
> 잠시 고민 끝에 규칙을 찾았다!



## 규칙은 뭐지?

> 우선 숫자로만 구성이 되어있었기에, 분명 무작위가 아닌, 해당 과목을 구별할 수 있는 규칙이 있을 것이고, 학교 사이트를 뒤져서 과목과 숫자가 엮여있는 것들을 모조리 모았다. 그 결론은 다음과 같다.

> 결론은 Database에 과목을 저장해야하는데 어떤 필드를 이용할까? 의 질문과 같았다.

- 2019 : 해당 과목의 수강년도 `Year`
- 2 : 학기 `Semester`
- 403480 : 과목코드 `Course_code`
- 01 : 분반 `Course_class`

> 내가 학교 수강과목에 대한 데이터베이스를 보지는 못하지만, Course테이블에는 아마 저렇게 필드가 짜여져 있을 것이다!

~~~
저기서 고유한값은 무엇일까? 단 하나로 구별될 수 있는 후보키같은 존재는 바로 과목코드다. 과목코드는 변하지 않을 것 이다.
~~~



## 그렇다면 인증방법에 대한 구현은?

> 숫자규칙을 알았으니 2019를 2018로 바꿔서 들어가보았다.
>
> 그리고, 인증없이 들어가졌다. 



## 자동화

> 이제 프로그래밍을 배웠다면 나는 이것을 자동화하기로 생각했다.
>
> 자동화를 하기 위해서는, 자료가 필요했다. 이 과목이 언제열리고, 분반, 교수를 알아야 하나의 URI를 만들 수 있다.
>
> 다행히 `단쿠키` DB에 원하는 정보가 있었고 이를 활용하여 학우들을 위해 제작하기로 했다.



## Back-end

~~~javascript
var express = require("express");
var router = express.Router();

var middle = require("./middleware.js");
var db = require("../models/db");
require("../models/lecture");

var Lecture = db.model("Lecture");

// 강의 조회
router.get("/", middle.requireLoggedIn, (req, res, next) => {
  const search_word = req.query[0];
  Lecture.find({
    prof_name: search_word
  })
    .then(list => {
      res.json({
        success: true,
        list
      });
    })
    .catch(next);
});
// 과목, 교수명
router.get("/courseinfo", (req, res, next) => {
  const course = req.query[0];
  const prof = req.query[1];
  Lecture.find({
    name: course,
    prof_name: prof
  })
    .then(list => {
      res.json(list);
    })
    .catch(next);
});
module.exports = router;
~~~

> 백엔드는 사실 매우 간단하다. REST방식에 맞게 GET을 택하고
>
> 교수명으로 데이터를 가져오는것과, 과목과 교수명으로 과목을 가져오면 된다.
>
> 사실 이 둘은 하나로 줄일 수 있다. 이미 교수명으로 list를 가져오는것에 모두 포함되어있기 때문이다.
>
> 하나로 받아왔을때, 프론트에서 받아온 모든 데이터를 과목별로 정리하기 어려웠고, 데이터베이스에 저장된 방식이 달라진 
>
> 경우가 있었기 때문에, 모든 예외처리를 고려하는 것보다. 차라리 한번 더 GET요청을 보내는 것을 택했다!



## Front-end

~~~javascript
routerToDKU(idx) {
      let year = this.course_info[idx].year;
      let semester = this.course_info[idx].semester;
      let code = this.current_code;
      let class_no = this.course_info[idx].class_no;
      let qs = (year + semester + code + class_no).toString();
      let url ='http://lms.dankook.ac.kr/Course.do cmd=viewStudyHome&boardInfoDTO.boardInfoGubun=study_home&courseDTO.courseId=' +
        qs;
      window.open(
        url,
        '_blank',
        'top=10, right=10,location=no, width=1400, height=1200, directories=no, status=no, menubar=no, toolbar=no, resizable=yes',
      );
    },
~~~

> 위 방식처럼 조합하여 해당 이러닝으로 갈수 있는 팝업을 열어준다.



## 배포

![2020-02-10 03 10 07](https://user-images.githubusercontent.com/55838461/74107421-20d0b780-4bb3-11ea-8e09-f73b90574566.gif)

![2020-02-10 03 10 30](https://user-images.githubusercontent.com/55838461/74107422-2201e480-4bb3-11ea-8815-cea1a1a7f19a.gif)

![2020-02-10 03 10 51](https://user-images.githubusercontent.com/55838461/74107423-229a7b00-4bb3-11ea-9902-834bae4f05d2.gif)

> https://www.dankookie.com/elearning 에서 확인할 수 있습니다.



## 느낀점

> 완성까지 아침에 시작해서 저녁전에 끝났던 것 같다.
>
> 무엇인가 개발을 했다기보다. 이렇게 발견을 통해 불편했던 점을 자동화하고 이러한 자료가 수강신청을 하는 입장에서는
>
> 미리 자료를 보고 판단할 수 있어서 굉장히 좋은 방법이 될 것 같다. 학우들도 잘 이용해주면 좋겠다:)
>
> 참 프로그래밍은 매력적이다!

