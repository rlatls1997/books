## 9. try-fianlly보다는 try-with-resources를 사용하라

자바 라이브러리에는 close메서드를 호출하여 직접 닫아줘야 하는 자원이 많다(InputStream, OutputStream, java.sql.Connection…)

자원 닫기는 클라이언트가 놓치기 쉽다. 그래서 예측할 수 없는 성능 문제로 이어진다.

이런 자원 대부분이 안전망으로 finalizer를 활용하고 있지만 믿을만하지 못하다.

예외가 발생해도 자원이 닫힘을 보장하기 위해 전통적으로 try-finally를 사용했다.

ex) try-finally : 자원을 회수하는 최선의 방책이 아님

```java
static String firstLineOfFile(String path) throws IOException {
	BufferedReader br = new BufferedREader(new FileReader(path));
	try{
		return br.readLine();
	} finally{
			br.close();
	}
}

```

나쁘지 않다. 하지만 자원을 하나 더 쓴다면 지저분해진다.

ex) 자원이 2개일때 지저분한 try-finally

```java
static void copy(String src, String dst) throws IPXception {
	InputStream in = new FileInputStream(src);
	try{
		OutputStream out = new FileOutputStream
		try{
			...
			while((n = in.read(buf)) >= 0)}{
				out.write(...)
			}
		} finally{
			out.close();
		}
	} finally{
		in.close();
	}
}
```

게다가 위 코드들은 문제가 있다.

try, finally문 모두에서 예외가 발생할 수 있는데, 이런 경우 두 번째 예외가 첫 번째 예외를 덮어써버리고 스택트레이스에 첫 번째 예외 내용이 남지 않는다. ⇒ 디버깅이 어려워진다.

이런 문제들(finally에서도 발생할 수 있는 예외의 문제, 난잡한 코드, 클라이언트의 자원 닫기 미스)을 try-with-resources로 해결하능하다

**이 구조를 사용하려면 자원이 AutoCloseable 인터페이스를 구현해야 한다**

만약 닫아야 하는 자원을 뜻하는 클래스를 작성한다면 AutoCloseable을 반드시 구현하자~

ex) try-with-resources

```java
static void copy(String src, String dst) throws IOException {
	try(InputStream in = new FileInputStream(src);
			OutputStream out - new FileOutputStream(dst)) {
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while((n = in.read(buf)) >= 0){
			out.write(buf, 0, n);
		}
	}
}
```

위에서 만약 try 내부와 close 호출 양쪽에서 예외가 발생한다면,

close에서 발생한 예외는 숨겨지고 try에서 발생한 예외가 기록될 것이다.

숨겨진 예외 또한 스택트레이스에 (suppressed) 라는 꼬리표를 달고 출력된다.

또한 Throwable의 getSuppressed 메서드를 사용하면 프로그램 코드에서 숨겨진 예외를 가져올 수 있다.

catch문도 같이 사용할 수 있다.

**결론**

꼭 회수해야 하는 자원을 다룰때는 try-with-resources를 사용하자.

코드는 간결해지고 생성되는 예외정보도 유용해진다.