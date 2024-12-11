# Chapter 28. 비지터 패턴

이전 챕터에 나왔던 Modem구조에서는 기반클래스가 모든 모뎀에 대한 공통된 메소드를 갖고있었다.

만약, configureForUnix라는 메서드가 필요하다고 가정하면, 이 메서드는 모든 모뎀의 파생형마다 상이하며 각각 구현되어야한다.

또한 configureForMac, configureForLinux 등 새로운 메서드가 추가되는 상황을 가정하면,  
Modem인터페이스는 변화에 대해 닫지 못하게 되므로(모든 파생형이 이를 구현해야하므로) 모든 모뎀 소프트웨어를 재배포해야한다.

비지터 집합에 속한 패턴은 기존 계층구조를 수정하지 않고 새로운 메서드를 계층구조에 추가할 수 있게 해준다.

## 디자인 패턴의 비지터 집합

- 비지터
- 비순환 비지터
- 데코레이터
- 확장 객체

## 비지터 패턴

Modem 계층 구조는 다음과 같다.  
모든 모뎀이 구현할 수 있는 일반적인 메서드를 포함하고 있다.

<img src="https://github.com/user-attachments/assets/ac9cb0d1-4d38-4658-917b-5d9940fd7690" width="600">

Modem인터페이스에 configureForUnix메서드를 추가하지 않으면서 이 모뎀들을 Unix에 맞춰 환경설정하도록 만들어보자.  
`이중 디스패치`를 사용하면 되는데 이것이 비지터패턴의 핵심이다.

<img src="https://github.com/user-attachments/assets/d4734c3b-5820-4437-a415-aef53bf84778" width="600">

HayesModem 파생형 코드를 보면..

- ModelVisitor.java

```java
public interface ModelVisitor {
	void visit(HayesModem modem);

	void visit(ZoomModem modem);

	void visit(ErnieModem modem);
}
```

- HayesModem.java

```java
public class HayesModem implements Modem {
	@Override
	public void dial(String pno) {
		// 모뎀을 다이얼링한다.
	}

	@Override
	public void hangup() {
		// 모뎀을 끊는다.
	}

	@Override
	public void send(char c) {
		// 문자를 보낸다.
	}

	@Override
	public char recv() {
		// 문자를 받는다.
		return 0;
	}

	@Override
	public void accept(ModelVisitor v) {
		v.visit(this);
	}

	String configurationString = null;
}
```

- UnixModemConfigurator.java

```java
public class UnixModemConfigurator implements ModelVisitor {
	@Override
	public void visit(HayesModem modem) {
		modem.configurationString = "&s1=4&D3";
	}

	@Override
	public void visit(ZoomModem modem) {
		// Set ZoomModem Unix configuration...
	}

	@Override
	public void visit(ErnieModem modem) {
		// Set ErnieModem Unix configuration...
	}
}
```

주목할 점은 모든 Modem파생형마다 비지터 계층구조에 메서드가 하나씩 존재한다는 점이다.

위와 같은 구조에선 Modem계층을 전혀 건드리지 않고도 새로운 운영체제용 환경 설정 함수를 추가할 수 있다.(ModelVisitor의 파생형을 만들면 됨)

이 패턴에서 이중 디스패치인 부분은,  
다형성을 이용해서 어떤 메서드 본체를 부를지 결정하는 작업(디스패치)를 두 번 수행하기 때문이다.

1. accept : accept가 호출되는 객체(Modem)가 어떤 종류(파생형)인지 파악해서 그 객체에 해당하는 accept메서드 본체를 호출한다.
2. visit : 어떤 visit메서드가 호출되어야 할지 결정하는 과정에서 디스패치가 일어난다.

## 비지터는 행렬과 같다

뒤에서 비순환 비지터와 함께 설명

## 비순환 비지터 패턴

비지터 패턴의 구조에선 계층 구조(Modem)의 기반클래스(Modem interface)가 비지터 계층구조(ModemVisitor)의 기반클래스에 의존한다.    
그리고 방문 대상인 계층 구조의 모든 파생형마다 비지터 계층 구조의 기반 클래스에 함수가 하나씩 존재한다.

따라서 방문 대상인 계층구조의 모든 파생형(모든 Modem)이 모두 의존 관계 순환에 빠진다.  
따라서 비지터구조를 점진적으로 컴파일하거나 방문대상인 계층구조에 새로운 파생형을 추가하기가 어려워진다.

비지터 패턴은 새로운 파생형을 자주 추가할 필요가 없는 프로그램에 효과적이다.

반면, 새로운 파생형이 자주 추가되거나 방문 대상 계층 구조가 매우 변경되기 쉽다면 비순환 비지터를 사용할 수 있다.

`비순환 비지터`는 Visitor 기반 클래스를(ModemVisitor)를 퇴화시켜서(메서드를 전부 없애서) 의존 관계 순환을 깬다.

<img src="https://github.com/user-attachments/assets/67fbeba2-fd45-454b-acec-85f03b671ae3" width="600">

비지터 인터페이스는 방문 대상인 계층 구조의 파생형마다 하나씩 존재한다.  
방문 대상인 파생형의 accept함수는 Visitor기반 클래스를 받아서 자신에 해당하는 Visitor 인터페이스로 형변환한다.  
형변환 후에 해당 인터페이스의 visit함수를 호출한다.

HayesModem 기준으로 코드를 보면..

- ModelVisitor.java

```java
public interface ModelVisitor {
	// 메서드가 없는 인터페이스임
}
```

- HayesModemVisitor.java

```java
public interface HayesModemVisitor {
	void visit(HayesModem modem);
}
```

- HayesModem.java

```java
public class HayesModem implements Modem {
	@Override
	public void dial(String pno) {
		// 모뎀을 다이얼링한다.
	}

	@Override
	public void hangup() {
		// 모뎀을 끊는다.
	}

	@Override
	public void send(char c) {
		// 문자를 보낸다.
	}

	@Override
	public char recv() {
		// 문자를 받는다.
		return 0;
	}

	@Override
	public void accept(ModemVisitor v) {
		try {
			HayesModemVisitor hmv = (HayesModemVisitor)v;
			hmv.visit(this);
		} catch (ClassCastException e) {
		}
	}

	String configurationString = null;
}
```

코드가 생략된 UnixModemConfigurator는 이런 모양일 것이다.

- UnixModemConfigurator.java

```java
public class UnixModemConfigurator implements ModemVisitor, HayesModemVisitor, ZoomModemVisitor, ErnieModemVisitor {
	@Override
	public void visit(HayesModem modem) {
		modem.configurationString = "&s1=4&D3";
	}

	@Override
	public void visit(ZoomModem modem) {
		// Set ZoomModem Unix configuration...
	}

	@Override
	public void visit(ErnieModem modem) {
		// Set ErnieModem Unix configuration...
	}
}
```

