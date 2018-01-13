---
layout: post
title:  "What Is Vuex?"
date:   2018-01-12 10:13:01
author: Mars
categories: javascript
---

### What is Vuex?
Vuex는 state management pattern + library로서, 앱의 상태를 예측 가능하게 변경해주는 중앙 집권화된 store 역할을 담당한다.
Vue의 공식 devtools extension 과도 통합되어 있어서 설정없이 time-travel debugging과 state snapshot export / import을 제공해준다.

### What is a "State Management Pattern"?
카운터 앱을 보자:

```
new Vue({
  // state
  data () {
    return {
      count: 0
    }
  },
  // view
  template: `
    <div>{{ count }}</div>
  `,
  // actions
  methods: {
    increment () {
      this.count++
    }
  }
})
```

이 앱은 아래의 모듈을 가지고 있다:
- state: 단일화된 상태 객체(the source of truth)를 가지고 있다;
- The view: state를 렌더링(mapping)한다.
- The actions: 사용자 행동(input)에 대한 반응으로, stats가 변경될 수 있다.

위의 예제는 아주 단순한 경우의 "one-way data flow" 개념을 보여주고 있다.

그렇지만 만약 여러개의 컴포넌트가 상태를 공유해야 한다면, 이런 단순성은 빠르게 무너질 수 있다:
- 여러개의 뷰가 동일한 상태 데이터에 의존한다.
- 다른 뷰들의 Actions 들이 동일한 상태 데이터를 조작해야 할 필요가 있다.

첫번째 경우에는, props를 이용한다면 깊게 심어져있는 컴포넌트를 처리할 때 고통이 따르고, 자식 노드가 아닌 이웃한 노트에는 전달할 방법이 없다.
두번째 경우에는, 참조를 설정하거나 이벤트를 전달해서 각 컴포넌트에서 상태를 가지도록 하지만 어떤 방식이든 무너지기 쉽고 빠르게 유지보수하기 어려운 코드가 된다.


So why don't we extract the shared state out of the components, and manage it in a global singleton? With this, our component tree becomes a big "view", and any component can access the state or trigger actions, no matter where they are in the tree!

In addition, by defining and separating the concepts involved in state management and enforcing certain rules, we also give our code more structure and maintainability.

This is the basic idea behind Vuex, inspired by Flux, Redux and The Elm Architecture. Unlike the other patterns, Vuex is also a library implementation tailored specifically for Vue.js to take advantage of its granular reactivity system for efficient updates.