---
layout: post
cover: false
title: Java Exception Processing
date:   2017-05-21 10:00:00
tags: fiction
subclass: 'post tag-fiction'
categories: 'java'
---

자바의 예외처리에 관해 이야기 해볼까 합니다. 자바의 예외처리에 대해서 헤드퍼스트는 다음과 같이 기술하고 있습니다. 


### 종종 예상치 못한 일이 일어나곤 합니다. 있는 줄 알았던 파일이 없거나 서버가 다운되는 경우도 흔히 있습니다.

제가 사용하고 개발하는 시스템도 수많은 예외처리를 사용하고 있습니다. 만약 예외처리를 사용하지 않는다면 어떻게 될까요? 위험요소가 있는 많은 메서드들에서 사용자들의 다양한 입력에 따라 발생하는 오류 때문에 시스템이 다운되는 일이 벌어질 수도 있습니다.

그래서 예외처리가 필요한 상황은 보통 이렇습니다.

1. 개발자가 메서드를 작성합니다.
2. 다른 누군가가 만든 클래스에 들어있는 메서드를 호출합니다.
3. 그 메서드에는 제대로 실행이 되지 않을 수도 있는 뭔가 위험한 작업이 있습니다.
4. 호출하려는 메서드에 위험요소가 있다는 것을 파악합니다.
5. 위험하다는 것을 파악하고나서는 실패상황을 처리할 수 있는 코드를 만듭니다.

그렇다면 실패상황이 발생한 것을 코드 내에서 어떻게 알 수 있을까요? 자바의 예외처리(exception handling) 메커니즘은 실행중에 생긴 예외상황을 처리할 수 있는 깔끔하고도 부담없는 방법입니다. 특정 메서드를 호출할 때 예외가 발생할 수 있다는 것을 알고 있다면 그러한 예외를 발생시킨 문제에 미리 대비할 수 있습니다. 그렇다면 자바의 예외처리 메커니즘이 어떤 방식으로 동작하는지 알아보겠습니다.


### 1. Exception 객체
- - -
Exception은 예외상황을 담고있는 객체입니다. Exception 유형의 객체입니다. 그렇기 때문에 우리가 잡아야 하는것도 객체입니다. 자바에서 이 Exception 객체를 처리하는 메커니즘은 다음과 같이 정형화 되어있습니다.

1. 위험요소를 가지고 있는 메서드에서 위험상황이 발생하면 예외객체를 생성해서 해당 메서드를 호출한 곳으로 던져준다(throws).
2. 던져진 예외객체를 받을 수 있는 메서드의 경우 try/catch 블록을 사용하여 예외상황를 처리한다.

<pre><code>
//  예외를 던지는 위험한 코드
public void takeRisk() throws BadException{

	if (abandonAllHope){
    	throw new BadException();
    }
}

/* 이렇게 한 메서드에서 던진 것을 다른 메서드에서 잡아야 한다. 예외는 언제나 그 메서드를 호출한 곳으로 던져진다. 예외를 던지는 메서드에서는 반드시 그 메서드에서 예외를 던질 수 있다는 것을 선언해야만 한다. */


// 위의 위험한 메서드를 호출하는 메서드
public void crossFingers(){

	try{
    	exObject.takeRisk();
    }catch(BadException ex){
    	System.out.println("예외다!!");
        ex.printStackTrace();
    }
    // 만약 catch 블록에서 예외상황을 해결할 수 없다면 적어도 printStackTrace()를 사용하여 
    // 스택 트레이스(stack trace)를 출력하는 정도까지는 해줘야 한다.

}

</code></pre>

예외의 종류는 매우 다양합니다. 서로 다른 종류의 예외를 처리하기 위해 Exception 객체를 상속받은 객체들을 사용합니다.

![](assets/images/exception.jpg)

위의 Exception들 중 RuntimeException은 컴파일러에서 따로 확인을 하지 않습니다. 이러한 종류의 예외를 '미확인예외(unchecked exception)'이라고 부릅니다. 물론 RuntimeException도 던지고 잡고 선언할 수 있지만, 컴파일러에서 확인을 하지 않으며 꼭 그래야 하는것도 아닙니다.

그 이유는 RuntimeException의 속성에서 찾아볼 수 있습니다. 대부분의 RuntimeException은 실행 중에 어떤 조건에 문제가 생기는 경우보다는 코드의 논리에 예측 및 예방할 수 없는 방식으로 문제가 생기는 경우에 발생합니다. 파일이 있는지 여부나 서버가 잘 돌아가고 있는지는 항상 장담할 수 없지만, 배열에서 인덱스 범위를 벗어나는 일 같은 경우는 코드를 잘짜면 확실히 방지할 수 있습니다.(.length 속성이 있으니까요)