이렇게 하면

- 의존관계 순환이 깨지고
- 새로운 방문 대상 파생형을 추가하기가 쉬워지고
- 점진적 컴파일 하기도 쉬워진다

단점은,

- 복잡하다.
- 형변환의 시점이 방문 대상 계층구조의 너비에 따라 좌우될지도 모르므로 특정 시점을 잡기 어렵다(?)

## 비순환 비지터는 희소 행렬과 같다.

한 축에 방문 대상 타입이 있고,  
다른 축에 수행할 기능이 있는 함수의 행렬을 만들면

비지터는 완전행렬과 같다.

- 비지터 패턴의 행렬화(0이 아닌 행렬)

| 방문대상\비지터         | UnixModemConfigurator | WindowModemConfigurator | MacModelConfigurator | ...    |
|------------------|-----------------------|-------------------------|----------------------|--------|
| HayesModem       | visit(HayesModem)     | visit(HayesModem)       | visit(HayesModem)    | ...    |
| ZoomModem        | visit(ZoomModem)      | visit(ZoomModem)        | visit(ZoomModem)     | ...    |
| ErnieModem       | visit(ErnieModem)     | visit(ErnieModem)       | visit(ErnieModem)    | ...    |
| 또다른Modem파생형      | visit(또다른Modem파생형)    | visit(또다른Modem파생형)      | visit(또다른Modem파생형)   | ...    |


반면 비순환 비지터는 모든 대상 파생형마다 visitor클래스에 visit함수를 구현하지 않아도 되므로  
빈순환 비지터는 희소 행렬과 같다. (방문대상\비지터 조합 간 일부를 무시할 수 있따)

- 비순환 비지터 패턴의 행렬화(희소행렬)

| 방문대상\비지터         | UnixModemConfigurator | WindowModemConfigurator | MacModelConfigurator | ...    |
|------------------|-----------------------|-------------------------|----------------------|--------|
| HayesModem       | visit(HayesModem)     | 0                       | visit(HayesModem)    | ...    |
| ZoomModem        | visit(ZoomModem)      | visit(ZoomModem)        | 0                    | ...    |
| ErnieModem       | 0                     | visit(ErnieModem)       | visit(ErnieModem)    | ...    |
| 또다른Modem파생형      | visit(또다른Modem파생형)    | 0                       | 0                    | ...    |

## 보고서 생성 프로그램에 비지터 사용하기
큰 자료 구조 내부를 방문하면서 보고서를 생성하긴 위해 비지터 패턴을 사용하는 일은 흔하다.  
자료구조와 보고서 생성코드를 분리할 수 있기 때문.

새로운 보고서를 추가하려면 자료구조의 코드를 만지지 않고 새로운 비지터만 추가하면 된다.

자재 명세서를 나타내는 자료구조를 예시로 들어보자..

조립품(Assembly)에 들어가는 전체 비용 보고서를 생성하거나,  
조립품에 들어가는 모든 부품(Part)의 목록 보고서를 생성할수도 있다.

구조를 보면..

<img src="https://github.com/user-attachments/assets/6aa7cc7d-da99-4b2e-b835-19606c5ff8e5" width="600">

Part클래스에 보고서를 생성하는 메서드를 추가해서 보고서들을 생성할 수도 있다.  
하지만 이렇게 하면 새로운 종류의 보고서가 추가될 때마다 Part계층구조를 변경해야한다.

단일책임원칙에 따라서 코드 변경이 생기는 이유가 다른 코드는 서로 분리해야한다.  
만약 새로운 종류의 부품이 필요한 경우라면 Part계층구조가 바뀌어도 좋지만,  
새로운 종류의 보고서가 필요한 이유 때문에 Part계층구조가 바뀌는 것은 안된다.

위 구조에서는 새로운 종류의 보고서가 추가되어도 Part계층구조가 바뀌지 않는다.

코드를 보면..

- Part.java
```java
public interface Part {
    void accept(PartVisitor v);
}
```
- Assembly.java
```java
public class Assembly implements Part {
    private Part[] parts;

    @Override
    public void accept(PartVisitor v) {
        for (int i = 0; i < parts.length; i++) {
            parts[i].accept(v);
        }
        v.visit(this);
    }
    
    // ... 이 외 Assembly의 구현
}
```

- PiecePart.java
```java
public class PiecePart implements Part {
    @Override
    public void accept(PartVisitor v) {
        v.visit(this);
    }
    
    // ... 이 외 PiecePart의 구현
}
```

- PartVisitor.java
```java
public interface PartVisitor {
    void visit(PiecePart pp);
    void visit(Assembly a);
}
```

- ExplodedCostVisitor.java
```java
public class ExplodedCostVisitor implements PartVisitor {
    private double cost = 0;

    @Override
    public void visit(PiecePart pp) {
		cost += pp.getCost();
    }

    @Override
    public void visit(Assembly a) {
    }
	
    public double getCost() {
        return cost;
    }
}
```

- PartCountVisitor.java
```java
public class PartCountVisitor implements PartVisitor {
	private int pieceCount = 0;

	@Override
	public void visit(PiecePart pp) {
		pieceCount++;
	}

	@Override
	public void visit(Assembly a) {
	}
	
	private int getPieceCount() {
            return pieceCount;
	}
}
```

## 비지터의 다른 용도
자료 구조를 여러 가지 방법으로 해석할 필요가 있는 어플리케이션이라면 언제나 비지터 패턴을 사용할 수 있다.

비지터를 사용하면 언제나 `방문대상인 자료구조` 자체와 그 `자료구조가 사용되는 용도`가 독립적이게 된다.  

비지터를 사용하면 기존 자료 구조를 재컴파일 하거나 재배치 하지 않고도  
이미 지료구조가 설치된 곳에 새로운 비지터를 만들어 배치하거나 기존 비지터를 변경한다음 재배치할 수 있다.

## 데코레이터 패턴
데코레이터 패턴 또한 기존 계층 구조를 바꾸지 않고도 메서드를 추가할 수 있다.

모뎀 계층 구조를 다시 보자..

<img src="https://github.com/user-attachments/assets/ac9cb0d1-4d38-4658-917b-5d9940fd7690" width="600">

여기서 어떤 사용자는 다이얼을 할 때 소리를 듣고 싶어 하고,  
어떤 사용자는 모뎀이소리를 내지 않기를 바란다고 가정해보자.

여러가지 해결책을 살펴보자.

1. 모든 Modem 파생형에 볼륨 조절 코드 넣기
```java
...
Modem m = user.getModem();

if (user.wnatsLoudDial()) {
	m.setVolume(10);
}

m.dial(...);
...
```

이 방법으로 해결하려면 이 코드조각이 몇갠지도 모를 Modem파생형 클래스에 전부 중복해서 들어가야한다. (구림)

