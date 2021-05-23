---
layout: post
title:  "Node.js And Babel"
date:   2019-08-25 08:43:59
author: Mars
categories: javascript
---
Node.js는 아직 ES6 문법 제대로 지원하지 않는다.
따라서 ES6 혹은 Typescript를 사용하고 싶다면 추가적인 설정, 즉 Babel이 필요하다.  


Babel을 사용해서 ES6 코드를 ES5로 변환(transpile)할 수 있다. 
{% highlight javascript %}

{% endhighlight %}
 


### Installation
<https://reactjs.org/docs/add-react-to-a-new-app.html>


### Create React App
#### create-react-app 설치하기
create-react-app 을 사용하면 쉽게 react app을 초기화할 수 있다.
- node 버전 6 이상이 필요함.


#### npm
{% highlight bash %}
npm install -g create-react-app
create-react-app my-app

cd my-app
npm start
{% endhighlight %}

브라우저에 React 샘플 앱이 렌더링될거다.

#### npx
npm 버전 5.2.0+ 이면, npx 를 사용할 수 있단다.
{% highlight bash %}
npx create-react-app my-app
cd my-app
npm start
{% endhighlight %}

#### Production 빌드하기
npm run build 를 실행하면 프로덕션 배포용 build를 생성해준다. (build 폴더에 앱 빌드가 생성됨.)



### Quick Start Hello World
{% highlight javascript %}
ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('root')
);
{% endhighlight %}
"Hello, world!" 를 페이지에 렌더링하는 코드이다.


### Introducing JSX
아래 변수 선언을 좀 보자.
```
const element = <h1>Hello, world!</h1>;
```
이건 스트링도 아니고 HTML도 아니다.
JSX 라는 놈이고, 자바스크립트 문법을 확장한거다. React랑 같이 쓰기를 권장하며, UI가 어떻게 보여야하는지 설명해주는거 같쟈나.

JSX 는 React "elements"를 만들 때 쓰는건데, 이런 React Elements가 어떻게 DOM에 렌더링 되는지는 나중에 살펴보고, 우선은 JSX를 좀더 알아보자.

### Why JSX?
React 는 UI 로직이 렌더링 로직과 본질적으로 관련이 깊다는 것에 착안했다. 이벤트가 어떻게 처리되는지, 시간이 지남에 따라 어떻게 상태가 변하는지, 데이터가 표시되기 위해 어떻게 준비되어야 하는지 ...


마크업과 로직을 분리해서 부자연스러워지기 보다는, React는 이 둘을 "components"라고 불리는 유닛으로 만들어서 Separates Of Concerns를 달성한다. 당연히 컴포넌트라는 개념도 나중에 살펴보자.
어쨋든 아직까지는 마크업을 JS에 넣는게 어색하게 느껴질테니까 이거 한번 보자. 아마 생각이 바뀔껄?
- https://www.youtube.com/watch?v=x7cQ3mrcKaY

React 가 JSX 사용을 강제하는건 아니지만, 대부분이 이 방식을 더 유용하다고 생각한다.

### Embedding Expressions in JSX
JSX 내부에 자바스크립트 expression(표현식)을 쓰고 싶다면 { } 로 감싸면 된다.
```
const element = (
  <h1>
    Hello, {formatName(user)}
  </h1>
);
```
```
ReactDOM.render(
  element,
  document.getElementById('root')
);
```

가독성을 유지하기 위해 JSX를 여러줄에 걸쳐서 작성했다. 그리고 혹시나 에디터의 자동완성 기능이 세미콜론(;)을 넣을까봐 소괄호 ()로 묶었다.


### JSX is an Expression Too
JSX도 하나의 expression 이므로 아래처럼 다른 자바스크립트 코드와 섞일 수 있다.
```
if (user) return <h1>Hello, {formatName(user)}</h>;
else return <h1>Hello, Stranger</h1>;
```

### Specifying Attributes with JSX
attribute 를 사용하고 싶으면 쌍따옴표를 쓰자.
```
const element = <div tabIndex="0"></div>;
```

당연히 자바스크립트 expression 으로 표현할 수도 있다.
```
const element = <img src={user.avatarUrl}></img>;
```

