---
layout: post
title:  "Vue: Vuex Wrap-Up"
date:   2018-01-13 12:56:59
author: Mars
categories: javascript
---

### Vuex State

##### Single State Tree
Vuex를 사용해서 앱의 모든 컴포넌트와 공유하는 단일 상태 객체(single source of truth)를 도입하자.
`Vue.use(Vuex)`를 사용해서 모든 자식 컴포넌트에 Single Source of Truth가 **inject**되도록 하자:

```
Vue.use(Vuex)

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

##### Getting Vuex State into Vue Components
컴포넌트에서는 computed property를 사용해서 state에 접근하자:

```
const Counter = {
  template: `<div>8</div>`,
  computed: {
    count () {
      return this.$store.state.count
    }
  }
}
```


##### The `mapState` HelperObject
mapState 핼퍼를 쓰면 컴포넌트의 computed property에 state에 대한 핼퍼 객체를 할당할 수 있으므로, computed getter를 쉽게 만들 수 있었다.

그런데 만약, 로컬 computed property와 Vuex State의 computed property를 둘 다를 쓰고 싶다면 ES6 스프레드 연산자를 활용하자:

```
computed: {
  localComputedProperty () { return this.problemsSolved },
  ...mapState({
    pv: state => state.pv,
    uv: state => state.uv
  })
}
```

##### Components Can Still Have Local State
Vuex의 비용이 저렴하진 않다는 걸 기억하자.

### Getters
Computed property는 state의 값을 그대로 사용할 때 유용하다. 만약 약간의 조작이 필요한 경우라면(예를 들어, 완료된 할일 목록) Getter를 쓰자:

```
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    }
  }
})

```

Getters는 store.getters로 접근할 수 있다:

```
store.getters.doneTodos // -> [{ id: 1, text: '...', done: true }]
```
이제 컴포넌트에서 가져다 쓰기만 하면 된다:

```
computed: {
  doneTodosCount () {
    return this.$store.getters.doneTodosCount
  }
}
```

Getters에 파라미터를 전달하고 싶다면 좀 복잡했었다:
* ES6의 화살표 함수를 두 번 활용했다.
```
getters: {
	getTodoById: (state) => (id) => {
  	return state.todos.find(todo => todo.id === id)
	}
}
store.getters.getTodoById(2) // -> { id: 2, text: '...', done: false }
```

State에 대한 mapState 핼퍼가 있었듯이, Getters에 대한 mapGetters 핼퍼도 있다는 것 기억하자.


### Mutations
Vuex의 state를 수정(mutate)하고 싶으면, mutations를 활용하자:

```
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment (state) {
      // mutate state
      state.count++
    }
  }
})
```
Mutations를 호출하고 싶으면 아래와 같이 써야한다:

```
store.commit('increment')
```
파라미터인 'increase'를 타입이라고 하며, 이에 대응하는 함수를 핸들러라고 했었다.

##### Commit with Payload
추가적인 파라미터도 전달할 수 있는데, 이를 payload 라고 부른다:
```
// ...
mutations: {
  increment (state, n) {
    state.count += n
  }
}
store.commit('increment', 10)
```
그런데 commit()을 호출할 때, payload 객체 하나로 전달하는 방법도 있다:
```
store.commit({
  type: 'increment',
  amount: 10
})
```
이 방법이 mutation 로그를 볼 때, 더 유용하다.

##### Mutations Must Be Synchronous
Mutation 핸들러는 반드시 동기화 코드만 처리해야 한다. 비동기 코드를 가진 mutation 핸들러는
핸들러 호출 결과를 로깅한다고 가정했을 때 before와 after 스냅샷이 정확하지 않게 되므로 상태 추적이 어려워진다.

만약 비동기 코드를 써서 상태를 갱신(mutate)하고 싶다면 Actions를 사용하자.

### Actions
Actions의 기본형태는 아래와 같다:

```
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      context.commit('increment')
    }
  }
})
```
Action 핸들러의 파라미터로 store와 거의 유사한 context가 파라미터로 전달된다는 것을 기억하자.

그런데 context에서 우리가 관심있는 부분은 commit 메서드 하나이므로, 이렇게 디스트럭처링해서 쓸 수 있었다.
```
actions: {
  increment ({ commit }) {
    commit('increment')
  }
}
```
##### Dispatching Actions
Actions는 store.dispatch 메서드로 트리거된다:
```
store.dispatch('increment')
```
그런데 action은 비동기 코드를 처리하기 위한 목적임을 상기해보자.
따라서 실제 코드는 아래와 좀더 유사할꺼다:
```
actions: {
  incrementAsync ({ commit }) {
    setTimeout(() => {
      commit('increment')
    }, 1000)
  }
}
```

일단 이만큼 이해했으면 개념 정리 끝.