2. Modem객체 안에 플래그 지정하기
```java
...
public class HayesModem implements Modem {
    private boolean loudDial = false;

    @Override
    public void dial(String pno) {
        if (loudDial) {
            setVolume(11);
        }
        ...
    }
}
```

이 방법도 모든 파생형마다 코드가 중복된다.  
또한 새로운 Modem파생형을 작성하는 사람은 이 코드를 복사해야 한다는 사실을 기억해야한다. 

3. 템플릿 메서드 패턴 사용 (Modem인터페이스를 클래스로 바꾸자)
```java
public abstract class Modem {
    private boolean wantsLoudDial = false;
	
    public void dial(...) {
        if (loudDial()) {
            setVolume(11);
        }
		dialForReal(...)
    }

    protected abstract void dialForReal();
}
```

이전보다 낫지만 Modem은 다이얼 소리의 크기에 대해 알 필요가 없다.

공통폐쇄원칙(CCP)에 따르면 변경 이유가 다른 것들은 분리해야한다.  
단일책임원칙(SRP)에 따르면 Modem의 진짜 기능은 다이얼 소리를 조절하는 기능과는 관련이 없다. 따라서 Modem의 일부분이 아니다.

데코레이터패턴을 사용해보자.

4. 데코레이터 패턴 사용
LoudDialModem이라는 새로운 클래스를 만든다.  
LoudDialModem은 Modem의 파생클래스이고, 기능은 자신이 포함하고 있는 다른 Modem인스턴스에 위임하는 방식으로 동작.

<img src="https://github.com/user-attachments/assets/9b0b2d65-da67-4737-a022-f682e3008c9a" width="600">

위 구조에서는 dial을 할 때 큰 소리를 낼 것인지에 대한 결정이 단 한 장소(LoudDialModem)에서 일어난다.

- HayesModem.java
```java
public class HayesModem implements Modem {
	private int volume;
	
    @Override
    public void dial(String pno) {
        // 다이얼링한다.
    }
	
    @Override
    public void setVolume(int v) {
    	volume = v;
    }
	
    public int getVolume() {
        return volume;
    }
}
```
- LoudDialModem.java
```java
public class LoudDialModem implements Modem {
    private Modem m;

    public LoudDialModem(Modem m) {
        this.m = m;
    }

    @Override
    public void dial(String pno) {
        m.setVolume(11);
        m.dial(pno);
    }

    @Override
    public void setVolume(int v) {
        m.setVolume(v);
    }

    public int getVolume() {
        return m.getVolume();
    }
}
```

사용은 다음과 같이..

- ModemDecoratorTest.java
```java
public class ModemDecoratorTest {
    public void testLoudDialModem() {
        Modem m = new HayesModem();
        Modem d = new LoudDialModem(m);
		d.dial("5551212");
		assertEquans(d.getVolume(), 11);
    }
}
```

## 다중 데코레이터
동일한 계층 구조에 데코레이터가 2개 이상 있는 경우도 있다.

예로, hangup메서드가 호출될때마다 exit문자열을 보내는 LogoutExitModem을 Modem계층구조의 데코레이터에 추가하고 싶다고 해보자.

LogoutExitModem도 LoudDialModem과 똑같이 위임 관련 코드를 중복해야 할 것이다.

하지만 이런 경우, ModemDecorator라는 새로운 클래스를 만들고,  
이 클래스가 위임 관련 코드를 제공하개 만들면 중복을 제거할 수 있다.

이제 실제 데코레이터는 ModemDecorator를 상속받아서 자신이 필요한 메서드들만 재정의하면 된다.

<img src="https://github.com/user-attachments/assets/8828e1ed-941b-4fc5-8bf5-330b9e916ad7" width="600">

- ModemDecorator.java (위임하는 역할만 수행)
```java
public class ModemDecorator implements Modem {
    private Modem m;

    public ModemDecorator(Modem m) {
        this.m = m;
    }

    @Override
    public void dial(String pno) {
        m.dial(pno);
    }

    @Override
    public void hangup() {
        m.hangup();
    }

    @Override
    public void setVolume(int v) {
        m.setVolume(v);
    }

    @Override
    public int getVolume() {
        return m.getVolume();
    }
	
    protected Modem getModem() {
        return m;
    }
}
```
- LoudDialModem.java
```java
public class LoudDialModem extends ModemDecorator {
    public LoudDialModem(Modem m) {
        super(m);
    }

	// 재정의가 필요한 dial메서드만 재정의
    @Override
    public void dial(String pno) {
        getModem().getMosetVolume(11);
		getModem().dial(pno);
    }
}
```

## 확장 객체 패턴
계층 구조를 변경하지 않고 기능을 추가하는 또 다른 방법.

이 방법에서는 계층 구조에 들어있는 객체마다 특별한 확장 객체 리스트를 유지한다.  
또한 확장 객체를 이름으로 찾을 수 있도록 메서드도 하나 제공한다.

다시 자재 명세서 시스템을 생각해보자.

객체마다 해당 객체를 XML로 만들 수 있는 기능이 필요하다고 가정해보면,  
1. 모든 객체에 toXML 메서드 추가 => 이러면 SRP를 어기게 된다. (객체를 XML로 만드는 것은 자재명세서와 관련 없음)
2. 비지터를 써서 XML 생성 => 자재명세서(BOM)의 객체종료무다 XML 생성 코드를 따로 갖도록 만들 수 없다.  
(하나의 비지터 클래스 안에서 메서드로 나뉘어지므로.., 비순환 비지터패턴도 마찬가지)

종류가 다른 자재명세서 객체마다 XML 생성 클래스를 따로 하나씩 만들고싶다면 어떻게 할 것인가?  
=> 확장 객체 패턴

<img src="https://github.com/user-attachments/assets/7f1e9b5b-6553-488b-89da-c735ecdc7798" width="600">

`<<marker>>` 는 마커인터페이스(메서드가 없는 인터페이스) 를 의미한다.

XML 확장객체를 위주로 코드를 보면..

- Part.java
```java
public abstract class Part {
    HashMap itsExtensions = new HashMap();
	
    public abstract String getPartNumber();
    public abstract String getDescription();
    
    public void addExtension(String extensionType, PartExtension extension) {
        itsExtensions.put(extensionType, extension);
    }
	
    public PartExtension getExtension(String extensionType){
        PartExtension pe = (PartExtension)itsExtensions.get(extensionType);
        if(pe == null) {
            pe = new BadPartExtension();
        }
        return pe;
    }
}
```
- PartExtension.java (마커인터페이스)
```java
public interface PartExtension {
}
```
- PiecePart.java
```java
public class PiecePart extends Part {
    private String partNumber;
    private String description;
    
    public PiecePart(String partNumber, String description) {
        this.partNumber = partNumber;
        this.description = description;
		addExtension("XML", new CSVPiecePartExtension()); // 만약 PartExtension 구현체에 대한 의존을 끊고싶다면 팩터리객체를 사용할 수 있다.
		addExtension("CSV", new XMLPiecePartExtension());
	}
    
    @Override
    public String getPartNumber() {
        return partNumber;
    }
    
    @Override
    public String getDescription() {
        return description;
    }
}
```
- XMLPartExtension.java
```java
public interface XMLPartExtension implements PartExtension {
    Element getXMLElement();
}
```