개발과 테스트 단계에서는 RuntimeException이 그냥 일어나게 두는 것이 낫습니다. 애초에 일어나지 말았어야 하는 일을 try/catch블록으로 처리하는 것은 바람직하지 않기 때문이죠. try/catch는 예외적인 상황을 처리하기 위한 것이지 코드에 있는 문제점을 처리하기 위한 것이 아닙니다. catch블록에서 반드시 성공하리라는 보장이 없는 코드를 시도해보고 코드가 실패했을때 그런 예외적인 상황을 해결하려는 목적으로 존재하는 것이죠.

하지만 예외가 발생하든 발생하지 않든 무조건 실행해야하는 내용이 있을 수 있습니다. 이런 경우 해당 코드는 try블록과 catch블록에 모두 넣어야 하겠죠. 그럼 같은 코드가 중복됩니다. 이런 문제를 해결하기 위해 try/catch블록 외에 finally 블록이 있습니다.

헤드퍼스트에서는 빵을 굽는 상황에 빗대어 finally 블록을 설명하고 있습니다. 

1. 오븐을 켠다
2. 빵을 굽는다
3. 오븐을 끈다

여기서 오븐을 끄는 일은 빵을 굽는 일이 성공하든 실패하든 실행해야 합니다. 따라서 코드로는 다음과 같이 작성합니다.

<pre><code>
try{
	turnOvenon();
    x.bake();
}catch(BakingException ex){
	ex.printStackTrace();
}finally{
	turnOvenOff();
}

</code></pre>

여기까지가 기본입니다. 하지만 큰 시스템 내에서 수많은 예외들을 처리하기 위한 방법들을 이해하기 위해서는 몇 가지 내용을 더 알아보는 것이 필요합니다.

_ _ _

### 2. 다양한 Exception의 처리

메서드를 작성할때 다양한 종류의 Exception이 발생할 수도 있습니다. 다음과 같은 경우이죠.

1. 호출되는 메서드에서 발생할 수 있는 exception이 여러 개인 경우
2. 여러개의 메서드를 호출하는데 호출되는 각각의 메서드에서 발생할 수 있는 exception이 다른 경우

이런 경우 두 가지 방법 중 선택할 수 있습니다.

1. Exception 객체의 다형성을 이용하여 발생하는 exception들을 한번에 잡는다.
2. catch 블록을 여러개 사용하여 발생하는 오류를 각각 잡아 처리한다.

위의 그림에서 봤던 것처럼 모든 예외는 Exception 객체를 상속받아 만든 것입니다. 따라서 catch 블록에서 Exception e를 받으면 모든 종류의 예외를 한번에 잡을 수 있습니다. 하지만 발생한 exception의 종류에 따라 다른 처리를 해줘야 하는 상황에서는 catch 블록을 여러개 사용하여 각각 다른 처리를 해주는 것이 바람직하겠지요.

Exception 객체의 다형성 때문에, 우리는
- 던지고자 하는 예외의 상위클래스 유형을 이용하여 예외를 선언할 수 있습니다.
- 던져지는 예외의 상위클래서 유형을 써서 예외를 잡을 수 있습니다.

이처럼 하나의 catch 블록에 여러종류의 예외를 잡을 수 있다는 특성 때문에 catch 블록을 여러개 사용하여 각각의 exception을 처리할 때 주의해야하는 일이 하나 생깁니다. 헤드퍼스트에서는 다음과 같이 표현합니다.

**큰 바구니를 작은 바구니보다 위에 놓을 수는 없습니다**

exception이 발생했을 때 catch 블록이 있으면 JVM은 무조건 첫번째 블록부터 시작해서 그 예외를 처리할 수 있는 catch 블록을 찾을 때까지 아래로 내려갑니다. 여기서 만약 첫번째 catch 블록이 catch(Exception e)라면 이 블록에서 모든 exception이 잡히므로 그 아래에 있는 catch 블록은 무의미해지고 맙니다. 이런 점을 이야기 한 것입니다.

_ _ _


### 3. Exception 객체의 메서드

1. String getMessage() : 발생된 예외의 메시지를 리턴
2. String toString()   : 발생된 예외의 클래스명과 메시지를 리턴
3. String printStackTrace() : 발생된 예외를 역추적하기 위해 표준 예외 스트림을 리턴(예외 발생장소)


_ _ _

위에서 설명한 것처럼 예외는
**처리하거나 던지거나 둘 중 하나는 무조건 해야합니다**

헤드퍼스트에서는 음악플레이어 어플리케이션으로 설명을 덧붙이고 있지만 저는 여기서 줄이겠습니다.