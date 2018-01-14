---
layout: post
title:  "Vue: Vuex Core Concepts - 2. Getters"
date:   2018-01-13 09:43:59
author: Mars
categories: javascript
---

### Getters
https://vuex.vuejs.org/en/getters.html

가끔씩, state를 그대로 쓰지 못하는 경우도 있다, 예를 들어 할일 목록을 완료된 것만 필터링해서 숫자를 계산해야 할 수도 있다:
```
computed: {
  doneTodosCount () {
    return this.$store.state.todos.filter(todo => todo.done).length
  }
}
```
만약에 이런 코드를 여러 컴포넌트에서 써야 한다면, 함수를 복제하거나, 공유되는 helper를 만들어서 import해야 할텐데 어느 방식이든 이상적이진 않다.

Vuex에 "getters"를 정의할 수 있는데, store에서 제공하는 일종의 computed property라고 생각하면 된다. Computed property랑 비슷하기 때문에 결과는 캐싱되고, 필요한 시점에 재계산된다.

Getters의 첫번째 파라미터는 state 객체다:
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

Getters는 `store.getters`로 접근할 수 있다:
```
store.getters.doneTodos // -> [{ id: 1, text: '...', done: true }]
```

Getters는 두번째 파라미터로 다른 getters를 전달받는다:
- 하나의 getter 메서드가 자신이 포함된 getters를 전달받는다는 의미인듯.
```
getters: {
  // ...
  doneTodosCount: (state, getters) => {
    return getters.doneTodos.length
  }
}
store.getters.doneTodosCount // -> 1
```

이제 컴포넌트에서 가져다 쓰기만 하면 된다:
```
computed: {
  doneTodosCount () {
    return this.$store.getters.doneTodosCount
  }
}
```

getters에 파라미터를 전달하고 싶다면 getters가 함수를 반환하게 만들어야 한다.
이 방식은 store에 있는 배열을 쿼리하고 싶을 때 특히 유용하다:
- ES6의 화살표 함수를 두 번 활용했다. ex) `let add3And = ((x) => (y) => x + y)(3); let five = add3And(2);`
```
getters: {
  // ...
  getTodoById: (state) => (id) => {
    return state.todos.find(todo => todo.id === id)
  }
}
store.getters.getTodoById(2) // -> { id: 2, text: '...', done: false }
```

### The mapGetters Helper
mapGetters는 일종의 helper로써 store getters를 local computed property에 매핑해준다:
```
import { mapGetters } from 'vuex'

export default {
  // ...
  computed: {
    // mix the getters into computed with object spread operator
    ...mapGetters([
      'doneTodosCount',
      'anotherGetter',
      // ...
    ])
  }
}
```
만약 로컬 컴포넌트에서 getters의 이름을 바꾸고 싶다면 오브젝트를 사용해 이름을 지정해주자:
```
...mapGetters({
  // map `this.doneCount` to `store.getters.doneTodosCount`
  doneCount: 'doneTodosCount'
})
```