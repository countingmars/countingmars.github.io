---
layout: post
title:  "Imperative To Reactive #1"
date:   2018-08-15 11:25:01
author: Mars
categories: java
---

추천 서적: Hands on reactive programming in Spring 5 



 
 
 
### Why Reactive?
쓰레드 하나가 작업을 처리하는데 250ms가 걸린다고 가정한다면 1초당 4개의 작업을 처리할 수 있다. 
만약 시스템의 총 쓰레드수가 500개라면 1초당 2000개의 작업을 처리해낼 수 있다.


사실 1초당 2000개씩 처리해낸다면 1분에 12만개를 처리해내기 때문에 나쁘지 않은 시스템이다.
좀더 계산해본다면 이러한 서버 10대가 존재한다면, 1시간에 5천만개 이상의 요청을 처리해낼 수 있기 때문이다. 


그런데 만약 특별한 날에 우리의 상상 이상의 트레픽이 발생한다면 어떻게 될까?
예를 들어 블랙프라이데이 같은 ...


유저는 끝없이 몰려들고 시스템은 한계를 시험받게 된다.
이러한 문제를 이미 경험한 기업들의 의견을 들어보자.

- https://www.cnet.com/news/amazon-com-hit-with-outages/ 
- https://www.forbes.com/sites/kellyclay/2013/08/19/amazon-com-goes-down-loses-66240-per-minute/#3fd8db37495c 
- https://techcrunch.com/2011/11/25/walmart-black-friday/ 


  
  
### Reactivity: Fundamental principles to build a robust system
Reactivity는 사실 견고한 시스템을 만들기 위한 근본적인 원칙을 내포한 단어이다.
시스템이 반응성(Reactivity)있다는 말의 의미는 변화에 긍정적으로 대응할 수 있다는 의미이기도 하다.


즉, 내부 모듈의 실패를 고립(isolation)시켜서 장애가 전파되지 않게 해야하고, 
각 모듈들의 독립성(independence)을 보장해서 모듈의 내부 변화가 자유롭게 이루어져야 한다.


예를 들어, 주문 시스템은 항상 고객의 주문을 받을 수 있어야 하고, 만약 결제 시스템에 장애(outage)가 발생했다 하더라도 주문은 가능해야 한다.
그리고 주문 시스템의 장애가 해결했을 때, 자동으로 재시도(auto-retry) 되어야 한다. 


### Message Driven Communication
이러한 Reactivity의 핵심은 메시징이라고 할 수 있다. 메시징 시스템은 메시지를 보낼 뿐이지 상대가 올바로 메시지를 처리했는지는 관심이 없다. 따라서 메시지를 처리해야 할 상대의 문제로부터 독립성을 유지할 수 있다. 또한 메시지를 처리해야 하는 모듈은 자신이 정상적인 상황일 때, 메시지 큐에서 필요한 만큼의 메시지를 가져와서 처리하면 된다. 이러한 시스템의 대표적인 사례는 메일 박스를 예로 들 수 있다. 
모바일 클라이언트에서의 메일 수신 실패가 메시지의 수신 실패 혹은 누락 혹은 삭제를 의미하지 않는다(실패에 강하다). 언제든 다양한 클라이언트 모듈을 이용해서 메일을 확인할 수 있다. 


이러한 메시징 커뮤니케이션의 핵심은 비동기 넌-블러킹(async non-blocking) 모델, 즉 메시지를 보내고 그 응답을 기다리지 않는 방식이라고 할 수 있다. 
- Fire And Forget 모델


반대로 메시징 커뮤니케이션을 하지 않는 방식은 블러킹(blocking) 모델이며 예를 들어 스프링의 `RestTemplate`으로 요청을 보내는 경우를 생각해볼 수 있다. 
이 방식은 I/O 작업을 위해 쓰레드를 블러킹시키므로(Being blocked), 굉장히 비효율적이라고 볼 수 있다. 
   

### Reactivity In Spring 4
Reactive System 이라면 메시지를 보낸 후에 그 응답을 기다리는 것(being blocked)이 아니라 자신이 해야 할 일을 지속할 수 있어야 하고, 
응답이 발생되었을 때(즉 변화가 발생했을 때), 그에 반응할 수 있어야 한다.


자바에서 블러킹 모델에 좀더 Reactivity 를 부여하기 위한 방법은 크게 2가지가 있다.
- Thread + Callback: 쓰레드를 추가로 사용해서 요청을 위임하고, 응답이 발생했을 때 처리해야 할 로직을 콜백으로 전달한다. 문제는 Callback Hell을 만들고, 쓰레드의 비효율적인 활용은 덤이다.
- `Future` or `AsyncRestTemplate`: Callback Hell은 해결하겠지만 여전히 비효율적인 멀티쓰레드의 활용 문제가 남아있다. 



