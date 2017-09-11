---
layout: post
comments: true
title: "JUnit参数化介绍与实践"
categories: ["junit", "parameterized"]
---

junit在4.11版本实现了参数化功能，基于这个功能，相当于可以动态生成测试用例的。先看看官方例子:

```java
@RunWith(Parameterized.class)
public class FibonacciTest {

    @Parameters(name = "{index}: fib({0})={1}")
    public static Iterable<Object[]> data() {
        return Arrays.asList(new Object[][] { { 0, 0 }, { 1, 1 }, { 2, 1 },
                { 3, 2 }, { 4, 3 }, { 5, 5 }, { 6, 8 } });
    }

    private int input;
    private int expected;

    public FibonacciTest(int input, int expected) {
        this.input = input;
        this.expected = expected;
    }

    @Test
    public void test() {
        assertEquals(expected, Fibonacci.compute(input));
    }
}
```

使用要点如下:

1. 类名用@RunWith(Parameterized.class)进行注解
2. 需要一个用于准备参数的方法，可以返回列表，但是注意列表中的元素需要是数组，对应于构造方法中的参数顺序。相当于junit会自动根据参数调用构造方法，然后再执行测试用例。这个准备参数的方法需要用@Parameters进行注解。
3. 可以通过设置@Parameters中的name属性自定义展示的测试名称。其中{index}表示当前参数在准备好的参数列表中的索引位置，{0},{1}...表示当前用例中的第x个参数值。以上面为例，第一个用例名称就是0::fib(0)=0

我在最近的工程中，就使用了这一特性。我准备了两个目录，一个用来配置请求报文，另外一个用来配置响应报文。
就是想通过测试用来来比较实际的响应报文和期望的响应报文是否规则匹配。

一开始我使用了在一个测试用例中做这个事情，虽然用例可以跑，但几十个报文是作为一个用例来运行的，
不能直观的展现出具体报文的处理情况。

后来我采用参数化的办法，在准备参数的时候，读取这两个目录生成对应的报文列表，再由测试用例根据请求报文进行业务处理，
得到实际响应报文后，与期望的响应报文进行规则匹配。整个架子的代码写完之后，只需要配置报文就可以增加测试用例了，
更重要的是，可以直观的展现出具体报文的处理情况。示例代码如下:

```java
@RunWith(Parameterized.class)
public class IntegrationTest {
    public IntegrationTest(String name) {
        this.name = name;
    }

    @Parameterized.Parameters(name = "{index}: {0}")
    public static Collection<String[]> data() {
        List<String[]> testData = new ArrayList<String[]>();

        ClassLoader loader = IntegrationTest.class.getClassLoader();
        String reqpath = loader.getResource("req").getPath();
        String[] files = new File(reqpath).list(new FilenameFilter() {
            @Override
            public boolean accept(File dir, String name) {
                return name.endsWith("_req.xml");
            }
        });

        for (String file : files) {
            int ldx = file.lastIndexOf("_req.xml");
            testData.add(new String[]{file.substring(0, ldx)});
        }
        return testData;
    }
    
    @Test
    public void clientTest() {
        //TODO with this.name
    }
}
```