- XMLPiecePartExtension.java
```java
public class XMLPiecePartExtension implements XMLPartExtension {
    private PiecePart itsPiecePart;
    
    public XMLPiecePartExtension(PiecePart pp) {
        itsPiecePart = pp;
    }
	
    @Override
    public Element getXMLElement() {
        // piecepart를 XML element 로 만들어서 반환
    }
```

자재명세서(BOM) 객체마다 객체의 생성자에서 확장 객체를 만들어서 추가한다는 점을 유의하자.  
이는 곧 자재명세서 클래스가 XML클래스와 CSV클래스에 어느정도 의존한다는 뜻이다.  
이런 사소한 의존까지 끊으려면 각각의 확장 객체들을 만들어서 기반타입으로 반환하는 팩터리객체를 사용할 수 있다.

## 결론
비지터 패턴 집합은 계층 구조에 들어있는 클래스를 고치지 않고도 행위를 변경할 수 있는 방법을 제공한다.  
결론적으로 OCP를 지키는 일에 도움이 된다.

하지만 비지터를 사용하는 것보다 더 간단하게 해결할 수 있는 경우도 많기 때문에 적절한때에 사용해야한다.

# Chapter 29. 스테이트 패턴
## 유한 상태 오토마타의 개괄
유한 상태 기계(FSM: finite state machine)의 한 예로 지하철 개찰구를 생각해보자.

<img src="https://github.com/user-attachments/assets/e129c882-7fd0-4410-92b1-c8d7109938ee" width="600">

위와 같은 다이어그램을 `상태 전이 다이어그램(STD: state transition diagram)`이라고 한다.

상태 전이 다이어그램은 최소 다음으로 구성된다.
- 상태 : 위 다이어그램에서 둥근 상자. Locked, Unlocked
- 전이 : 상태를 연결하는 화살표
- 이벤트 : 전이를 유발하는 이벤트. coin, pass
- 행동 : 이벤트에 따른 행동. lock, unlock

위 다이어그램을 상태 전이 테이블(STT: state transition table)이라는 표로도 나타낼 수 있따.

| 현재상태 | 이벤트 | 다음상태 | 행동 |
|----------|--------|----------|------|
| Locked   | coin   | Unlocked | unlock |
| Unlocked | pass   | Locked   | lock   |


상태 전이 다이어그램(STD)와 상태 전이 테이블(STT)는 기계의 행위를 간결하고 명쾌하게 설명한다.  
동시에 강력한 설계 도구이기도 하다.

설계자는 STD, STT를 보고 이상한 조건이나 정의되지 않은 조건을 찾기 쉬워진다.

위 다이어그램에서는 생략된 전이를 쉽게 알아챌 수 있다.
- Unlocked상태에서 coin이벤트를 다루는 전이가 없다
- Locked상태에서 pass이벤트를 다루는 전이가 없다

생략된 전이는 논리적 결함이 된다.  
STD, STT를 통해 설계가 모든 상태에서 모든 이벤트를 전부 다루는지 쉽게 점검할 수 있다.

다이어그램에서 생략된 전이를 보정하면 다음과 같다.

<img src="https://github.com/user-attachments/assets/204ac5c1-f216-46ca-9e7b-54d9e0b2c504" width="600">

## 구현 기법
## 1. 중첩된 switch/case문
유한 상태 기계(FSM)를 switch/case문으로 구현할 수 있다.

- TurnstileController.java
```java
public interface TurnstileController {
    void lock();
    void unlock();
    void thankYou();
    void alarm();
}
```
- Turnstile.java
```java
public class Turnstile {
	// 상태
	public static final int LOCKED = 0;
	public static final int UNLOCKED = 1;

	// 이벤트
	public static final int COIN = 0;
	public static final int PASS = 1;

	// 전용
	int state = LOCKED;

	private TurnstileController turnstileController;

	public Turnstile(TurnstileController tc) {
		turnstileController = tc;
	}

	public void event(int event) {
		switch (state) {
			case LOCKED:
				switch (event) {
					case COIN:
						state = UNLOCKED;
						turnstileController.unlock();
						break;
					case PASS:
						turnstileController.alarm();
						break;
				}
				break;
			case UNLOCKED:
				switch (event) {
					case COIN:
						turnstileController.thankyou();
						break;
					case PASS:
						state = LOCKED;
						turnstileController.lock();
						break;
				}
		}
	}
}
```

### 범위가 패키지인 상태 변수
위 코드에서 state는 private이 아니다.

작가는 테스트하기 위해 state를 범위가 패키지인 변수로 만드는 대신 주석으로 `//전용`이라고 표시했다.

여기에 그냥 getter, setter 만들면 되지 않냐? 라고 반박할 수 있는데  
=> Test클래스 외에서 노출하려는 변수가 아닌데 의도에 어긋난다. 또한 이는 public과 다를게 없다.

### 행동을 테스트하기
Turnstile을 테스트하기 위해 TurnstileController를 인터페이스로 만들었다.

TurnstileController인터페이스가 유한상태기계의 논리와 행동 사이의 결합을 끊어놓았다.

각 단위를 독립적으로 테스트할 수 있게 되었다.

### 중첩된 switch/case 구현의 비용과 장점
장점
- 명쾌하고 효율적

단점
- 상태와 이벤트가 많아지면 코드를 알아보기 어렵다.

## 2. 전이 테이블 해석
전이를 설명하는 데이터 테이블을 만들어서 FSM을 구현할 수도 있따.

- 개찰구 전이 테이블 구축하기
```java
public Turnstile(TurnstileController tc) { {
    this.tc = tc;
    addTransition(LOCKED, COIN, UNLOCKED, unlock());
    addTransition(LOCKED, PASS, LOCKED, alarm());
    addTransition(UNLOCKED, COIN, UNLOCKED, thankyou());
    addTransition(UNLOCKED, PASS, LOCKED, lock());
}
```

- 전이 엔진(전이테이블을 순회하면서 상태별 이벤트에 해당하는 전이정보를 가져온 뒤 알맞는 행동을 실행한다)
```java
public void event(int event) {
    for(int i = 0; i < transitions.size(); i++) {
        Transition t = (Transition)transitions.get(i);
		
        if(state == t.currentState && event == t.event) {
            state = t.newState;
			t.action.execute();
        }
    }
}
```

