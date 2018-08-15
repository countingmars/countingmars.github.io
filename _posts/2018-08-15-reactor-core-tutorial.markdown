---
layout: post
title:  "Reactor Core Tutorial"
date:   2018-08-15 11:25:01
author: Mars
categories: java
---

원본 <http://sinhamohit.com/writing/reactor-core-tutorial>

### A Reactor Core Tutorial
Reactive Programming은 쉽게 확장(scale)할 수 있는 비동기, 넌 블러킹, 이벤트 드리븐 앱을 만드는 것과 관련된 기술이다. 

Reactor는 Reactive 라이브러리로써 넌 블러킹 앱을 만들기 위한 것으로 Reactive Streams 명세를 기반으로 했다. 이 라이브러리는 자바9에 통합되었다.

Reactive Streams는 push 기반이며, Publisher(이벤트를 발행하는 스트림)에 새로운 데이터가 생길 때 마다 Subscriber에게 알린다. 이러한 push 성향(aspect)이 reactive가 되는 비결이다.

##### Dependencies
이 튜토리얼에서는 reactor-core와 reactor-test를 사용한다.
```
plugins {
    id "io.spring.dependency-management" version "1.0.3.RELEASE"
}
...
dependencyManagement {
    imports {
        mavenBom "io.projectreactor:reactor-bom:Aluminium-SR1"
    }
}
...
dependencies {
    compile 'io.projectreactor:reactor-core'
    testCompile 'io.projectreactor.addons:reactor-test'
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```

### Publishers (Mono and Flux)
Mono와 Flux는 Publisher 인터페이스를 구현한 구현체이다. Flux는 0개 ~ N개의 아이템을 관찰하며(observe, push 이벤트 발생횟수), Mono는 0개 ~ 1개의 아이템을 관찰한다. 최대 0개의 아이템을 나타내는(hinting) `Mono<Void>`도 있다.

테스트 코드로 이 라이브러리를 어떻게 쓰는지 확인해보자.
```
@Test
public void empty() {
    Mono<String> emptyMono = Mono.empty();
    StepVerifier.create(emptyMono).verifyComplete();
    Flux<String> emptyFlux = Flux.empty();
    StepVerifier.create(emptyFlux).verifyComplete();
}
```
`Mono.empty()`로 비어있는 Mono를 생성했다. 마찬가지로 `Flux.empty()`로 비어있는 Flux를 생성했으며, 이를 확인하기 위해 StepVerifier를 사용했다. Publisher인 emptyMono 객체와 emptyFlux 객체는 어떠한 객체도 방출(emit)하지 않고 완료되었음을 StepVerifier로 검증했다.


```
@Test
public void initialized() {
    Mono<String> nonEmptyMono = Mono.just("Joel");
    StepVerifier.create(nonEmptyMono).expectNext("Joel").verifyComplete();
    Flux<String> nonEmptyFlux = Flux.just("John", "Mike", "Sarah");
    StepVerifier.create(nonEmptyFlux).expectNext("John", "Mike", "Sarah").verifyComplete();
    Flux<String> fluxFromIterable = Flux.fromIterable(Arrays.asList("Tom", "Hardy", "Bane"));
    StepVerifier.create(fluxFromIterable).expectNext("Tom", "Hardy", "Bane").verifyComplete();
}
```
이 예제에서는 Mono와 Flux를 다르게 초기화했다. 
문자열을 0 ~ 1회 방출하는 스트림 Mono<String> 타입의 객체 nonEmptyMono를 생성했으며, 
"Joel"이라는 문자열을 방출하도록 했다.  
이후 StepVerifier로 Mono 객체가 생성되었음과 "Joel" 문자열이 방출되었음을 검증했다.

Flux<String> 타입의 객체 nonEmptyFlux를 생성하여 "John", "Mike", "Sarah"를 방출하도록 했다.
StepVerifier로 순서대로 문자열이 방출하는 것을 검사했다.

마지막으로 Array를 이용하여 Flux를 생성하도록 했으며, StepVerifier로 오브젝트 방출 순서를 검증했다. 

```
@Test
public void operations() {
    Mono<String> monoMap = Mono.just("James").map(s -> s.toLowerCase());
    StepVerifier.create(monoMap).expectNext("james").verifyComplete();

    Flux<String> fluxMapFilter = Flux.just("Joel", "Kyle")
            .filter(s -> s.toUpperCase().startsWith("K"))
            .map(s -> s.toLowerCase());
    StepVerifier.create(fluxMapFilter).expectNext("kyle").verifyComplete();
}
```

### Java8 Stream With Mono And Flux
자바8 스트림 연산자를 Mono와 Flux에 사용해서 하나의 Mono를 다른 Mono로 맵핑해보자. 
```
Mono<String> monoMap = Mono.just("James").map(s -> s.toLowerCase());
```
위 코드는 문자열을 방출하는 Mono를 소문자 문자열을 방출하는 Mono로 맵핑하는 예제이다.
StepVerifier로 `monoMap`이 생성되었음을 검증하고, 이후에 마지막 Mono가 "james"가 방출하였음을 검증하고 있다.

-> 역주: 모든 Publisher(Mono와 Flux)는 생성된 이후에 누군가가 Subscribe을 시작해야만 동작하기 시작한다. 
따라서 StepVerifier는 Publisher를 Subscribe함으로써 Publisher가 동작하도록 만들고, 그 동작을 검사(verify)한다. 

다음은 `Flux<String>` 타입의 객체를 생성하고 "Joel"과 "Kyle"을 방출하도록 설정했으며, 
`filter()` 메서드를 통해 대문자 "K"로 시작하는 문자열만 통과시킨 후 
`map()` 메서드를 통해 소문자 문자열을 방출하는 Flux로 맵핑시켰다. 

