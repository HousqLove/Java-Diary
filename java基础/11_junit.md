# 概述
junit是著名的单元测试库，目前发展到JUnit5，Junit5需要运行在Java 8或更高版本。JUint5包括：
- JUnit平台：作为一个基础发射测试框架在JVM上，还定义了TestEngine。
- JUnit Jupiter：新的编程模型和扩展模型组合，用于在JUnit5中编写测试和扩展。
- JUnit Vintage：提供一个TestEngine，用于在平台上测试运行基于JUnit3和Junit4的测试的功能。

# 编写测试用例

## 注解
- @Test：标注测试方法。
- @ParameterizedTest：表示参数化测试。
- @RepeatedTest：表示重复测试的测试模板
- @TestFactory：表示方法是动态测试的测试工厂
- @TestTemplate：表示方法是测试用例的模板，设计用于根据已注册提供者返回的调用上下文的数量被多次调用的测试用例。
- @TestMethodOrder：用于为带注解的测试类配置测试方法的执行顺序
- @TestInstance：用于为带注解的测试类配置测试实例生命周期
- @DisplayName：声明测试类或测试方法的自定义名称
- @DisplayNameGeneration：声明测试类的自定义名称生成器
- @BeforeEach：声明这个方法将在每个测试方法之前执行，类似JUnit4的@Before
- @AfterEach：声明这个方法将在每个测试方法之后执行，类似Junit4的@After
- @BeforeAll：声明这个方法将在所以的测试方法之前执行，类似Junit4的@BeforeClass，所有被注解的测试方法都必须是静态和可继承的。
- @AfterAll：声明这个方法将在所以的测试方法之后执行，类似Junit4的@AfterClass，所有被注解的方法都必须是静态和可继承的。
- @Nested：声明这个方法是一个嵌套的，非静态的方法。由于Java语言的限制，@BeforeAll和@AfterAll不能用在@Nested注解的方法上。
- @Tag：被用于通过声明标签来过滤测试方法，既不是在方法级别也不是在类级别，相当于Junit4中的测试组
- @Disabled：用于声明一个测试方法或测试类失效，类似于Junit4的@Ignore
- @ExtendWith：被用于使用在用户定制的扩展组件上。

Junit Jupiter注解可用于元注解。意味着你可以定义自己的定制的注解，定制的注解就可以从元注解进行继承。

测试类和测试方法都可以通过@DisplayName指定名称，指定的名称将用于测试任务和测试报告中。

# 断言Assertions
Junit Jupiter继承了许多Junit4中的断言方法，同时增加了适配Java 8 lambdas特点的方法。断言方法都是静态方法。

# 假定Assumption
Junit Jupiter继承了Junit4中一部分假定方法，同时提供了部分借用Java 8 lambdas的特性。所有的假定都是静态方法。
```
@Test
void testOnlyOnCiServer() {
    assumeTrue("CI".equals(System.getenv("ENV")));
    // remainder of test
}
```

# 禁用测试
通过@Disabled注解标注禁用测试的类或方法。

# 标签和过滤（Tagging and Filtering）
测试类和方法可以被打上标签，这些标签可以被之后用来过滤

