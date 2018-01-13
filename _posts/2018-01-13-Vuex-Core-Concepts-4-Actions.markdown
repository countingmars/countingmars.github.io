---
layout: post
title:  "Vue: Vuex Core Concepts - 4. Actions"
date:   2018-01-13 11:43:59
author: Mars
categories: javascript
---

### Actions
Actions는 mutations랑 비슷한데, 차이는 이렇다:
- mutations가 상태(state)를 변경한다면, actions은 mutations을 호출(commit)한다.
- Actions은 임의의 비동의 연산을 포함할 수 있다.

간단한 Action을 등록해보자:
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
Action 핸들러는 context 객체를 전달받는데, store 객체와 동일한 methods와 properties를 가지고 있기 때문에
mutation을 호출하기 위해 `context.commit`을 사용하거나 context.state, context.getters를 쓸 수도 있다.
* 다음에 볼 Modules 챕터에서 왜 context가 store instance가 아닌지 살펴볼거다.

실전에서는, 종종 코드를 단순화시키기 위해서 ES2015 argument destructuring을 사용할거다. (특히 commit을 여러번 호출해야 할 때):
* 꿀팁이군.

```
actions: {
  increment ({ commit }) {
    commit('increment')
  }
}
```

### Dispatching Actions
Actions은 store.dispatch 메서드로 트리거된다:

```
store.dispatch('increment')
```

이걸 본 첫 느낌은 멍청하게 보일꺼다: store.commit('increase') 호출하면 되는데 ... 그런데 mutation은 동기화여야 한다는 걸 기억해보자. Actions은 그럴 필요가 없다. 그러니 우리는 action에서 비동기 코드를 작성할 수 있다:

```
actions: {
  incrementAsync ({ commit }) {
    setTimeout(() => {
      commit('increment')
    }, 1000)
  }
}
```

Actions은 mutations와 동일한 payload 형태를 지원하고 object-style dispatch도 지원하므로:

```
// dispatch with a payload
store.dispatch('incrementAsync', {
  amount: 10
})

// dispatch with an object
store.dispatch({
  type: 'incrementAsync',
  amount: 10
})
```

좀더 실전적인 예제는 쇼핑 카트의 상품을 계산(check out)하는 Action이다. 이 동작은 비동기 API를 호출하고, 다수의 mutation을 수행한다:

```
actions: {
  checkout ({ commit, state }, products) {
    // save the items currently in the cart
    const savedCartItems = [...state.cart.added]
    // send out checkout request, and optimistically
    // clear the cart
    commit(types.CHECKOUT_REQUEST)
    // the shop API accepts a success callback and a failure callback
    shop.buyProducts(
      products,
      // handle success
      () => commit(types.CHECKOUT_SUCCESS),
      // handle failure
      () => commit(types.CHECKOUT_FAILURE, savedCartItems)
    )
  }
}
```

우리가 비동기 플로우를 수행 중이라는 것과 Action의 결과(side effect)를 (commit를 호출해서) 기록하고 있다는 것에 주목하자.

### Dispatching Actions in Components
컴포넌트에서 액션을 호출하는 방법은 `this.$store.dispatch('xxx')` 하거나, 혹은 mapActions helper 사용하는거다:
* requires root store injection

```
import { mapActions } from 'vuex'

export default {
  // ...
  methods: {
    ...mapActions([
      'increment', // map `this.increment()` to `this.$store.dispatch('increment')`

      // `mapActions` also supports payloads:
      'incrementBy' // map `this.incrementBy(amount)` to `this.$store.dispatch('incrementBy', amount)`
    ]),
    ...mapActions({
      add: 'increment' // map `this.add()` to `this.$store.dispatch('increment')`
    })
  }
}
```

### Composing Actions
Actions 은 종종 비동기이니까, 액션이 끝났다는 걸 언제 알 수 있을까?
더 중요한건, 여러개의 Action을 조합하는 복잡한 비동기 플로우를 어떻게 제어할 수 있을까?

가장 우선적으로 알아야 할 사항은 store.dispatch는 action 핸들러가 반환한 Promise를 처리할 수 있으며, Promise를 반환해주기까지 한다:
```
actions: {
  actionA ({ commit }) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        commit('someMutation')
        resolve()
      }, 1000)
    })
  }
}
```

이제 이렇게 할 수 있다:
```
store.dispatch('actionA').then(() => {
  // ...
})
```
이제 또 다른 액션에서 이렇게 할 수 있다:
```
actions: {
  // ...
  actionB ({ dispatch, commit }) {
    return dispatch('actionA').then(() => {
      commit('someOtherMutation')
    })
  }
}
```
마지막으로, async / await 를 사용해서, action을 이런식으로 만들 수 있다:

```
// assuming `getData()` and `getOtherData()` return Promises

actions: {
  async actionA ({ commit }) {
    commit('gotData', await getData())
  },
  async actionB ({ dispatch, commit }) {
    await dispatch('actionA') // wait for `actionA` to finish
    commit('gotOtherData', await getOtherData())
  }
}
```
* store.dispatch가 서로 다른 컴포넌트에 있는 여러개의 action handlers를 트리거 할 수 있고,
이러한 경우에 반환된 값은 모든 핸들러에서 resolve된 후에 resolve되는 Promise 객체일꺼다.