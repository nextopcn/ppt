# Mockito

## 什么是Mock?

在面向对象程序设计中,模拟对象(英语：mock object,也译作模仿对象)是以可控的方式模拟真实对象行为的假的对象。程序员通常创造模拟对象来测试其他对象的行为,很类似汽车设计者使用碰撞测试假人来模拟车辆碰撞中人的动态行为。

## 为什么要使用Mock?

在单元测试中,模拟对象可以模拟复杂对象的行为, 如果真实的对象无法放入单元测试中,使用模拟对象就很有帮助。  

在下面的情形,可能需要使用模拟对象来代替真实对象：  

1. 真实对象很难搭建起来 (例如,依赖很多其他对象)；
2. 真实对象的行为很难触发(例如,网络错误)；
3. 真实对象速度很慢(例如,一个完整的数据库,在测试之前可能需要初始化)；

## 如何使用Mockito

### 依赖

```java  

<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>3.2.4</version>
    <scope>test</scope>
</dependency>

```

### 简单使用

```java  

import static org.mockito.Mockito.*;

// 创建mock对象
List<String> list = mock(List.class);

when(list.add("1")).thenReturn(true);
when(list.add("2")).thenThrow(new UnsupportedOperationException());
when(list.remove(0)).thenReturn("1");

list.add("1"); // return true
list.add("1"); // return true

list.remove(0); // return "1"

// 验证是否被调用2次
verify(list, times(2)).add(anyString());

assertEquals(true, list.add("1"));

list.add("2"); // throw UnsupportedOperationException

```

### 模拟行为

```java  

when(list.add("1")).thenCallRealMethod();
when(list.add("2")).thenThrow(new UnsupportedOperationException());
when(list.add("3")).thenReturn(true);

```

1. 一个例子  

```java  

	@Test
	public void test() {
		// 创建mock对象
		Mockee mockee = mock(Mockee.class);
		
		when(mockee.test0(1)).thenCallRealMethod();
		when(mockee.test1(2)).thenCallRealMethod();

		System.out.println(mockee.test0(1));
		System.out.println(mockee.test1(2));
	}
	
	class Mockee {
		
		private final String b;
		public Mockee(String b) {
			this.b = b;
		}
		
		public String test0(int a) {
			return String.valueOf(a);
		}
		
		public String test1(int a) {
			Objects.requireNonNull(b);
			return a + b;
		}
		
		public void test2(int a) {
			System.out.println(a);
		}
	}

```

2. 模拟多次调用

```java  

		Mockee mockee = mock(Mockee.class);
		
		when(mockee.test0(1)).thenReturn("1", "2", "3");
		when(mockee.test0(2)).thenReturn("2").thenThrow(new UnsupportedOperationException());

		System.out.println(mockee.test0(1));
		System.out.println(mockee.test0(1));
		System.out.println(mockee.test0(1));
		
		System.out.println(mockee.test0(2));
		System.out.println(mockee.test0(2));

```

3. 模拟返回值是void的函数

```java  

		Mockee mockee = mock(Mockee.class);
		
		doCallRealMethod().when(mockee).test2(0);
		doNothing().when(mockee).test2(1);
		doThrow(new UnsupportedOperationException()).when(mockee).test2(2);
		
		mockee.test2(0);
		mockee.test2(1);
		mockee.test2(2);

```

4. 参数匹配

```java  

		Mockee mockee = mock(Mockee.class);
		when(mockee.test0(anyInt())).thenReturn("1");
		when(mockee.test1(intThat(e -> e > 1 && e < 5))).thenAnswer(e -> String.valueOf((int)e.getArgument(0)));

```

5. 常用参数匹配
```java  
any
anyObject
anyInt
anyBoolean
anyString
anyList
anySet
anyMap
isNull
notNull
eq
same
argThat
doubleThat
stringThat
...
@see ArgumentMatchers
```

6. 常用验证方式

```java  

		Mockee mockee = mock(Mockee.class);

		when(mockee.test0(1)).thenReturn("1");

		mockee.test0(1);
		
		verify(mockee, times(1)).test0(1);
		verify(mockee, never()).test0(2);
		verify(mockee, atMost(1)).test0(2);
		verify(mockee, atLeast(1)).test0(2);

```

