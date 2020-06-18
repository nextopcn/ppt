# JMH

JMH是Openjdk开发的一款java基准测试工具

JVM上进行基准测试的两条重要准则

1. 在同样的硬件环境，同样的参数下比对测试结果
2. 考虑JVM warmup, gc对基准测试的影响

# 简单用法

```java  

@State(Scope.Benchmark)
public class ExampleBenchmark {
	
	private Test test;
	
	@Setup
	public void setup() {
		test = new Test();
	}
	
	@Benchmark
	public void benchmark() { 
		test.invoke();
	}
	
	public static void main(String[] args) throws Exception {
		Options opt = new OptionsBuilder()
				.include(ExampleBenchmark.class.getSimpleName())
				.warmupIterations(3)
				.measurementIterations(5)
				.warmupTime(TimeValue.seconds(1))
				.measurementTime(TimeValue.seconds(1))
				.shouldDoGC(true)
				.forks(1)
				.build();
		new Runner(opt).run();
	}
	
	private static class Test {
		String invoke() { return null; }
	}
}

```

结果解析

```java  

# JMH version: 1.23
# VM version: JDK 1.8.0_201, Java HotSpot(TM) 64-Bit Server VM, 25.201-b09
# VM invoker: C:\Java\jdk1.8.0\jre\bin\java.exe
# VM options: -javaagent:C:\Program Files\JetBrains\IntelliJ IDEA 2019.2.4\lib\idea_rt.jar=53619:C:\Program Files\JetBrains\IntelliJ IDEA 2019.2.4\bin -Dfile.encoding=UTF-8
# Warmup: 3 iterations, 1 s each
# Measurement: 5 iterations, 1 s each
# Timeout: 10 min per iteration
# Threads: 1 thread, will synchronize iterations
# Benchmark mode: Throughput, ops/time
# Benchmark: cn.nextop.erebor.common.util.ExampleBenchmark.benchmark

# Run progress: 0.00% complete, ETA 00:00:08
# Fork: 1 of 1
# Warmup Iteration   1: 328788698.098 ops/s
# Warmup Iteration   2: 345128735.736 ops/s
# Warmup Iteration   3: 384900394.095 ops/s
Iteration   1: 384850432.016 ops/s
Iteration   2: 355602743.216 ops/s
Iteration   3: 398663694.390 ops/s
Iteration   4: 378253474.410 ops/s
Iteration   5: 392780102.432 ops/s


Result "cn.nextop.erebor.common.util.ExampleBenchmark.benchmark":
  382030089.293 ±(99.9%) 64227926.234 ops/s [Average]
  (min, avg, max) = (355602743.216, 382030089.293, 398663694.390), stdev = 16679788.478
  CI (99.9%): [317802163.059, 446258015.527] (assumes normal distribution)


# Run complete. Total time: 00:00:14

REMEMBER: The numbers below are just data. To gain reusable insights, you need to follow up on
why the numbers are the way they are. Use profilers (see -prof, -lprof), design factorial
experiments, perform baseline and negative tests that provide experimental control, make sure
the benchmarking environment is safe on JVM/OS/HW level, ask for reviews from the domain experts.
Do not assume the numbers tell you what you want them to tell.

Benchmark                    Mode  Cnt          Score          Error  Units
ExampleBenchmark.benchmark  thrpt    5  382030089.293 ± 64227926.234  ops/s

```

# JMH注解

## State注解 Setup TearDown注解

```java  
@State(Scope.Benchmark)
public class ExampleBenchmark {
	
	private Test test;
	
	@Setup
	public void setup() {
		test = new Test();
	}
	
	@Benchmark
	public void benchmark(ExampleBenchmark that) { 
		that.test.invoke();
	}
}
```

等价于

```java  

public class ExampleBenchmark {
	
	@State(Scope.Benchmark)
	public static class BenchmarkState {
		private Test test;
		
		@Setup
		public void setup() {
			test = new Test();
		}
	}
	
	@Benchmark
	public void benchmark(BenchmarkState state) { 
		state.test.invoke();
	}
}

```

等价于

```java  

public class ExampleBenchmark {
	
	private static Test test;
	static {
		test = new Test();
	}
	
	@Benchmark
	public void benchmark(BenchmarkState state) { 
		test.invoke();
	}
}

```

## Threads注解

```java  

public class ExampleBenchmark {
	
	@State(Scope.Benchmark)
	public static class BenchmarkState {
		private Test test;
		
		@Setup
		public void setup() {
			test = new Test();
		}
	}
	
	@Benchmark
	@Threads(5)
	// state 对象需要线程安全
	public void benchmark(BenchmarkState state) { 
		state.test.invoke();
	}
}

```

一个稍微复杂的线程池基准测试例子[LitePoolBenchmark](https://github.com/nextopcn/lite-pool/blob/master/src/test/java/cn/nextop/lite/pool/benchmark/LitePoolBenchmark.java)

## Fork Param注解 

```java  
@State(Scope.Benchmark)
@Warmup(iterations = 3, time = 1, timeUnit = TimeUnit.SECONDS)
@Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
public class FastThreadLocalBenchmark {
	
	private ThreadLocal<String> raw = new ThreadLocal<>();
	private FastThreadLocal<String> fast = new FastThreadLocal<>("test");
	private io.netty.util.concurrent.FastThreadLocal<String> netty = new io.netty.util.concurrent.FastThreadLocal<>();
	
	@Param("abc")
	private String value;
	
	@Benchmark
	@Fork(1)
	public void benchThreadLocal() {
		raw.set(value); raw.get();
	}
	
	@Benchmark
	@Fork(value = 1, jvmArgs = {"-Djmh.executor=CUSTOM", "-Djmh.executor.class=cn.nextop.erebor.common.util.concurrent.support.XThreadExecutor"})
	public void benchFastThreadLocal() {
		fast.set(value); fast.get();
	}
	
	@Benchmark
	@Fork(value = 1, jvmArgs = {"-Djmh.executor=CUSTOM", "-Djmh.executor.class=cn.nextop.erebor.common.util.concurrent.support.NettyExecutor"})
	public void benchNettyFastThreadLocal() {
		netty.set(value); netty.get();
	}
	
	public static void main(String[] args) throws Exception {
		Options opt = new OptionsBuilder()
				.include(FastThreadLocalBenchmark.class.getSimpleName())
				.shouldDoGC(true)
				.build();
		new Runner(opt).run();
	}
}

Benchmark                                           (value)   Mode  Cnt          Score          Error  Units
FastThreadLocalBenchmark.benchFastThreadLocal           abc  thrpt    5  193088333.586 ± 17097130.218  ops/s
FastThreadLocalBenchmark.benchNettyFastThreadLocal      abc  thrpt    5  206931335.821 ± 18390810.648  ops/s
FastThreadLocalBenchmark.benchThreadLocal               abc  thrpt    5  118572794.293 ± 73803453.678  ops/s
```

## State Setup TearDown作用域



## 避免编译优化