-> 역주: 자바8 스트림 연산자 `filter()`, `map()` 등등을 사용할 때, 
Publisher가 방출하는 객체를 조작할 수 있지만, 그 결과는 여젼히 Publisher 타입임을 숙지하자.

아래 코드에서 `map()` 메서드의 람다에서 문자열 s를 대문자로 맵핑했지만 반환값은 `Mono<String>`임을 다시 확인해보자.
```
Mono<String> result = Mono.just("a").map(s -> s.toUpperCase());

```

### Combining Fluxes
-> 역주: 앞으로 소개할 `Flux.zip()`, `Flux,merge()`, `Flux.concat()`은 여러 Flux를 조합할 때 사용하는 연산들이다.


```
@Test
public void zipping() {
    Flux<String> titles = Flux.just("Mr.", "Mrs.");
    Flux<String> firstNames = Flux.just("John", "Jane");
    Flux<String> lastNames = Flux.just("Doe", "Blake");

    Flux<String> names = Flux
            .zip(titles, firstNames, lastNames)
            .map(t -> t.getT1() + " " + t.getT2() + " " + t.getT3());

    StepVerifier.create(names).expectNext("Mr. John Doe", "Mrs. Jane Blake").verifyComplete();

    Flux<Long> delay = Flux.interval(Duration.ofMillis(5));
    Flux<String> firstNamesWithDelay = firstNames.zipWith(delay, (s, l) -> s);

    Flux<String> namesWithDelay = Flux
            .zip(titles, firstNamesWithDelay, lastNames)
            .map(t -> t.getT1() + " " + t.getT2() + " " + t.getT3());

    StepVerifier.create(namesWithDelay).expectNext("Mr. John Doe", "Mrs. Jane Blake").verifyComplete();
}
```

`Flux.zip`은 여러 Flux 객체의 방출 순서를 맞춰준다(combine). 
즉 결합된 모든 Flux가 n번째 아이템을 방출한 시점을 (가장 늦게 방출하는 Flux의 속도에 맞춰) 일치시켜준다. 
이 예제에서는 호칭(mr, mrs) Flux, firstName Flux, lastName Flux를 `zip()`하고, 
이를 조합하여 full name을  방출하는 Flux를 생성했다.
아래 코드를 다시 감상해봐라.
```
Flux<String> names = Flux
            .zip(titles, firstNames, lastNames)
            .map(t -> t.getT1() + " " + t.getT2() + " " + t.getT3());
```

`Flux.zip()`을 사용해서 여러 Flux의 이벤트 방출 타이밍을 맞출 수 있음을 확인했다. 이제 느리게(5ms) 이벤트를 방출하는 딜레이 Flux를 사용해서, `Flux.zip()` 메서드가 진짜로 순서를 맞춰줌을 다시 확인해보자.
`Flux.interval(Duration.ofMillis(5))`로 딜레이를 만드는 Flux를 생성했다. 
`firstNames.zipWith(delay, (s, l) -> s)`로 두 개의 Flux의 타이밍을 맞췄고, 람다 표현식으로는 firstNames에서 방출한 객체를 반환하도록 하여, 최종적으로 firstName을 방출하는 Flux를 생성했다.
딜레이를 주는 Flux가 추가되었지만 여전히 여러 Flux의 순서가 잘 결합됨을 확인할 수 있다.
```
@Test
public void interleave() {
 Flux<Long> delay = Flux.interval(Duration.ofMillis(5));
 Flux<String> alphabetsWithDelay = Flux.just("A", "B").zipWith(delay, (s, l) -> s);
 Flux<String> alphabetsWithoutDelay = Flux.just("C", "D");

 Flux<String> interleavedFlux = alphabetsWithDelay.mergeWith(alphabetsWithoutDelay);
 StepVerifier.create(interleavedFlux).expectNext("C", "D", "A", "B").verifyComplete();

 Flux<String> nonInterleavedFlux = alphabetsWithDelay.concatWith(alphabetsWithoutDelay);
 StepVerifier.create(nonInterleavedFlux).expectNext("A", "B", "C", "D").verifyComplete();
}
```

*Interleaving*은 성능 향상을 목적으로 비-순차적으로 데이터가 쓰여지는 개념이다. 여러 Flux가 조합되었을 때, 개별 Flux의 방출(emit)을 독자적으로 수행할 수 있도록 해준다. 

`Flux.mergeWith()`는 두 개의 Flux를 독립성을 유지한 채로(interleaved sequence)병합해준다.

-> 역주: , interleaver가 무슨 의미인지 와닫지 않아서 상호간에(inter) 떨어진채로(leave)라는 의미로 받아들였다.

따라서 딜레이가 추가된 Flux.just("A", "B")와 Flux.just("C", "D")를 병합(mergeWith)하면 객체 방출 순서가 C, D, A, B 순이 된다.

`Flux.concatWith`는 non interleaved sequence를 보장해주는 병합이므로, 딜레이가 추가된 Flux.just("A", "B")와 Flux.just("C", "D")를 병합할 경우 순서대로 방출되므로 최종 결과는 A, B, C, D가 된다.


### Blocking Publisher
마지막으로 Publisher가 객체를 방출할 때 까지 무한히 대기(block)하는 방법을 알아보자. 
`Mono.block()`을 호출하거나 `Flux.toIterable().iterator().next()`를 호출하는 것이다. 
```
@Test
public void block() {
 String name = Mono.just("Jesse").block();
 assertEquals("Jesse", name);

 Iterator<String> namesIterator = Flux.just("Tom", "Peter").toIterable().iterator();
 assertEquals("Tom", namesIterator.next());
 assertEquals("Peter", namesIterator.next());
 assertFalse(namesIterator.hasNext());
```