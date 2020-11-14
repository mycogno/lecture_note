# ReactJS

git remote add origin "github 주소" 의 의미는?

git add. // git commit -m "" // git push origin master

- index.html 소스코드에는 아무것도 없음
- js와 요소를 만들고 그것을 html에 넣음
- react가 virtual DOM(가상의! 실제로는 코드에 존재하지 않는)을 만든다
- component는 html을 반환하는 함수
- jsx : html과 js 사이
- react application은 한번에 하나의 component만을 rendering 해야 함
- application 안에 여러 component들을 포함할 수 있음

---

- jsx + props로 component 재사용 가능, component에 정보 전달 가능
- component에 props를 넣을 수 있음
- props는 객체 형태로 argument로 전달 됨
- props.fav 대신에 { fav }를 argument로 쓸 수 있다.
- component 이름은 대문자로 가야함

---

- js map함수 : array 각 item에서 function의 result를 갖는 array를 줌
- key prop를 써서 uniqueness를 주는 것이 좋음, , 실제로 사용하지는 않음(react 내부에서만 사용)
- prop-types : 내가 전달받은 props가 내가 원하는 prop인지 확인해줌
    - type, 필수 여부 등 확인 가능

---

- dynamic data를 다루는 state
- function component와 class component
- class component는 return이 아닌 render method를 가지고 있음
    - react는 자동으로 모든 class component의 render method를 실행하려고 함
- state는 object이다(데이터가 변함, 동적)
- react에서는 자동적으로 주어진 <button>이 onClick prop을 가지고 있음
- onClick={ this.add } 는 add 함수 자체를 onClick 프로퍼티에 할당해두는 것이고 onClick={ this.add() } 는 add 함수를 '실행' 한 결과를 onClick 프로퍼티에 담는 것

---

- "state를 직접 변경하지 마시오"
- react는 render function을 refresh하지 않음
- setState를 사용하지 않으면 새 State와 함께 render function이 호출되지 않음(업데이트가 안돼)
- setState를 호출할 때마다 react는 새로운 state와 함게 render function을 호출

---

- life cycle method : react가 component를 생성하고 없애는 방법
- mounting(아래 항목 순서대로 call)
    - constructor(js에서 class만들 때 호출)
    - render()
    - componentDidMount() : 이 component는 처음 render 됐어! 를 알려줌
- updating : state를 변경
- unmounting : component가 죽을 때

순서 : setState호출 → component 호출 → render() 호출 → update 완료 →  componentDidUpdate 호출

---

- setState를 사용할 때 state 안에 default 값들을 선언할 필요는 없음

---

- async, await : 무언가 기다려야해!
- movies:movies를 { movies }로 쓸 수 있다.(이름이 같을 때)

---

- style component?

---

- jsx에서 class 키워드는 component class와 헷갈리므로 className 이라고 써야한다.
- map 함수는 item과 item number라는 argument를 준다.

---

- gh-pages
- git remote -v (리모트 저장소 목록?)
- npm run build
- npm run deploy
- deploy는 predeploy를 호출

---

- state를 갖기 위해 꼭 class component를 쓸 필요가 없다?
    - react hook 때문에, class component를 쓰는 다른 방식

---

- react-router-dom : navigation 만들어주는 툴
- /about 이면 About.js , /home이면 Home.js 컴포넌트를 불러줌
- /home과, /home/introduction 이면 2개의 라우터가 매치됨 (겹쳐서 렌더링)
- exact={true} 를 추가해주면 딱 그 주소 일 때만 렌더링

---

- html에 의해 새로고침 되어 버리면 react가 죽음...
- 그래서 <a> 대신 <Link>를 씀 Link는 router 안에 있어야 함!
- BrowserRouter는 HashRouter와 어떤 차이가? (BrowserRouter는 github page에 무언가 설명해야한다고 합니다.)

---

- route props
- Router 안에 Route 들은 모두 props를 가짐