### 전이 테이블을 해석하는 접근 방법의 비용과 장점
장점
- 코드를 상태 전이 테이블처럼 읽을 수 있다는 점.(상태 전이를 한 눈에 보기 쉽다)
- 새로운 전이를 추가하는것이 간단하다.

단점
- 전이 테이블을 검색해야하므로 느리다.
- 전이테이블을 지원하기 위한 코드의 양이 많다.


## 3. 스테이트 패턴
스테이트 패턴도 유한 상태 기계(FSM)을 구현하는 방법 중 하나이다.

상태에 대한 인터페이스를 만들고 각 상태에 대한 파생형을 만든다.

<img src="https://github.com/user-attachments/assets/ef9e414c-ec30-4502-a6a9-cad11ad854a7" width="600">

TurnstileLockedState 클래스는 Locked 상태일 때의 이벤트를,  
TurnstileUnlockedState 클래스는 Unlocked 상태일 때의 이벤트를 처리한다.

- TurnstileState.java
```java
public interface TurnstileState {
    void coin(Turnstile t);
    void pass(Turnstile t);
}

class LockedTurnstileState implements TurnstileState {
    @Override
    public void coin(Turnstile t) {
        t.setUnlocked();
        t.unlock();
    }

    @Override
    public void pass(Turnstile t) {
        t.alarm();
    }
}
...
```


- TurnStile.java
```java
public class Turnstile {
    private static TurnstileState lockedState = new LockedTurnstileState(this);
    private static TurnstileState unlockedState = new UnlockedTurnstileState(this);

    private TurnstileController tc;    
    private TurnstileState state = lockedState;
	
    public void coin() {
        state.coin(this);
    }

    public void pass() {
        state.pass(this);
    }
    ...
```

### 스테이트와 스트래터지
스테이트 패턴은 스트래터지 패턴과 비슷하다.  
두 패턴 모두 컨텍스트 클래스가 있고, 두 패턴 모두 파생형이 여러개 있는 다형적인 기반 클래스에 위임한다.

차이점은,  
스테이트에서는 파생형이 컨텍스트 클래스에 대한 참조를 갖고있다는 점이다.

<img src="https://github.com/user-attachments/assets/cc9f8d1b-5e5b-4ed8-8e58-7a7d5c733f03" width="600">

스테이트 패턴은 스트래터지패턴이라고 볼 수 있다.(스트래터지 패턴이 더 큰 집합)

### 스테이트 패턴의 비용과 장점
장점
- 행동은 context클래스, 논리는 state클래스의 파생형들에 분산되어 변경이 쉬워진다.
- 효율적이다.

단점
- state의 파생형을 작성하는 작업은 귀찮다.(상태가 수십개 있으면..)
- 상태기계의 논리를 한 눈에 볼 수 있는 장소가 없다. => 유지보수가 힘들다

## 상태 기계 컴파일러(SMC)
텍스트로 작성된 상태전이테이블을 스태이트 패턴을 구현하는데에 필요한 클래스들로 변환하주는 컴파일러.
(http://www.cleancoders.com 에서 다운 가능하다)

### 스테이트 패턴에 SMC접근 방법을 사용할 때의 비용과 장점
장점
- 별도 파일에 SMC를 위한 유한상태기계에 대한 설명이 한 장소에 모여있어서 알아보기 쉽고 유지보수하기 쉽다.
- 유한상태기계의 논리와 행동의 구현이 철저히 분리되어있다
- 코드를 만들어주므로 코딩량이 적어진다.

단점
- SMC를 사용해야하는것 자체가 비용이다. 새로운 도구의 학습, 설치

## 결론
유한상태기계는 많이 활용되지 않으나 ,  
유한상태기계의 사용이 분명하고 명확하고 유연한 시나리오가 있다.

이런 시나리오에서 스테이트 패턴과상태기계 컴파일러(SMC)를 활용하면 도움이 된다.

# Chatper 30. ETS 프레임워크
ETS : 교육 시험 서비스(Educational Testing Service)

책에서는 교육 시험 서비스 중 건축사 면허 시험 프로그램을 만든 예시가 나온다.

## 소개
### 프로젝트 개요
미국가 캐나다에서 건축사 면허를 따기 위해선 시험을 통과해야한다.

시험은 9과목으로 구성되어 있고 그래픽 과목 3개에서 CAD같은 환경에서 물체를 그리거나 배치하여 해결방안을 만들어야한다.  
ex) 특정 종류의 빌딩 평면도를 설계하라. 빌딩에 들어맞는 지붕을 설계하라 등..

그래픽 시험은 15개의 문제로 나뉜다.  
각 문제는 비네트(vignette)라고 부른다.  
비네트마다 각각 특정 지식 분야를 시험한다. (어떤 비네트는 지붕 설계에 대한 이해를 시험, 어떤 비네트는 평면도 설계에 대한 이해를 시험...)

각 비네트는 두 항목으로 세분된다. 
- 수행평가 : 응시자가 지금 푸는 문제의 해결방안을 '그리게' 해주는 GUI 부분
- 채점 : 수행평가 항목에서 작성된 답안을 읽어서 채점

### 프로젝트 초창기(1993~1994)
최초 비네트 중 가장 복잡한 `빌딩 설계`의 수행평가와 채점을 개발.  

`빌딩 설계` 비네트를 만들면서 나머지 14개의 비네트에서도 재사용 가능한 프레임워크를 만들기 시작했음.

이후 `빌딩 설계`에 대한 수정과 개선사항이 많이 생겨서 오래걸림.  

결론적으로 비네트를 만들기 위한 프레임워크 및 비네트 10개를 더 만들기로 계약함.

### 프레임워크?
새로운 비네트에 프레임워크를 재사용하려고 해보니 잘 되지 않았따.  
결과적으로 코드의 일부분을 새로운 비네트에 복붙해서 사용하게 되었음.

제대로 작동하는 프레임워크를 만들지 못한 이유는
- `빌딩 설계`비네트에만 초점을 맞추고 있었다.
- `빌딩 설계`의 요구사항 변동 및 일정의 압박

결국 `빌딩 설계`에만 한정된 개념들이 프레임워크에 스며들게 되었다.

### 프레임워크!
지금 프레임워크는 재기능을 하지 못하는 것이 분명했다.  
하지만 프레임워크는 필요하다는 결론.

### 1994년 팀
팀원소개

### 마감 시각
1994년 3월에 계약체결.  
프로그램은 1997년에 시험에 사용될 것으로 정함.

### 전략
#### 채택되지 않은 대안
**비네트를 만들기 전에 프레임워크를 재설계 하는 것**  
이렇게 하는 것이 아키텍처 주도적 접근 방법이라고 생각할 수 있따.  

하지만 이 방법대로 하면 실제로 돌아가는 비네트 내부에서 테스트해볼 수 없는 프레임워크 코드가 많이 만들어지게 된다.  
비네트가 프레임워크에서 필요로 하는게 무엇인지 예측할 수 없었기 때문에 실제로 돌아가는 비네트에서 아키텍처를 즉시 검중할 수 있어야 한다고 생각했따.