### 高级用法及原理

1. Spying on real objects  

```java  

		Mockee mockee = new Mockee("mockee");
		Mockee spyMockee = spy(mockee);
		when(spyMockee.test1(2)).thenReturn("a");

		System.out.println(spyMockee.test0(1)); // return "1"
		System.out.println(spyMockee.test1(2)); // return "a"

```

```java  

		Mockee mockee = new Mockee(null);
		Mockee spyMockee = spy(mockee);
		// when(spyMockee.test1(2)).thenReturn("a"); 空指针
		doReturn("a").when(spyMockee).test1(2);

		System.out.println(spyMockee.test0(1));
		System.out.println(spyMockee.test1(2));
```

2. 构造mock的常用扩展参数

```java  

	Mockee mockee = new Mockee("mockee");
	Mockee mockee = mock(Mockee.class, Mockito.RETURNS_SMART_NULLS);
	Mockee mockee = mock(Mockee.class, Mockito.CALLS_REAL_METHODS);
	Mockee mockee = mock(Mockee.class, Mockito.RETURNS_DEEP_STUBS);
```

```java  

	Foo mock = mock(Foo.class, RETURNS_DEEP_STUBS);
	when(mock.getBar().getName(), "deep");

	// 等价于
	Foo foo = mock(Foo.class);
	Bar bar = mock(Bar.class);
	when(foo.getBar()).thenReturn(bar);
	when(bar.getName()).thenReturn("deep");

```
3. 原理

```java  

public class Mockito {

	public static Map<Invocation, Object> results = new HashMap<Invocation, Object>();
	public static Invocation lastInvocation;
	
	public static Mockee mock(Class<Mockee> clazz) {
		return new MockeeEx();
	}

	public static <T> When<T> when(T o) {
		return new When<T>();
	}

	public static class When<T> {
		public void thenReturn(T retObj) {
			results.put(lastInvocation, retObj);
		}
	}

	public static void main(String[] args) {
		Mockee mockee = mock(Mockee.class);
		when(mockee.test0(1)).thenReturn("a");

		System.out.println(mockee.test0(1));
	}
}

class Invocation {
	private final Object mock;
	private final String method;
	private final Object[] arguments;

	public Invocation(Object mock, String method, Object[] arguments) {
		this.mock = mock;
		this.method = method;
		this.arguments = arguments;
	}

	@Override
	public boolean equals(Object o) {
		if (this == o) return true;
		if (o == null || getClass() != o.getClass()) return false;
		Invocation that = (Invocation) o;
		return Objects.equals(mock, that.mock) &&
				Objects.equals(method, that.method) &&
				Arrays.equals(arguments, that.arguments);
	}

	@Override
	public int hashCode() {
		int result = Objects.hash(mock, method);
		result = 31 * result + Arrays.hashCode(arguments);
		return result;
	}
}

class MockeeEx extends Mockee {

	public MockeeEx() {
		super(null);
	}

	public String test0(int a) {
		Invocation invocation = new Invocation(this, "test0", new Object[] {a});
		Mockito.lastInvocation = invocation;
		if (Mockito.results.containsKey(invocation)) {
			return (String)Mockito.results.get(invocation);
		}
		return null;
	}

	public String test1(int a) {
		Invocation invocation = new Invocation(this, "test1", new Object[] {a});
		Mockito.lastInvocation = invocation;
		if (Mockito.results.containsKey(invocation)) {
			return (String)Mockito.results.get(invocation);
		}
		return null;
	}
}

class Mockee {

	private final String b;
	public Mockee(String b) {
		this.b = b;
	}

	public String test0(int a) {
		return String.valueOf(a);
	}

	public String test1(int a) {
		Objects.requireNonNull(b);
		return a + b;
	}
}

```

### 文档

* [Mockito doc](https://javadoc.io/static/org.mockito/mockito-core/3.2.4/org/mockito/Mockito.html)