# 嵌套测试（Nested Tests）
嵌套测试给测试者提供表达不同测试组之间关系的功能。如下测试一个堆栈：
```java
@DisplayName("A stack")
class TestingAStackDemo {

    Stack<Object> stack;

    @Test
    @DisplayName("is instantiated with new Stack()")
    void isInstantiatedWithNew() {
        new Stack<>();
    }

    @Nested
    @DisplayName("when new")
    class WhenNew {

        @BeforeEach
        void createNewStack() {
            stack = new Stack<>();
        }

        @Test
        @DisplayName("is empty")
        void isEmpty() {
            assertTrue(stack.isEmpty());
        }

        @Test
        @DisplayName("throws EmptyStackException when popped")
        void throwsExceptionWhenPopped() {
            assertThrows(EmptyStackException.class, () -> stack.pop());
        }

        @Test
        @DisplayName("throws EmptyStackException when peeked")
        void throwsExceptionWhenPeeked() {
            assertThrows(EmptyStackException.class, () -> stack.peek());
        }

        @Nested
        @DisplayName("after pushing an element")
        class AfterPushing {

            String anElement = "an element";

            @BeforeEach
            void pushAnElement() {
                stack.push(anElement);
            }

            @Test
            @DisplayName("it is no longer empty")
            void isEmpty() {
                assertFalse(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when popped and is empty")
            void returnElementWhenPopped() {
                assertEquals(anElement, stack.pop());
                assertTrue(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when peeked but remains not empty")
            void returnElementWhenPeeked() {
                assertEquals(anElement, stack.peek());
                assertFalse(stack.isEmpty());
            }
        }
    }
}
```
只有非静态的嵌套类（如：内部类）可以被标记为@Nested测试。

# 对构造函数和方法的依赖注入
Junit Jupiter测试构造函数和方法被允许拥有参数。
- ParameterResolver定义了在运行期间动态的解析参数值的API接口。如果一个测试单元的构造函数或一个 @Test, @TestFactory, @BeforeEach, @AfterEach, @BeforeAll, 和 @AfterAll 注解的方法接受了一个参数, 这个参数必须在 运行的时候通过一个注册的 ParameterResolver 进行解析。有两个内置的自动注册的解析器：
    - TestInfoParameterResolver：如果一个方法参数是一种TestInfo类型， 这个 TestInfo可以被用于 获取一个测试的 展示名称, 测试的类, 测试的方法 或者对应的标签tags. 展示名称要么是一个技术名称, 比如 测试类的或者测试方法的名字, 要么是一个 通过 @DisplayName 定制化的名称。

    - RepetitionInfoParameterResolver:如果@ RepeatedTest，@ BeforeEach或@AfterEach方法中的方法参数的类型为RepetitionInfo，则RepetitionInfoParameterResolver将提供RepetitionInfo的实例。 然后，可以使用RepetitionInfo检索有关当前重复以及相应@RepeatedTest的重复总数的信息。 但是请注意，RepetitionInfoParameterResolver并未在@RepeatedTest的上下文之外注册。 

- TestReporterParameterResolver：如果一个方法的参数 是一个 TestReport 类型, 这个 TestReporterParameterResolver 将会 获取一个 TestReport实例. 这个 TestReport将会被用于输出目前测试运行中的数据. 这些数据将被可以用于 TestExecutionListener.reportingEntryPublished() 同时将会被IDEs使用, 或者被包含在测试报告中。

在JUnit Jupiter中，应该使用TestReport来打印Junit4的stdout和stderr。通过使用@RunWith(JunitPlatform.class)会将所有的报告都进入到stdout。
```java
class TestInfoDemo {

    @BeforeEach
    void init(TestInfo testInfo) {
        String displayName = testInfo.getDisplayName();
        assertTrue(displayName.equals("TEST 1") || displayName.equals("test2()"));
    }

    @Test
    @DisplayName("TEST 1")
    @Tag("my tag")
    void test1(TestInfo testInfo) {
        assertEquals("TEST 1", testInfo.getDisplayName());
        assertTrue(testInfo.getTags().contains("my tag"));
    }
}
class TestReporterDemo {

    @Test
    void reportSingleValue(TestReporter testReporter) {
        testReporter.publishEntry("a status message");
    }

    @Test
    void reportKeyValuePair(TestReporter testReporter) {
        testReporter.publishEntry("a key", "a value");
    }

    @Test
    void reportMultipleKeyValuePairs(TestReporter testReporter) {
        Map<String, String> values = new HashMap<>();
        values.put("user name", "dk38");
        values.put("award year", "1974");

        testReporter.publishEntry(values);
    }

}
```

# 接口默认方法（Interface Default Methods）
Junit Jupiter运行@Test，@TestFactory，@BeforeEach和@AfterEach称为接口默认的方法。一个可能的使用场景，我们可以为接口写测试方法了。

