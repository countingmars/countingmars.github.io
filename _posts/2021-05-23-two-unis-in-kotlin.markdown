---
layout: post
title:  "코틀린 타입시스템, 두개의 우주"
date:   2021-05-23 09:43:59
author: Mars
categories: episodes
---

  
일반적으로 우주 사이를 이동하기 위해서는 막대한 에너지를 사용하는 비프로스트를 이용하여 점프해야 합니다.

**그림: 비프로스트가 변수들을 다른 우주로 보내는 장면**
![그림: 비프로스트로 다른 우주로 이동하는 변수들](https://miro.medium.com/max/500/0*mm6Xj-kaBsJATUX9)

<br /><br />

# non-nullable과 nullable
코틀린은 non-nullable 우주와 nullable 우주 2개가 존재합니다. 다행히 엄청난 진화를 이루어낸 코틀린은 비프로스트를 이용하지 않고도 우주 사이를 빠르게 이동할 수 있습니다.

<br /><br />

**코드: ?를 사용하여 nullable 우주를 탈출하는 장면**
{% highlight kotlin %}


var nullableAge: Int? = null
nullableAge?.toChar()


{% endhighlight %}

그리고 nullable 우주에서 non-nullable 우주로 이동함으로써 null-safety라는 평화를 얻을 수 있습니다.



# super-type과 sub-type
자바의 최상위 타입은 Object 입니다. 코틀린의 최상위 타입은 Any? 입니다. 이 말의 의미는 Any 타입은 Any? 의 서브 타입이라는 의미입니다.

**그림: Any의 조상 Any?** 
<img src="https://miro.medium.com/max/129/0*4cfsgakE87aFmIQm" width="100px" style="width:100px">

그런데 이 관계는 스트링 타입에도 적용됩니다.

**그림: String의 조상 String?**
<img src="https://miro.medium.com/max/210/0*eK2eXZ2X_0-s7g_7" width="150px" style="width:150px">

이 규칙은 심지어 내가 만든 클래스에도 적용됩니다. 만약 내가 Cat 클래스를 만들었다면 자동으로 상위 타입은 Cat? 가 됩니다. 그리고 Any? 타입은 모든 것의 최상위 타입이므로 Cat? 역시도 Any? 타입의 하위 타입이 됩니다.
코틀린의 슈퍼 타입, 서브 타입 관계에서 기본적으로 non nullable 타입이 상위 타입이라는 인사이트를 얻을 수 있습니다. 더하여 전 우주의 최상위 타입은 Any? 타입임을 알 수 있습니다.

<br /><br />

# 어리석은 우주
코틀린 세계의 2개의 우주 중 하나는 어리석기 그지 없습니다. 무의 존재를 부정하는 우주, 바로 non-nullable 우주입니다.

<br /><br />


**코드: 무지한 변수 Age의 탄생 장면**
{% highlight kotlin %}


val age: Int = 1


{% endhighlight %}

<br />
<br />

# 타노스
nullable 우주에서의 변수는 null 혹은 non-null 둘 중 하나의 상태를 가집니다.
한편 타노스(본명 토니 호아)는 null 상태의 변수들을 한번의 핑거스냅(null pointer exception)으로 사망에 이르게 합니다. 최근 나이를 먹고 고향으로 돌아간 타노스는 null 이라는 개념을 만든 자신의 과오를 뉘우치듯한 발언(The billion dollar mistake)을 하기도 했습니다. 아무래도 어벤저스의 역습이 무서운가 봅니다.

<br />
<br />

# 어벤저스 Any
하지만 어리석은 우주 non-nullable 에는 Any라는 어벤저스(최상위 타입)가 존재합니다. Any 타입은 모든 변수들을 null로부터 보호해주는 null-safety 보호자입니다. 그리고 Any의 하위 타입들은, 자신들이 Any?의 하위 타입이라는 사실도 모르고 있습니다.
<br />

**그림: 타노스의 핑거스냅(npe)을 막고 있는 Any 타입들**
<img src="https://miro.medium.com/max/664/0*V-Yzh_-bB4FlmY45" width="200" style="width:200px">

<br />
<br />  
   
과연 이들의 어리석음(Void 혹은 Nothing)은 끝일까요?






