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

이 앱은 다음의 부분(Part)을 가지고 있다:
- State: 단일화된 상태 객체(the source of truth)를 가지고 있다.
- View: state를 렌더링(mapping)한다.
- Actions: 사용자 행동(input)에 대한 반응으로, stats가 변경될 수 있다.

위의 예제는 아주 단순한 경우의 "one-way data flow" 개념을 보여주고 있다.

그렇지만 만약 여러개의 컴포넌트가 상태를 공유해야 한다면, 이런 단순성은 빠르게 무너질 수 있다:
- 여러개의 뷰가 동일한 상태 데이터에 의존한다.
- 다른 뷰들의 Actions 들이 동일한 상태 데이터를 조작해야 할 필요가 있다.

첫번째 경우에는, props를 이용한다면 깊게 심어져있는 컴포넌트를 처리할 때 고통이 따르고, 자식 노드가 아닌 이웃한 노트에는 전달할 방법이 없다.
두번째 경우에는, 참조 관계를 설정해서 해결하거나 이벤트를 전달하는 방식으로 각 컴포넌트가 복제된 상태값을 가지도록 해서 처리할 수 있지만 어떤 방식이든 깨지기 쉽고, 금방 유지보수하기 어려운 코드가 된다.


그렇다면 왜 공유되는 상태를 컴포넌트 밖으로 꺼내서 글로벌 싱글톤으로 관리하지 않을까?
이렇게 함으로써, 우리의 컴포넌트 트리는 하나의 큰 "view"가 되고, 어떤 컴포넌트든 어디에 있든 상관없이, 상태에 접근하거나 actions를 트리거 할 수 있다!

추가로, 상태 관리에 내포된 개념을 정의하고 분리하고 특정한 룰을 강제함으로써, 코드를 더욱 구조화하고 유지보수하기 쉽게 만들 수 있다.

이것이 Vuex에 숨겨진 아이디어로써 Flux, Redux, The Elm Architecture에서 영감을 얻은 것이다.
다른 패턴과는 다르게, Vuex는 Vuejs에 특화되어 재단된 라이브러리 구현체로써, 효율적으로 (View를) 업데이트하기 위한 세밀한 reactivity 시스템의 혜택을 가진다.


### When Should I Use It?
Vuex가 공유된 상태 관리를 돕기는 하지만, 복잡한 개념이고 추가적인 코드로 인해 비용을 지불해야만 한다.
이것은 단기 생산성과 장기 생산성 사이의 트레이드 오프이다.

만약에 거대한 SPA를 만드는게 아닌데 Vuex를 사용한다면, 거추장스럽고 기운 빠지게 할거다.
그런게 일반적인거고 앱이 심플하면 Vuex 없이도 충분할거다. 단순한 글로벌 이벤트 버스로 충분할거다.
그런데 만약에 medium-to-large-scale SPA를 만든다면, 어려운 상황에 처한 결과, 상태를 어떻게 잘 제어할지 생각하게 만들 것이고, 자연스런 다음 단계는 Vuex가 될 것이다.

Redux의 저자인 Dan Abramov로부터 멋진 인용구가 있다:
* Flux libraries are like glasses: you’ll know when you need them.