결론은 새로운 비네트들 여러개와 프레임워크 개발을 동시에 진행

비네트를 개발하면서 프레임워크에는 다음과 같은 공통 기능들이 추가되었다.
- UI화면 구조
- 그래픽 요소를 만들고 움직이고 조정하고, 식별해내고, 지우는 기능
- 확대 및 스크롤
- ... 등 여러가지 

### 결과
#### 하나를 버리기
기존의 `빌딩 설계`는 어떻게 할것인가?
새로운 비네트들이 프레임워크를 재사용할수록 `빌딩 설계`는 잊혀져갔다. (다른 비네트와 달리 구버전 프레임워크를 사용하므로)  
그래서 별도의 수단으로 유지보수하고 발전시켜야만 했다.  

결론은 프로젝트 후반부에서 재설계하기로 정했다.

#### 긴 초기 개발 시간
프레임워크 전략의 부정적인 결과는 첫번째 비테느를 만드는데 상대적으로 개발기간이 길게 걸린다는 것

#### 재사용의 효율성
최초 비네트들이 완성되었다.  
프레임워크 코드는 컸으나 비네트 수행 프로그램들의 크기는 작았다.(일반적으로 4000라인, 작언건 500라인정도)

#### 개발 생산성
`빌딩 설계`를 포함한 첫 비네트 5개는 비네트 1개 당 1인/year가 필요했다.

하지만 이후 비네트들은 1개 당 2.6인/month의 시간이 필요했다.

#### 매주 전달
개발기간에 걸쳐 ETS에게 매주 프로젝트의 중간판을 전달했다.  
ETS는 중간판을 테스트 및 평가한 뒤 변경사항 목록을 전달했다.

#### 견고하고 유연한 설계
아키텍처와 프레임워크 덕분에 요구사항의 변경을 무리없이 대처할 수 있었다.(매 주 변경사항과 고쳐야할 항목이 있었음에도)  
빈번한 변경에도 소프트웨어 설계는 흐트러지지 않았다.

#### 최종 결과
마감기간 내 성공

## 프레임워크 설계
### 채점 애플리케이션들에 대한 공통된 요구사항
간단한 예제로 `초급 수학 시험`을 가정해보자  
덧셈, 뺄셈, 곱셈, 나눗셈 등 다양한 수준의 문제가 있을 것이다.  
프로그램의 목표는 학생들에게 통과/실패 점수를 주는 것이다

지식과 기량을 획득했는지 판단해여 `통과`/`실패` 여부를 알려주고,  
확실하게 판단할 수 없는 경우에는 `미정` 이라는 점수를 준다.

프로그램의 또 다른 목표로는 초급 수학의 주제를 하위 주제로 나눈 다음,  
학생의 강점과 약점도 파악하는 것이다.

<img src="https://github.com/user-attachments/assets/85fe3158-ad85-49aa-9233-8b45dd6586f7" width="600">

말단(leaf)상자는 `특징`이라고 한다.  
`특징`은 평가할 수 있고
- 승인(acceptable, A)
- 불승인(unacceptable, U)
- 미정(indeterminate, I)

를 매길 수 있는 지식의 단위이다.

덧셈을 예로 들면,  
덧셈에는 자릿수가 바뀌는 `받아올림(carry)`(자릿수가 변하는 덧셈)과 덧셈의 기본 `성질(property)`(교환법칙, 결합법칙 등 덧셈의 성질)이 있다.

만약 모든 덧셈문제를 맞췄으면 `받아올림`의 결과도 `승인(A)` 이겠지만,  
덧셈에 틀린 문제가 있을 경우에는 `받아올림`의 특징에서의 평가는 다를 수 있다.

덧셈 문제의 절반을 틀렸고 대부분이 `받아올림`이라면 `받아올림`의 특징은 `불승인(U)`일 것이다.
하지만 `받아올림` 잘못이라고 생각되는게 1/4정도라면 `받아올림`의 특징은 `미정(I)`일 것이다.

다음 단계는 이 분석으로부터 최종점수를 계산하는 것이다.  
비중과 행렬을 사용해서 각 특징의 점수들을 계층 구조 위쪽으로 합쳐올라간다.  
각 단계의 특징의 점수에 비중인자를 적용하고, 비중이 들어간 점수를 이 행렬이 제공하는 표에 적용한다.

- 덧셈의 전체적인 점수를 생성하게 될 표

<img src="https://github.com/user-attachments/assets/1e8352d7-96ea-482b-bf60-b67eb3445b7e" width="600">

예로,  
`받아올림`의 점수가 `미정(I)`이고,  
`성질`의 점수가 `승인(A)`이라고 해보면

두 특징에서 모두 `불승인(U)`점수가 없으므로 가장 왼쪽 열을 본다.  
`받아올림`의 점수가 `미정(I)`인데 비중이 2이므로 3번째 행을 본다.

즉, 덧셈의 결과는 (2, 0) (2행 0열)에 해당하는 `미정(I)`이다.

---

또 다른 예로,
`받아올림`의 점수가 `미정(I)`이고,  
`성질`의 점수가 `미정(I)`이라고 해보면

두 특징에서 모두 `불승인(U)`점수가 없으므로 가장 왼쪽 열을 본다.  
`받아올림`의 점수가 `미정(I)`인데 비중이 2이고,  
`성질`의 점수가 `미정(I)`인데 비중이 1이므로 4번째 행을 본다.

즉, 덧셈의 결과는 (3, 0) (3행 0열)에 해당하는 `불승인(U)`이다.

--- 

또 다른 예로,
`받아올림`의 점수가 `승인(A)`이고,  
`성질`의 점수가 `승인(A)`이라고 해보면

두 특징에서 모두 `불승인(U)`점수가 없으므로 가장 왼쪽 열을 본다.  
두 특징에서 모두 `미정(I)`점수가 없으므로 가장 위쪽 행을 본다.  

즉, 덧셈의 결과는 (0, 0) (0행 0열)에 해당하는 `승인(A)`이다.

이런 과정을 최종 점수가 나올 때까지 계층 구조의 단계마다 반복해서 적용한다.

### 채점 프레임워크 설계

<img src="https://github.com/user-attachments/assets/08c815c0-fbd8-4057-be88-e3d9fff6ead4" width="600">

이 구조는 두 주요 부분으로 나누어 볼 수 있다.

오른쪽 클래스 3개는 프레임워크의 일부분이 아니다.  
이들은 특정한 채점 애플리케이션마다 따로 작성해야만 하는 클래스들을 나타낸다.

나머지 클래스들은 모든 채점 애플리케이션에서 공통으로 쓰이는 프레임워크 클래스들이다.

