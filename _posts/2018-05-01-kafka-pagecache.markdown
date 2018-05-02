---
layout: post
title:  "Kafka Design Filesystem"
date:   2018-04-24 13:25:01
author: Mars
categories: tools
---

원본 <https://kafka.apache.org/documentation/#design_filesystem>


### Persistence
**Don't fear the filesystem!**

Kafka가 메시지를 저장하고 캐싱할 때 파일시스템에 크게 의존한다.
"disks are slow"라는 통념 때문에 Kafka의 성능을 회의적으로 바라보겠지만, 어떻게 사용하느냐에 따라 (기대보다) 더 느려질 수도 있고 네트웍 만큼 빨라질 수도 있다. 
​
디스크 성능은 하드 드라이브의 disk seek 대기시간(latency)에 의해 결정되는데 
linear write의 성능(600MB/sec)은 random write의 성능(100k/sec)보다 6000배 이상 빠를 수 있다. 
(JBOD with six 7200rpm SATA RAID-5 기준)  


그리고 이러한 linear access는 대부분 예측가능하기에  
read-ahead(prefetch large block multiples), write-behind(group smaller writes into large physical writes) 기술을 통해 OS에 의해 최적화된다.

![디스크와 메모리 Access 성능비교](https://deliveryimages.acm.org/10.1145/1570000/1563874/jacobs3.jpg )

위 이미지는 순차 디스크 접근(sequential disk access)이 랜덤 메모리 접근(random memory access)보다 경우에 따라서 더 빠를 수 있다는 것을 설명한다.

​
​또한 이러한 성능 편차를 극복하기 위해, 최신 OS들은 disk caching에 메인 메모리를 적극적으로 활용하게 되었다.
최신 OS는 모든 free 메모리를 디스크 캐싱에 투입할 것이다. (메모리 반환 시에 성능저하도 거의 없다.) 
모든 디스크 사용은 이 단일화된(unified) 캐쉬를 거치게 될 것이다.


게다가 JVM을 사용한다면 메모리 오버헤드는 실제 데이터 사이즈의 두배 이상이 될 수도 있는데다가 
가비지 콜렉션 비용 또한 메모리 소비 비용에 따라 증가할 것이다.

​
따라서 pagecache에 기반한 파일 시스템을 사용하는 것이 in-memory 캐쉬를 사용하는 것보다 월등히 유리할 것이다. 
심지어 캐쉬의 사이즈는 32GB 장비에서 28-30GB까지 사용할 수 있고, 
서비스가 재시작하더라도 사용할 수 있는 상태로 남아있을 뿐만 아니라
(in-process cache는 10GB 캐쉬 생성하는데 아마도 10분이 걸리겠지만), 
코드 레벨에서 캐쉬와 파일시스템 사이의 일관성을 유지할 필요가 없다. 
만약 디스크를 linear reads로 쓴다면 read-ahead는 이 캐쉬를 미리 준비해놓을 것이다!

​
따라서 모든 데이터는 파일시스템의 영속적인 로그로 쓰여지고(디스크 flush 없이), 
사실상 이것이 의미하는 바는 데이터가 커널의 pagecache에 전송되었다는 것이다.


이러한 pagecache-centric 설계 스타일은 여기에 설명되어 있다: 
http://varnish-cache.org/docs/trunk/phk/notes.html  