### Do Not Pull, Push To Follow The Reactive Design
Reactive System 이라면 변화에 반응할 수 있어야 한다. 
이 말의 의미는 변화가 발생했을 때 즉시 알아챌 수 있어야 한다는 의미이다.

이러한 시스템의 변화에 반응하는 시스템이라면 시스템의 변화를 검사하기 위해 요청하기(pull)보다는, 변화를 감지한 곳에서 즉시 변화를 전파(push)해줘야 한다.

즉 변화를 감지하기 위해 감시(observe)하거나 변화를 구독(subscribe)해야 한다.


- 변화가 존재하는지 검사하기 위해 요청(pull)하는 방식은 불필요한 네트웍 왕복 시간을 낭비하게 되며, 상대가 온전히 데이터를 받을 때까지 기다려주기도 해야 한다.

### Observer Pattern 
옵저버 패턴은 변화를 감지하기 위한 좋은 방식이다. 
이벤트가 발생했을 때, 이 이벤트의 관찰자들에게 통지(notify)해주기 때문이다.

또한 이벤트를 발생시키는 Subject와 이벤트를 소비하는 Observer는 정의된 인터페이스를 상속하고 구현하기 때문에, 서로에 대한 의존성이 낮다고 볼 수 있다. 
- Decoupling: Isolation and Independence
```
interface Subject {
	void notify(String event);
}
interface Observer {
	void observe(String event);
}

Subject subject = new Subject() {
	public void register(Observer o) { this.observers.add(o); }
	@Override public void notify(String event) { observers.observe(event); }
};
subject.register(new Observer() { 
	@Override public void observe(String event) { 
		println('got an event'); 
	} 
});

subject.notify('first event');
```

그런데 만약 하나의 Subject에 여러 옵져버가 등록되었고, 하나의 옵져버가 너무 느리다면 Thread를 활용해서 전체 옵져버들에게 이벤트를 전파하는 속도를 떨어뜨리지 않을 수 수 있다.
Thread를 직접쓰기 보다는 ExecuterService가 편하다.
```
observers.forEach(o -> executerService.submit(() -> o.observe(event)) }

``` 

물론 여전히 많은 쓰레드를 써야하면서 발생할 수 있는 문제들이 남아있으며, 그 외에도 옵져버들 입장에서는 언제 자신이 메모리에서 사라져도 괜찮은지 스스로 판단할 수 있는 기준이 없다. 
예를 들어 더이상 자신에게 전달될 이벤트가 없다는 것을 알 수 있다면 굳이 메모리를 차지할 필요가 없을테니까. 즉 이벤트가 더 이상 발생하지 않게 된 시점에 옵져버 스스로 이에 반응할 수 있으면 좋겠다.   


### Publisher-Subscriber
내가 만약 메시지를 발생시키는 모듈을 구현해야 하는데 매 구현마다 비동기를 위해 ExecutorService 를 활용해야 하는 모양을 생각해보면 아주 번거롭다.

Pub-Sub 패턴은 중간에 메시지 브로커(Event Channel or Message Broker or Event Bus)를 추가해서 메시지를 수신하는 모듈이 존재한다는 점에서 Observer Pattern 과 차이가 있다. 


메시지 브로커를 통해서 메시지를 라우팅하거나 필터링 할 수 있다. 
또한 Observer Pattern 에서 모호했던 비동기 처리의 책임을 메시지 브로커에게 맡길 수도 있겠다.

   
### Observer + Iterator = RxObserver
Iterator Pattern 은 메시지를 계속해서 발생시킬 수 있는 메서드(next)와, 메시지의 끝을 알리는 메서드(hasNext)를 제공한다.
```
interface Iterator<T> {
	T next();
	boolean hasNext();
} 
``` 
Observer Pattern은 메시지를 Push할 수 있는 특성(notify)과 메시지를 수신할 수 있는 특성(register)을 가지고 있다.
```
interface Subject {
	void notify(String event);
	void register(Observer o);
	void unregister(Observer o);
}
``` 

이러한 두 패턴의 특징을 모두 가지게 되면 그 모양이 RxObserver의 형태와 유사하다.
```
interface RxObserver {
	void onNext(T next);
	void onComplete();
	void onError(Throwable t);
}  
```
메시지의 끝을 알리는 onComplete()과 시스템의 장애를 알리는 onError()가 추가되어 시스템의 변화를 전파할 수 있는 형태를 완성했다.



insert coin to be continued