* JSX가 HTML 보다는 자바스크립트에 더 가깝기 때문에, React DOM은 property naming convention으로 camelCase를 사용한다. 예를 들어 class는 className, tabindex는 tabIndex 이런 식이다.

### Specifying Children with JSX
Empty 태그도 허용된다.
```
const element = <img src={user.avatarUrl} />;
```

자식 태그도 허용된다.
```
const element = (
  <div>
    <h1>Hello!</h1>
    <h2>Good to see you here.</h2>
  </div>
);
```

### JSX Prevents Injection Attacks
유저 입력을 JSX 에 표현할려면 인젝션 공격을 방어하는게 안전하다. 디폴트로, React DOM은 JSX에 포함된 값을 escape 처리한다. 따라서 XSS 공격을 방어하는데 도움이 된다.
- https://stackoverflow.com/questions/7381974/which-characters-need-to-be-escaped-on-html
- https://en.wikipedia.org/wiki/Cross-site_scripting

### JSX Represents Objects
Babel 은 JSX 를 React.createElement() 구문으로 컴파일한다.

아래 두 코드는 동일하다.
```
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
```
```
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```
React.createElement() 의 역할은 아래와 같은 JSON 객체를 생성하는 거다.
```
// Note: this structure is simplified
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world'
  }
};
```
이런 객체를 "React elements"라고 부르고, React 는 이 객체를 읽어서 DOM 을 생성하거나 DOM 의 상태를 최신으로 유지하는데 쓴다.
React elements 를 DOM 으로 렌더링하는건 나중에 더  살펴보자.

* 참고로 Babel language definition 을 쓰면 당신의 개발툴에서 ES6와 JSX 코드가 하이라이팅된다. 이거 바바 이쁘지? https://labs.voronianski.com/oceanic-next-color-scheme/

### Rendering Elements
Elements 는 리엑트에서 가장 작은 building block 이라고 볼 수 있다.
브라우저 DOM 엘리먼트랑은 다르게, React elements 는 단순한 객체이고, 생성 비용이 저렴하다. React DOM 은 React elements 에 대응하는 DOM 을 알아서 업데이트해준다.


### Rendering an Element into the DOM
HTML 파일에 <div> 태그에 있다고 가정해보자.
```
<div id="root"></div>
```

이 태그 내부의 모든 것들이 React DOM 에 의해 관리될 것이므로 이 태그를 "root" DOM 노드라고 부른다.

React 앱은 보통 하나의 루트 DOM 노드를 가진다. (만약 기존의 앱에 React 를 적용하고자 한다면 여러개의 root DOM 노드를 가지게 될거다.)

React element 하나를 DOM 으로 렌더링하고 싶으면 ReactDOM.render() 를 호출하면 된다.
```
const element = <h1>Hello, world</h1>;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```
코드팬에서 확인해볼 수 있다.
- http://codepen.io/gaearon/pen/rrpgNB?editors=1010


### Updating the Rendered Element
React elements 는 불변객체(immutable)이다. 일단 element 를 하나 만들었다면 변경할 수 없다. 이건 마치 영화의 프레임처럼 간주될 수 있다. 특정 시점의 UI인 것이다.

지금까지 우리가 배운 유일한 UI 업데이트 방식은 새로운 element 를 만들어서 ReactDOM.render() 에 전달하는 방법 뿐이다.

똑딱 시계 예제를 생각해보자.
```
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(
    element,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```


이 예제는 setInterval()을 이용해서 매초마다 ReactDOM.render()를 호출한다.

* 실제로는, 대부분의 React 앱이 ReactDOM.render()를 오직 한번만 호출한다. 다음 섹션에서 stateful components 를 보면 알게된다.

### React Only Updates What’s Necessary
React DOM 은 기존 elements 와 신규 elements 를 비교해서, 업데이트가 필수인 DOM 만 적용한다.

똑딱 시계 예제를 브라우저 툴로 inspect 해서 확인해볼 수 있다. 심지어 우리가 전체 요소를 표현하는 React elements 를 생성하더라도 필요한 부분만 변경된다.

경험상, 특정시점에 UI가 어떻게 보여야만 하는지 생각하는게 시간이 지남에 따라 UI가 어떻게 변경되어야 하는지 고민하는 것 보다 버그를 줄이는데 낮더라.

