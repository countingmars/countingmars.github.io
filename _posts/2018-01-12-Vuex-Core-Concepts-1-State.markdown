---
layout: post
title:  "Vue: Vuex Core Concepts - 1. State"
date:   2018-01-13 08:43:59
author: Mars
categories: javascript
---

### State
https://vuex.vuejs.org/en/state.html


#### Single State Tree
Vuex 는 단일 상태 객체인 Tree 를 사용한다. 그러니까 하나의 앱의 상태를 관리하는 하나의 객체가 제공된다는 말이지. 이것을 유식하게 single source of truth 라고 부르는거 같다.

일단 이런 형태가 가장 직관적으로 상태를 표현한다고 볼 수 있고, 디버깅 할 때도 현재 상태를 쉽게 볼 수 있다.

그리고 하나의 상태 트리(그러니까 일종의 전역변수 느낌)가 왜 모듈화랑 충돌하지는 않는지는 나중에 살펴보자.

#### Getting Vuex State into Vue Components
Vue 컴포넌트에서 어떻게 Store (Single State Tree를 의미)의 상태를 디스플레이할까?
Vuex store는 reactive 하기 때문에, 상태를 "조회"하는 가장 쉬운 방법은 computed property를 사용하는거다:
```
// let's create a Counter component
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return store.state.count
    }
  }
}
```

store.state.count 가 변경되면, computed property가 재평가될테고, 관련된 DOM 을 업데이트하게 만들거다.
* 아마도 reactive 하다는 것이 무슨 말인지 이해해야 충분히 납득할 수 있을거다.

그런데 이 방식(pattern)의 문제는 컴포넌트가 global store singleton에 의존하는거다.
모듈 시스템을 사용하게 되면, store를 사용하고 싶은 모든 컴포넌트가 store를 임포트해야 하는데다가 심지어 테스트하고 싶으면 store를 mocking해야 한다.


다행히도 root 컴포넌트에 store 옵션을 적용하면, Vuex는 모든 자식 컴포넌트에도 "inject" 해주는 메커니즘(`Vue.use(Vuex)`)을 제공해준다:
```
const app = new Vue({
  el: '#app',
  // provide the store using the "store" option.
  // this will inject the store instance to all child components.
  store,
  components: { Counter },
  template: `
    <div class="app">
      <counter></counter>
    </div>
  `
})
```
위의 예제처럼 root 컴포넌트에 store를 제공해주면, 모든 자식 컴포넌트에도 inject 되고 `this.$store`로 접근할 수 있게 된다.
위에 있는 Counter 구현을 수정해보자:
```
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return this.$store.state.count
    }
  }
}
```

### The mapState Helper
만약 컴포넌트가 여러개의 상태를 알아야할 상황이라면, 노가다성 반복 작업을 좀 해야할꺼다.
이런걸 해결하려면 mapState helper라는 computed getter를 생성해주는 놈을 써보자. 타이핑 노가다를 조금 줄여준다:
```
// in full builds helpers are exposed as Vuex.mapState
import { mapState } from 'vuex'

export default {
  // ...
  computed: mapState({
    // arrow functions can make the code very succinct!
    count: state => state.count,

    // passing the string value 'count' is same as `state => state.count`
    countAlias: 'count',

    // to access local state with `this`, a normal function must be used
    countPlusLocalState (state) {
      return state.count + this.localCount
    }
  })
}
```
store.state의 속성명과 computed property의 이름이 같다면 문자열 배열로 전달해도 된다.
```
computed: mapState([
  // map this.count to store.state.count
  'count'
])
```

### Object Spread Operator
지금까지는 자식 컴포넌트의 computed 속성에 mapState 객체를 전달했었다. (따라서 local computed property는 사용할 수 없는 상황이다.)
그런데 만약 자식 컴포넌트의 local computed property와 store.state 둘 다를 써야하는 상황이라면?
스프레드 연산자(ES6 stage-3 스펙)를 쓰면 엄청 쉬워진다:

```
computed: {
  localComputed () { /* ... */ },
  // mix this into the outer object with the object spread operator
  ...mapState({
    // ...
  })
}
```

### Components Can Still Have Local State
Vuex 를 사용하겠다는 것이 모든 상태를 Vuex 에 의존하겠다는 것은 아니다.
앱의 상태값을 Vuex에 넣는게 상태의 변경과 디버깅을 좀더 명시적으로 만들어주긴 하겠지만,
이런 방식은 종종 코드를 좀더 길게 만들고 직관적이지 않게(indirect) 만든다.

그러니까 특정 상태가 하나의 컴포넌트에만 속한다면, local state로 관리해도 괜찮다.
trade-off 가 뭔지 고민하면서, 상황에 맞는 적절한 결정을 내리자.
