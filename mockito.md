#Mockito

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