### Components and Props
컴포넌트는 독립적이고 재사용 가능한 UI 조각이다. 개념적으로는 자바스크립트 함수랑 유사하고, 임의의 입력값을 받아서(props라고 불림), React elements를 반환해준다.

### Functional and Class Components
컴포넌트를 정의하는 가장 단순한 방법은 자바스크립트 함수를 만드는거다.
```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

이 함수는 유효한 React 컴포넌트인데 props를 파라미터로 받고, 하나의 React element를 반환하기 때문이다. 이러한 컴포넌트를 "functional"이라고 부르는데, 딱 봐도 자바스크립트 함수니까...

ES6 class 문법을 이용해서도 컴포넌트를 만들 수 있다.
```
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```
React 관점에서 위 두개의 컴포넌트는 동일한거다.

Class 컴포넌트는 몇 가지 기능들이 더 있는데, 다음 섹션에서 살펴보자. 그 전까진 functional 컴포넌트를 좀더 보자.


### Rendering a Component
이전까지는, DOM 태그를 의미하는 React elements들만 살펴봤었다.
```
const element = <div />;
```

이제는, elements가 사용자 정의 컴포넌트를 의미할 수도 있다.
```
const element = <Welcome name="Sara" />;
```

React가 component를 만나면, JSX attributes(tabIndex 같은거, 기억남?) 를 단일 객체로 컴포넌트에게 전달한다. 그리고 이 객체를 "props" 라고 부른다.

예제를 보자.
```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Sara" />;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```
이 예제에서 일어난 것들을 살펴보자.
1. ReactDOM.render()가 호출되었고, 파라미터는 <Welcome name="Sara" /> 엘리먼트였다.
2. React는 Welcome 컴포넌트를 호출했고, 파라미터(props)는 {name: 'Sara'} 이다.
3. Welcome 컴포넌트는 <h1>Hello, Sara</h1> 엘리먼트를 반환했다.
4. React DOM은 효과적으로 <h1>Hello, Sara<h1>에 대응하는 톰을 갱신했다.

* 규칙 : 컴포넌트 이름은 항상 대문자로 시작하자. React 는 소문자로 시작하는 컴포넌트를 DOM 태그로 취급한다. 예를 들어 <div /> 처럼.


### Composing Components
컴포넌트는 다른 컴포넌트를 사용할 수 있다. 따라서 우리는 어디에나 컴포넌트 추상화를 적용할 수 있다. 버튼, 폼, 다이얼로그, 스크린 이런 모든 것들을 컴포넌트로 표현할 수 있다.

예를 들어, 우리는 Welcome 컴포넌트를 여러번 사용하는 App 컴포넌트를 생성할 수 있다.
```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

function App() {
  return (
    <div>
      <Welcome name="Sara" />
      <Welcome name="Cahal" />
      <Welcome name="Edite" />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```
일반적으로, 새로운 React App은 최상위 레벨에 하나의 App 컴포넌트를 가진다. (만약 레거시에 React를 적용해야 한다면 버튼과 같은 작은 컴포넌트로 하나씩 쌓아가야 할거다.)


### Extracting Components
컴포넌트를 작은 컴포넌트로 쪼개는걸 두려워하지 말자.
예를 들어 Comment 컴포넌트를 살펴보자.
```
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <img className="Avatar"
          src={props.author.avatarUrl}
          alt={props.author.name}
        />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```
이 컴포넌트는 author, text, 그리고 date를 props로 가지며, 소셜 미디어 사이트의 댓글을 표현하고 있다.

이런 컴포넌트는 변화에 유연하지 못하며, 각각의 부분을 재사용하기도 어렵다. 여기서 몇개의 컴포넌트를 꺼내보자.


처음으로는 Avatar 컴포넌트를 끄집어내보자.
```
function Avatar(props) {
  return (
    <img className="Avatar"
      src={props.user.avatarUrl}
      alt={props.user.name}
    />

  );
}
```
Avatar는 Comment 내부에서 렌더링 될꺼라는 걸 알 필요가 없다. 따라서 author라는 이름보다는 좀더 범용적인 이름인 user가 낫다. (props.author가 user로 변경됨)