가장 중요한 클래스는 `Evaluator` 이다.  
채점 트리 구조의 말단 노트와 매트릭스 노드 둘 다 나타내는 추상 클래스이다.  
이 클래스의 Evaluate(ostream&) 함수는 채점 트리 구조에서 어떤 노드의 점수가 필요할 때 호출된다.  
이 함수는 출력 장치에 점수를 기록하기 위한 표준적인 방법을 제공하기 위해 템플릿 메서드 패턴을 사용한다.

- Evaluator
- Evaluator::Evaluate
```cpp
Evaluator :: Score Evaluator :: Evaluate(ostream & o) {
    static char scoreName[] = {'A', 'U', 'I', 'F', 'X'};
    o << itsName << ":" score = DoEval();
    o << scoreName[score] << endl;
    
    return score; 
```

Evaluate() 함수는 채점트리구조 노드의 실제 점수 평가 작업을 수행하는 DoEval()을 호출한다.  

채점트리구조의 말단 노드는 VignetteFeature 클래스로 나타낸다.  
각 VignetteFeature 클래스는 자신이 맡고 있는 특징의 점수를 계산하기 위해 DoEval()을 재정의한다.

채점트리구조의 행렬 노드는 FeatureGroup 클래스로 나타낸다.  
- addEvaluator : 자식노드를 FeatureGroup에 추가할 수 있다.  
=> Addition노드는 FeatureGroup이 될 것이고 Carry와 Properties 노트를 추가하기 위해 addEvaluator()를 2번 호출할 것이다.

- FeatureGroup
```cpp
class FeatureGroup : public Evaluator {
    public :
        ...
        void AddEvaluator(Evaluator* e, int rank);
        
        void AddMatrixElement(int i, int u, Score s);
        ...
```

- addMatrixElement : 행렬에 요소를 추가한다.  
=> 좀전에봤던 평가를 위한 행렬의 내용을 추가
```cpp
addition.AddMatrixElement(0, 0, Evaluator::A);
addition.AddMatrixElement(0, 1, Evaluator::U);
addition.AddMatrixElement(0, 2, Evaluator::U);
...
```

그리고 DoEval()은 Evaluator들의 리스트 원소를 하나씩 방문하면서 점수에 등급을 곱하고,  
그 결과를 I와 U 누적 점수 중 해당하는쪽에 더하면 된다.  
모든 원소를 방문하고 나면 누적점수들을 행렬 요소의 좌표로 사용하여 최종 결과를 얻어내면 된다.

VignetteScoringApp 클래스를 통해 실제 애플리케이션을 변경하지 않고도 트리의 모양과 비중을 바꿀 수 있따.  
...이하 생략(이 구조에서 자신만의 채점 애플리케이션을 만드는 내용)

## 템플릿 메소드 사례
비네트 중 하나는 빌딩의 평면도를 그리는 능력을 시험하는 비네트이다.  
응시자는 방, 복도, 문, 창문, 벽의 틈새, 층계, 엘리베이터 등을 그려야 한다.

비네트 프로그램은 응시자가 그린 것을 채점 프로그램이 해석할 수 있는 자료구조로 전환한다.  
구조는 다음과 같다.

<img src="https://github.com/user-attachments/assets/9182312f-bf5c-43eb-a182-e3fcdd203c8f" width="600">

채점은 다음과 같은 특징들을 가지고 시험하는 방법으로 이루어진다.
- 응시자가 필요한 공간을 모두 그렸는가?
- 기관에서 가장 높은 사람의 방에서 산의 경치를 볼 수 있는가?
- 식당과 부엌을 쉽게 오갈 수 있는가?
- ...

위의 각 특징들마다 하나의 클래스(Feature)를 만들었다.  
각 Feature 클래스마다 위의 자료구조를 순회하며 점수를 계산하는 Evaluate메서드를 만들었다.  
결과는, 동일 자료구조를 순회하는 Feature가 수십개에 달하여 코드중복이 매우 많아졌다.

### 반복문을 단 한번만 작성하기
코드중복 문제를 해결하기 위해 템플릿 메서드 패턴을 사용했다.

- solspcft.h
```cpp
// 이 클래스는 응시자가 그린 해결 방안의 공간들을 순회하면서
// 우리가 원하는 공간과 일치하는 것을 찾는 반복문을 제공하는 기반 클래스이다.
// 일치하는 공간을 찾았을 때 원하는 일을 할 수 있도록 순수 가상 함수를 제공한다.

class SolutionSpaceFeature : public Evaluator {
    public :
        SolutionSpaceFeature(Queue<SolutionSpace*>&);
        virtual ~SolutionSpaceFeature();
        virture Evaluator::Score DoEval();
        virture void NewSolutionSpace(SolutionSpace*);
        virtual Evaluator::Score getScore() = 0;
        
    private :
        SolutionSpaceFeature(const SoluitonSpaceFeature&);
        SolutionSpaceFeature& operator=(const SolutionSpaceFeature&);
        
        Queue<SolutionSpace*>& itsSolutionSpaces;
    ...
```

- solspcft.cpp
```cpp
...
Evaluator::Score SolutionSpaceFeature::DoEval() {
    Set<SolutionSpace*>& theSet = GscoreFilter -> GetSolutionSpaces();
    SelectiveIterator<SolutionSpace*>ai(theSet, itsSolutionSpaceQuery);
    
    for(; ai; ai++) {
        SolutionSpace& as = **ai;
        NewSolutionSpace(*ai);
    }
    
    return GetScore;
}
```

DoEval()함수는 모든 SolutionSpace객체를 순회하는 반복문이 있다.=> 템플릿 메서드   
이 반복문에서 순수가상함수 NewSoliutionSpace()를 호출한다.  
각 특징을 나타내는 SolutionSpaceFeature의 각 파생형들은 NewSolutionSpace()를 재정의하여 특정한 채점기준으로 각 공간을 채점한다.

## 수행평가 애플리케이션에 대한 공통된 요구사항
수행평가 프로그램들은 서로 겹치는 부분이 많았다.  

예로, 화면 구조가 모든 비네트에서 동일했고,  
화면의 버튼(삭제, 이동/조정, 확대, 완료) 등의 행위가 같았다.

반면 비네트별로  
어떤 비네트는 방을 다루고,  
어떤 비네트는 윤곽선을 다루고,  
어떤 비네트는 땅 경계선이나 지붕을 다루는 베트느도 있었다.

하지만 전반적인 패러다임은 비슷했다.

유사성이 많다는 말은 재사용할 수 있는 기외가 많다는 뜻이다.

## 수행평가 프레임워크 설계
ETS프레임워크의 `이벤트모델`과 `작업감독자 아키텍처`를 살펴보자

### 이벤트 모델
이벤트는 사용자가 하는 모든 행동마다 발생한다.

