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

