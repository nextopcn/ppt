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
	@BenchmarkMode(Mode.Throughput)
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

## BenchmarkMode注解

```java  
@BenchmarkMode(Mode.Throughput)
@BenchmarkMode(Mode.AverageTime)
@BenchmarkMode(Mode.SampleTime)
@BenchmarkMode(Mode.SingleShotTime)
@BenchmarkMode(Mode.All)

```

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
	public void benchThreadLocal(Blackhole bh) {
		raw.set(value); bh.consume(raw.get());
	}
	
	@Benchmark
	@Fork(value = 1, jvmArgs = {"-Djmh.executor=CUSTOM", "-Djmh.executor.class=cn.nextop.erebor.common.util.concurrent.support.XThreadExecutor"})
	public void benchFastThreadLocal(Blackhole bh) {
		fast.set(value); bh.consume(fast.get());
	}
	
	@Benchmark
	@Fork(value = 1, jvmArgs = {"-Djmh.executor=CUSTOM", "-Djmh.executor.class=cn.nextop.erebor.common.util.concurrent.support.NettyExecutor"})
	public void benchNettyFastThreadLocal(Blackhole bh) {
		netty.set(value); bh.consume(netty.get());
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
FastThreadLocalBenchmark.benchFastThreadLocal           abc  thrpt    5  111152325.054 ±  4537967.385  ops/s
FastThreadLocalBenchmark.benchNettyFastThreadLocal      abc  thrpt    5  135880432.285 ±  6920888.144  ops/s
FastThreadLocalBenchmark.benchThreadLocal               abc  thrpt    5  105680948.033 ± 14024203.749  ops/s
```

## State Setup TearDown作用域

```java  

@State(Scope.Benchmark)
@State(Scope.Thread)

@Setup(Level.Trial)
@Setup(Level.Iteration)
@Setup(Level.Invocation)

@TearDown(Level.Trial)
@TearDown(Level.Iteration)
@TearDown(Level.Invocation)

```

* 常用作用域组合

```java  
//
@State(Scope.Benchmark) @Setup(Level.Trial)
@State(Scope.Benchmark) @Setup(Level.Iteration)

// 当带有Threads注解时(不常用)
@State(Scope.Thread) @Setup(Level.Trial)
@State(Scope.Thread) @Setup(Level.Iteration)

```

* 作用域等价代码`@State(Scope.Benchmark)`, `@Setup(Level.Trial)`

```java  

	@State(Scope.Benchmark)
	public static class BenchmarkState {
		private Test test;
		
		@Setup(Level.Trial)
		public void setup() {
			test = new Test();
		}
	}
	
	@Benchmark
	public void benchmark(BenchmarkState state) { 
		state.test.invoke();
	}
```

```java  

BenchmarkState state = new BenchmarkState();
state.setup();

for(int i = 0; i < iterations; i++) {
	iteration(() -> benchmark(state));
}

```

* 作用域等价代码`@State(Scope.Benchmark)`, `@Setup(Level.Iteration)`

```java  

	@State(Scope.Benchmark)
	public static class BenchmarkState {
		private Test test;
		
		@Setup(Level.Iteration)
		public void setup() {
			test = new Test();
		}
	}
	
	@Benchmark
	public void benchmark(BenchmarkState state) { 
		state.test.invoke();
	}
```

```java  

BenchmarkState state = new BenchmarkState();

for(int i = 0; i < iterations; i++) {
	state.setup()
	iteration(() -> benchmark(state));
}

```

## 避免编译优化

```java  
	@Benchmark
	@Fork(1)
	public void benchThreadLocal() {
		raw.set(value); raw.get();
	}

	@Benchmark
	@Fork(1)
	public void benchThreadLocal(Blackhole bh) {
		raw.set(value); bh.consume(raw.get());
	}
```

## 避免常量折叠
```java  
    private double x = Math.PI;

    private final double wrongX = Math.PI;

    @Benchmark
    public double measureWrong_1() {
        // This is wrong: the source is predictable, and computation is foldable.
        return Math.log(Math.PI);
    }

    @Benchmark
    public double measureWrong_2() {
        // This is wrong: the source is predictable, and computation is foldable.
        return Math.log(wrongX);
    }

    @Benchmark
    public double measureRight() {
        // This is correct: the source is not predictable.
        return Math.log(x);
    }
```

[JMHSample_10_ConstantFold](https://hg.openjdk.java.net/code-tools/jmh/file/b6f87aa2a687/jmh-samples/src/main/java/org/openjdk/jmh/samples/JMHSample_10_ConstantFold.java)

## 避免循环优化

```java  

    @Benchmark
    public int measureRight() {
        return (x + y);
    }

    private int reps(int reps) {
        int s = 0;
        for (int i = 0; i < reps; i++) {
            s += (x + y);
        }
        return s;
    }

    @Benchmark
    public int measureWrong_100() {
        return reps(100);
    }

```

[JMHSample_11_Loops](https://hg.openjdk.java.net/code-tools/jmh/file/b6f87aa2a687/jmh-samples/src/main/java/org/openjdk/jmh/samples/JMHSample_11_Loops.java)

# JMH可视化
```java  

        Options opt = new OptionsBuilder()
                .include(LitePoolBenchmark.class.getSimpleName())
                .warmupIterations(10)
                .measurementIterations(10)
                .forks(1)
                .resultFormat(JSON)
                .result("benchmark-" + System.currentTimeMillis() + ".json")
                .build();
        new Runner(opt).run();

```

* [JMH Visualizer](https://jmh.morethan.io/)
* [Lite-pool Benchmark](https://github.com/nextopcn/lite-pool#6-benchmark)
* [JMH samples](https://hg.openjdk.java.net/code-tools/jmh/file/b6f87aa2a687/jmh-samples/src/main/java/org/openjdk/jmh/samples/)