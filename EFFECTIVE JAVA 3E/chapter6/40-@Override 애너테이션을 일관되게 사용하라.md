## 40. @Override 애너테이션을 일관되게 사용하라

@Override : 메서드 선언에만 달 수 있는 애너테이션. 상위 타입의 메서드를 재정의했음을 뜻한다.
이 애너테이션을 일관되게 사용하면 여러 버그들을 예방할 수 있다.

ex) 알파벳 2개로 구성된 문자열을 표현하는 클래스 - 버그가 있음

```java
public class Bigram {
	private final char first;
	private final char second;

	public Bigram(char first, char second){
		this.first = first;
		this.second = second;
	}
	
	public boolean equals(Bigram b){
		return b.first == first && b.second == second;
	}
	
	public int hashCode(){
		return 31 * first + second;
	}

	public static void main(String[] args){
		Set<Bigram> s = new HashSet<>();
		for(int i = 0; i< 10; i++){
			for(char ch = 'a'; ch <= 'z'; ch++){
				s.add(new Bigram(ch, ch));
			}
		}
		sout(s.size());
	}
}
```

위 코드는 버그가 있다.

Set은 중복을 허용하지 않으므로 Bigram(a,a), Bigram(b,b) … Bigram(z,z) 총 26개의 원소가 있을것으로 예상되지만 s.size()는 260을 반환한다.

원인은 equals메서드인데,

Object의 equals메서드는 매개변수 타입이 Object인데 위 equals메서드는 Bigram이므로 overriding한 것이 아니라 overloading을 한 것이다. 따라서 Set이 원하는대로 중복을 제거하지 못했다.

만약 위 코드에 @Override애너테이션을 달아줬더라면 컴파일 오류가 발생하여 잘못된 부분을 빠르게 찾을 수 있었을 것이다.

```java
	@Override
	public boolean equals(Object o){
		if(!(o instanceof Bigram)){
			return false;
		}
		Bigram b = (Bigram)o;
		return b.first == first && b.second == second;
	}
```

**버그를 방지하기 위해 상위 클래서의 메서드를 재정의하려는 모든 메서드에는 @Override 애너테이션을 달아주자**

**예외 케이스**
하나의 예외 케이스가 있다.

구체클래스에서 상위클래스의 메서드를 재정의할때는 굳이 @Override를 달아주지 않아도 된다.

아직 구현하지 않은 추상메서드가 있으면 컴파일러가 알려주기 때문이다. 달아줘도 괜찮다.

IDE에서 설정을 해두면 @Override가 달려있지 않은 메서드가 재정의를 했을 때 이를 알려주는 경고를 주기도 한다. IDE, 컴파일러를 통해 의도한 재정의만 정확하게 해낼 수 있다.

@Override는 인터페이스의 메서드를 재정의할때도 사용할 수 있다. 추상ㅋ

**결론**

재정의한 모든 메서드에 의식적으로 @Override애너테이션을 달아서 실수를 줄이자.