대부분의 이벤트들은 프레임워크가 처리할 수 있지만,  
각개의 비네트에서 프레임워크가 특정 이벤트를 처리하는 방식을 재정의할 필요가 있을 수도 있다.  

따라서 비네트가 이벤트처리를 재정의할 수 있도록 만들어야 했다.

어떤 비네트의 명령창에서 이벤트를 처리하기 위해 사용되는 유한 상태 기계를 보자

<img src="https://github.com/user-attachments/assets/d8c096d6-10f8-4a7d-aac4-5fff0fa73fbe" width="600">

ScreenCursor이벤트를 살펴보자.  
이 이벤트로 인해 커서의 상태는 바뀌지만 이벤트처리기에서 상태 변화가 일어나지 않는다.

어떤 객체를 지우고싶을땐 `삭제`버튼을 클릭하고, 작업창에서 지울 요소들을 선택한 뒤 `삭제`버튼을 한번 더 누르면 된다.  
첫 번째 Erase이벤트를 받으면 Idle -> Erasing 상태로 가는 전이가 일어난다.

Erasing 상태에서 ScreenCursor 이벤트가 발생해도 삭제 작업은 계속된다.  
Erasing 상태에서 나오는 방법은 2개이다.  
- 또 다른 Erase이벤트가 발생시 요소들을 삭제하고 Erase Task 종료
- 명령창의 일부 버튼을 클릭(그리기(Sketch), 길이재기(Measure) 등)

그리기메뉴에서 `길이재기` 항목을 선택하면 Measure 이벤트가 발생한다.  
길이를 재는 동안 임의의 지점 2개를 클릭할 수 있다. 그러면 두 점 사이의 거리가 나타난다.  
Measuring 상태인 동안 계속 반복된다.  
종료방법은 `삭제`나 `그리기` 같이 다른 작업을 시작하는 버튼을 클릭해야한다.

#### 이벤트 모델 설계

<img src="https://github.com/user-attachments/assets/131225c6-3b18-4c35-a07a-61b48467f4fa" width="600">

CommandWindow, StandardCommandWindow, StandardFSM은 프레임워크 클래스이다.  
나머지는 특정 비네트에 속한 클래스들이다.

VignetteCommandWindow는 이벤트를 받아서 FSM에 넘긴다.  
이는 이 이벤트에 맞는 행동들을 구현하고 있는 CommandWindow에 넘겨진다.

CommandWindow는 MeasureTask나 EraseTask같은 표준 행동들의 구현을 제공한다.  
StandardCommandWindow는 표준 이벤트를 상태 기계로 넘겨 처리하는 작업을 제공한다.  
VignetteComandWindow는 특정 비네트 전용 행동의 구현과 비네트 전용 이벤트의 처리 작업을 제공한다.  

이런식으로 프레임워크는 모든 공통 작업의 기본 구현과 이벤트를 제공하지만,  
어떤 구현이나 이벤트 처리라도 비네트에서 재정의될 수 있따.

#### 표준 이벤트 추적하기

<img src="https://github.com/user-attachments/assets/c9630bc2-87db-4b8d-b17c-88af03262e74" width="600">

- 1:Measure : 사용자에 의해 이벤트가 발생한다. 이는 GUI에서 VignetteCommandWindow에 전달된다
- 1.1:Measure : 이 이벤트의 기본처리는 StandardCommandWindow에서 제공한다. 이 에벤트는 StandardFSM에게 전달된다.
- 1.1.1:Measure : Measure이벤트는 StandardFSM의 실제 구현인 VignettedFSM 클래스차원까지 내려간다.  
VignetteFSM은 이 Measure메시지를 VignetteFSMState에게 보낸다
- 1.1.1.1:SetState(ms) : VignetteFSMState는 추상클래스이다. 각 상태마다 이 클래스의 파생형이 하나씩 있다.  
현재 Idle상태라고 가정하면 이는 IdleState 객체에 전달된다.
- 1.1.1.2:MeasureTask() : Idle상태에서 Measure이벤트에 대한 반응으로 일어나야 하는 행동을 VignetteFSM에 다시 전달한다.  
MeasureTask()는 이를 구현하고 있는 VignetteFSMGlue클래스에서 구현한다.
- 1.1.1.2.1:MeasureTask() : VignetteFSMGlue는 이 행동이 CommandWindow에서 선언된 표준 행동임을 인식하고 MeasureTask()메시지로 다시 그곳에 돌려준다.

이렇게해서 메시지가 한 바퀴 돌아서 다시 GUI로 나타날 것이다.

#### 특정 비네트 전용 이벤트 추적하기

<img src="https://github.com/user-attachments/assets/8eefd9c4-4e32-4793-87e9-693c66966e93" width="600">

MeasureTask()가 아닌 SpecificTask()가 발생했을때는 StandardCommandWindow를 거치지 않고 처리된다.

메시지도 똑같이 VignetteFSMGlue까지 내려가지만 메시지가 특정 비네트 전용이므로 바로 전용 행동이 구현되어 있는 VignetteCommandWindow에게 전달된다.

#### 명령 창 상태 기계를 생성하고 재사용하기
이벤트와 행동을 처리하기 위해 위와 같이 굉장히 많은 클래스가 필요한 것은 의아하다.

하지만 클래스가 많을지 몰라도 객체의 수는 매우 적다.

실제로 인스턴스화할 객체는 VignetteCommandWindow, VignetteFSMGlue와 다양한 상태객체들 뿐인데,  
이는 도구를 사용하면 자동으로 생성된다. (SMC)

## 작업감독자 아키텍처
MeasureTask를 생각해보자

Measure를 시작하고,  

한 점을 클릭하면 그 지점에 표시가 남는다.  
이 상태에서 마우스를 움직이면 클릭한 지점부터 마우스 커서까지 직선이 동적으로 그려진다.  
이 직선의 길이도 메시지창에 표시된다.  
사용자가 두번째로 마우스를 클릭하면 또 다른 표시가 그려지고, 직선은 사라지며 두 지점 사이의 최종거리가 표시된다.  
사용자가 다시 클릭하면 전체과정이 다시 시작된다.

<img src="https://github.com/user-attachments/assets/877c16d2-e647-44e0-a232-b38f26ad699f" width="600">

이거 말고 아래 그림처럼 `두 점 상자`를 그리는 작업도 있는데 비슷하니 패스

<img src="https://github.com/user-attachments/assets/ebca56a3-e8d6-431b-b6f9-2c6561dbb026" width="600">



<img src="https://github.com/user-attachments/assets/0d676aa4-97ca-4ed0-bf40-776d3ba97311" width="600">

위 그림에서 Measure Task를 보면,

개발자가 작성해야 하는 클래스는 VignetteTaskWindow와 각 작업 클래스의 비네트 전용 파생형뿐이다.

## 결론
재사용 가능한 프레임워크를 만드는 과정을 봤다.


