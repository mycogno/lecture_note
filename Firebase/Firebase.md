# Firebase

- 아이디어를 백엔드 구현 없이 빠르게 테스트하는데 주로 사용
- import firebase from "firebase/app"; 를 이용한 불러오기
- env 파일 만들때 환경변수에 REACT_APP_ 앞에 붙이는거 필수
- 빌드하면 결국은 api key가 노출되게 되어있다. 다만, github에 코드 노출시키지 않기위해 하는 것
- npm i react-router-dom
- Fragment : 많은 요소들을 render 하고 싶을 때 (부모 요소 없을 때) <></>

---

- auth 사용전 먼저 import 해야함
- jsconfig.json 파일을 이용해 import 파일경로 간결화
- authService.currentUser 이용해 로그인 여부 알 수 있다.