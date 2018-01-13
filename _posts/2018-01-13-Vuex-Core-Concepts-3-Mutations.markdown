---
layout: post
title:  "Vue: Vuex Core Concepts - 3. Mutations"
date:   2018-01-13 10:43:59
author: Mars
categories: javascript
---

### Mutations
Vuex의 store.state를 수정하고 싶으면, 우선 Vuex mutations 객체를 정의해야 한다.
mutations 객체에 정의될 각각의 mutation은 아래의 예제와 같은 형태이며 첫 번째 파라미터로 state를 전달받는다:

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
위 코드에서 함수명인 increase를 타입이라고 부르며, 함수 자체는 핸들러라고 부른다.
위의 mutations 정의는 마치 이벤트 등록과 유사하며, 이렇게 해석할 수 있다: "increase 타입의 mutation이 발생하면(trigger), 그 핸들러를 호출하라."
만약 increase에 대한 mutation 핸들러를 실행하고 싶으면 `store.commit(type)`를 호출해야 한다:
```
store.commit('increment')
```

### Commit with Payload
store.commit을 호출할 때 추가적인 파라미터를 전달할 수 있는데, payload 라고 부른다:
```
// ...
mutations: {
  increment (state, n) {
    state.count += n
  }
}
store.commit('increment', 10)
```
웬만하면 payload는 오브젝트여서 여러 필드를 포함할 수 있어야 하고, mutation이 로깅된(recorded)다면, 더욱 서술적(decriptive)으로 보일꺼다:
```
// ...
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
store.commit('increment', {
  amount: 10
})
```

### Object-Style Commit
mutation을 수행하는 다른 방법은 위의 방식처럼 파라미터 두 개(type과 payload)를 전달하는 것이 아니라, 하나의 오브젝트로 전달하는 것이다:
```
store.commit({
  type: 'increment',
  amount: 10
})
```
object-style commit 스타일을 사용하더라도 전체 객체가 payload로 전달되기 때문에 핸들러 코드가 변경되진 않을꺼다:
```
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
```

### Mutations Follow Vue's Reactivity Rules
Vuex의 store.state 가 reactive 하기 때문에, state를 변화(mutate)시킬 때,
이 상태와 관련된(observing) 컴포넌트는 자동으로 업데이트 될꺼다.
그런데 이것이 의미하는 바는 Vuex mutations이 plain Vue와 연동될 때, 동일한 reactivity 규칙을 따라야 한다는 것이다:
1. store의 state와 모든 필드들은 초기화되는 것이 낫다(prefer).
2. 하나의 객체에 새 속성이 추가되어야 한다면, 아래 두 규칙 중 하나를 사용해야 한다:
- Vue.set(obj, 'newProp', 123)
- 기존 객체를 새 객체로 대체하면서 새 속성을 추가해야 한다. 예를 들어 스프레드 연산자와 함께 이렇게 작성할 수 있다:
```
state.obj = { ...state.obj, newProp: 123 }
```

### Using Constants for Mutation Types
Flux를 살펴보면 Mutation 타입으로 상수를 사용한다는 것을 알 수 있다.
이렇게 함으로써 linter와 같은 툴의 장점을 누릴 수 있고, 모든 상수를 하나의 파일에 넣음으로써, 동료들이 무엇을 mutation할 수 있는지 한번에 알 수 있게 된다:
```
// mutation-types.js
export const SOME_MUTATION = 'SOME_MUTATION'
// store.js
import Vuex from 'vuex'
import { SOME_MUTATION } from './mutation-types'

const store = new Vuex.Store({
  state: { ... },
  mutations: {
    // we can use the ES2015 computed property name feature
    // to use a constant as the function name
    [SOME_MUTATION] (state) {
      // mutate state
    }
  }
})
```
상수를 쓸지말지는 대체로 기호라고 볼 수 있으므로 - 개발자가 많다면 유익하겠지만 - 니 맘대로 하자.


### Mutations Must Be Synchronous
꼭 기억해야 할 중요한 규칙은 mutation 핸들러 함수는 반드시 동기화되어야 한다는 것이다.
왜? 아래 예제를 살펴보자:
```
mutations: {
  someMutation (state) {
    api.callAsyncMethod(() => {
      state.count++
    })
  }
}
```

이제 비동기 코드를 가진 mutation 핸들러를 디버깅한다고 생각해보자.
모든 로그에는, 상태에 대한 before와 after 스냅샷이 있어야 한다. 그런데 비동기 콜백 호출은 이것을 불가능하게 만든다:
mutation이 호출됐을 때 콜백은 실행되지 않고, devtool에서는 실제로 언제 콜백이 실행될지 알 수도 없다.
게다가 콜백에서 수행된 state mutation은 추적할 수도 없다.

### Committing Mutations in Components
컴포넌트에서 mutation을 수행하고 싶으면 `this.$store.commit(type)` 혹은 mapMutations를 사용할 수 있다는 이야기.
-  root 컴포넌트에 store injection 필수

```
import { mapMutations } from 'vuex'

export default {
  // ...
  methods: {
    ...mapMutations([
      'increment', // map `this.increment()` to `this.$store.commit('increment')`

      // `mapMutations` also supports payloads:
      'incrementBy' // map `this.incrementBy(amount)` to `this.$store.commit('incrementBy', amount)`
    ]),
    ...mapMutations({
      add: 'increment' // map `this.add()` to `this.$store.commit('increment')`
    })
  }
}
```
### On to Actions
stats mutation이 비동기와 결합되면 원인을 파악하기 굉장히 어려우지는 경향이 있다.
예를 들어 상태를 변경하는 두 개의 비동기 콜백이 호출되면 어느 콜백이 먼저 실행되었는지 알기 어렵다.
이것이 mutation과 action을 분리한 이유이다.

Vuex에서는, mutations은 동기 트랜잭션이다:
```
store.commit('increment')
// any state change that the "increment" mutation may cause
// should be done at this moment.
```
비동기 연산을 위해서는, Actions을 보자.