전체적인 상황에 따른 이름보다는 컴포넌트 그 자체의 관점에서 props의 이름을 정하기를 추천한다.

이제 Comment 컴포넌트를 약간 단순하게 만들 수 있다.
```
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <Avatar user={props.author} />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```
이제 Avatar 옆에 렌더링되는 사용자 이름을 컴포넌트로 빼내보자.
```
function UserInfo(props) {
  return (
    <div className="UserInfo">
      <Avatar user={props.user} />
      <div className="UserInfo-name">
        {props.user.name}
      </div>
    </div>
  );
}
```
이제 Comment를 좀더 단순하게 만들어보자.
```
function Comment(props) {
  return (
    <div className="Comment">
      <UserInfo user={props.author} />
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```
처음에는 컴포넌트로 쪼개는 일이 번거롭게 느껴질테지만, 큰 앱을 만들 때는 재사용 가능한 컴포넌트의 팔레트를 가지게 되므로, 그 가치가 충분할거다. 좋은 규칙은 UI 한 부분이 여러 곳에서 사용되어지거나, 그 자체로도 꽤 복잡하거다면 컴포넌트로 빼는걸 고려하는 식이다.


### Props are Read-Only
컴포넌트가 함수형이든 클래스형이든, props는 절대로 수정되어서는 안된다.
sum 함수를 생각해보자.
```
function sum(a, b) {
  return a + b;
}
```
이런 함수를 "pure" 라고 부르는데, 왜냐하면 입력값을 변경하려 하지 않고, 동일한 입력에 대해 항상 같은 결과를 반환하기 때문이다.

"pure" 하지 않은 함수는 이런 식이다.
```
function withdraw(account, amount) {
  account.total -= amount;
}
```
React가 꽤나 유연하긴 하지만, 하나의 엄격한 규칙을 가진다.

모든 React 컴포넌트는 props에 대해서 pure 함수처럼 동작해야 만 한다.

당연히, App UI 는 동적이며 시간이 흐름에 따라 변한다. 다음 섹션에 "state"라는 새로운 개념을 살펴보자. 이 규칙을 어기지 않으면서 유저의 행동이나 네트웍 응답 등등에 따라 컴포넌트의 출력이 변하는 것을 볼 수 있다.


### State and Lifecycle
똑딱 시계 예제를 다시 보자.

지금까지 배운바에 의하면 UI를 업데이트하기 위해서는 ReactDOM.render()를 호출해야만 했다.
```
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(
    element,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

이 섹션에서는 Clock 컴포넌트를 진짜로 재사용 가능하고 추상화된 컴포넌트로 만드는 법을 배우게 될거다.

일단 기존 코드에서 Clock의 UI를 추상화해보자.
```
function Clock(props) {
  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {props.date.toLocaleTimeString()}.</h2>
    </div>
  );
}

function tick() {
  ReactDOM.render(
    <Clock date={new Date()} />,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```
하지만 여전히, Clock 컴포넌트는 timer와 매 초 갱신 요구사항을 가지고 있지 않다.

만약 Clock이 이런 모양이면 만족할 거 같다.
```
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```
이런 요구사항을 달성할려면, state 라는 것이 Clock 컴포넌트에 추가되어야 한다.

State 는 props 와 유사하지만, private 이고 완전히 컴포넌트에 의해서만 제어할 수 있다.

클래스 컴포넌트가 가지는 추가적인 기능 중 하나가 바로 local state이다. (즉 함수형 컴포넌트는 이런거 없다.)

### Converting a Function to a Class
함수형 컴포넌트를 클래스형 컴포넌트로 바꾸는 5 단계:
1. ES6 클래스를 만들어서 함수형 클래스와 동일한 이름을 주고, React.Component를 상속받자.
2. render() 함수를 만들자.
3. 함수형 컴포넌트의 구현을 render() 함수로 옮기자.
4. render() 함수의 구현에서 props를 this.props로 변경하자.
5. 함수형 컴포넌트를 삭제하자.
(엄청 긴데 별거 없음, 클래스 추가하고 this.props로 수정)

이제 클래스형 컴포넌트가 생겼으므로, local state 랑 lifecycle hooks 를 사용할 준비가 됐다.


### Adding Local State to a Class
To be continued ... someday