# 动态测试（Dynamic Tests）
动态测试是指可以通过@TestFactory注解的工厂方法在运行时产生的测试类型。@TestFactory并不是测试本身，而是一个测试单元的工厂。一个动态的测试方法就是这个工厂的产品。@TestFactory方法必须返回Stream, Collection, Iterable 或者 DynamicTest 实例。这些动态实例会被懒加载，确保动态和不确定的测试实例产生。

DynamicTest 是运行时生成的测试. 它由 展示名称(display name) 和 Executable组成. Executable 是一个 @FunctionalInterface的实现,这意味着 这些动态测试的实现可以通过 lambda expressions 或者 方法引用。
```java
class DynamicTestsDemo {

    // This will result in a JUnitException!
    @TestFactory
    List<String> dynamicTestsWithInvalidReturnType() {
        return Arrays.asList("Hello");
    }

    @TestFactory
    Collection<DynamicTest> dynamicTestsFromCollection() {
        return Arrays.asList(
            dynamicTest("1st dynamic test", () -> assertTrue(true)),
            dynamicTest("2nd dynamic test", () -> assertEquals(4, 2 * 2))
        );
    }

    @TestFactory
    Iterable<DynamicTest> dynamicTestsFromIterable() {
        return Arrays.asList(
            dynamicTest("3rd dynamic test", () -> assertTrue(true)),
            dynamicTest("4th dynamic test", () -> assertEquals(4, 2 * 2))
        );
    }

    @TestFactory
    Iterator<DynamicTest> dynamicTestsFromIterator() {
        return Arrays.asList(
            dynamicTest("5th dynamic test", () -> assertTrue(true)),
            dynamicTest("6th dynamic test", () -> assertEquals(4, 2 * 2))
        ).iterator();
    }

    @TestFactory
    Stream<DynamicTest> dynamicTestsFromStream() {
        return Stream.of("A", "B", "C").map(
            str -> dynamicTest("test" + str, () -> { /* ... */ }));
    }

    @TestFactory
    Stream<DynamicTest> dynamicTestsFromIntStream() {
        // Generates tests for the first 10 even integers.
        return IntStream.iterate(0, n -> n + 2).limit(10).mapToObj(
            n -> dynamicTest("test" + n, () -> assertTrue(n % 2 == 0)));
    }

    @TestFactory
    Stream<DynamicTest> generateRandomNumberOfTests() {

        // Generates random positive integers between 0 and 100 until
        // a number evenly divisible by 7 is encountered.
        Iterator<Integer> inputGenerator = new Iterator<Integer>() {

            Random random = new Random();
            int current;

            @Override
            public boolean hasNext() {
                current = random.nextInt(100);
                return current % 7 != 0;
            }

            @Override
            public Integer next() {
                return current;
            }
        };

        // Generates display names like: input:5, input:37, input:85, etc.
        Function<Integer, String> displayNameGenerator = (input) -> "input:" + input;

        // Executes tests based on the current input value.
        Consumer<Integer> testExecutor = (input) -> assertTrue(input % 7 != 0);

        // Returns a stream of dynamic tests.
        return DynamicTest.stream(inputGenerator, displayNameGenerator, testExecutor);
    }

}
```

- 第一种方法返回无效的返回类型。由于在编译时无法检测到无效的返回类型，因此在运行时检测到JUnitException时会抛出该异常
- 接下来的五个方法是非常简单的示例，它们演示了DynamicTest实例的Collection，Iterable，Iterator或Stream的生成。
- 最后一种方法本质上是真正动态的。 generateRandomNumberOfTests（）实现一个Iterator，该Iterator生成随机数，一个显示名称生成器和一个测试执行器，然后将这三个都提供给DynamicTest.stream（）。 尽管generateRandomNumberOfTests（）的不确定性行为当然与测试可重复性冲突，因此应谨慎使用，但它可以证明动态测试的表现力和力量。