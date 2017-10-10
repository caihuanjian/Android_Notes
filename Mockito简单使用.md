# Mockito简单使用

## 添加依赖

```java
dependencies {
        // 如果要使用Mockito，你需要添加此条依赖库
        testCompile 'org.mockito:mockito-core:1.+'
        // 如果你要使用Mockito 用于 Android instrumentation tests，那么需要你添加以下三条依赖库
        androidTestCompile 'org.mockito:mockito-core:1.+'
        androidTestCompile "com.google.dexmaker:dexmaker:1.2"
        androidTestCompile "com.google.dexmaker:dexmaker-mockito:1.2"
    }
```

## 使用mockito大致步骤
1. 使用 mockito 生成 Mock 对象
2. 定义 Mock 对象的行为和输出
3. 调用 Mock 对象方法进行单元测试
4. 对 Mock 对象的行为进行验证

## mockito两大作用
1. 指定方法返回值
> **如不指定，所有非void方法都返回默认值，int、long类型方法将返回0，boolean方法将返回false，对象方法将返回null等等**
2. 执行特定的动作

## 创建Mockito
1. 使用 MockitoAnnotations.initMocks(this) 方式 （推荐）
    ```java
    public class TestUserLogin {

            @Mock
            User user;

            @Before
            public void setupUser(){
                MockitoAnnotations.initMocks(this);
            }

            @Test
            public void testIsNotNull(){
                assertNotNull(user);
            }
        }
    ```
2. 使用 @RunWith(MockitoJUnitRunner.class) 方式
    ```java
    @RunWith(MockitoJUnitRunner.class)
    public class TestUserLogin {
    
        @Mock
        User user;
    
        @Test
        public void testLogin() {
            Assert.assertFalse(user.isLogin());
            Mockito.when(user.isLogin()).thenReturn(true);
            Assert.assertTrue(user.isLogin());
        }
    }
    ```

## 常用验证方式

1. 验证方法调用次数，并且参数正确
    ```java
    @Test
    public void testParams() {
        user.setUserName("jack");
        verify(user, times(1)).setUserName(matches("jack"));//验证调用了一次setUserName并且参数是jack
    }
    
    ```
    如果不关心方法参数可以使用anyInt,anyLong,anyDouble等等。anyObject表示任何对象，any(clazz)表示任何属于clazz的对象
    
2. 设置mock的某个行为
    ```java
    @Test
    public void testLogin() {
        Assert.assertFalse(user.isLogin());
        Mockito.when(user.isLogin()).thenReturn(true);//当调用isLogin方法时返回true
        Assert.assertTrue(user.isLogin());//由于上一句代码设置了，所以这里成立
    }
     
    ```
## 局部模拟
mock对象所有使用的方法全是mock的，**mock对象的所有方法调用都没有真正运行方法中的代码逻辑**。但是有时进行测试的时候，假如我们并不希望模拟使用对象的所有方法，只希望模拟对象的部分方法，剩下的方法依然能够正常的运行其内部的逻辑，因为这些方法中并没有对其他模块或者外部资源的依赖。这时我们有两种选择：

1. 一种是创建一个mock对象，**对某些特定需要调用真实逻辑的方法，告诉mockito使用真实的方法，即使用doCallRealMethod，或者使用mock(Class,CALLS_REAL_METHODS)构造mock对象**，这样构造出的mock对象默认情况下会执行对象方法的真实逻辑。
```java
public class Jerry {
    public void goHome() {
        System.out.println("go home");
        doSomeThingA();
        doSomeThingB();
    }

    public void doSomeThingB() {
        System.out.println("doSomeThingB");
    }

    public void doSomeThingA() {
        System.out.println("doSomeThingA");
    }

    public boolean go() {
        System.out.println("I say go go go!!");
        return true;
    }
}
```
####　测试方法1
```java
    @Test
    public void callRealMethodTest() {

        Jerry jerry = Mockito.mock(Jerry.class);
        Mockito.doCallRealMethod().when(jerry).goHome();
//        Mockito.doCallRealMethod().when(jerry).doSomeThingA();
        Mockito.doCallRealMethod().when(jerry).doSomeThingB();

        jerry.goHome();

        Mockito.verify(jerry).goHome();
        Mockito.verify(jerry, Mockito.times(1)).doSomeThingA();
        Mockito.verify(jerry, Mockito.times(1)).doSomeThingB();
    }
```
结果打印go home doSomeThingB，可见goHome与doSomeThingA方法调用了真实逻辑

#### 使用mock(Class,CALLS_REAL_METHODS)
```java
    @Test
    public void AllCallReallMethodTest() {
        Jerry jerry = Mockito.mock(Jerry.class, Mockito.CALLS_REAL_METHODS);
        jerry.goHome();
        Mockito.verify(jerry).goHome();
        Mockito.verify(jerry).doSomeThingA();
        Mockito.verify(jerry).doSomeThingB();
    }
```
结果：go home
doSomeThingA
doSomeThingB，说明该方式方法都调用了真实逻辑

2. 另一种是**创建一个类似的真实对象--spy对象**，spy对象默认情况执行对象方法的真实逻辑。

```java
    @Test
    public void spyTest() {
        Jerry spyJack = Mockito.spy(Jerry.class);

        spyJack.goHome();
        Mockito.verify(spyJack).goHome();
        Mockito.verify(spyJack, Mockito.times(1)).doSomeThingA();
        Mockito.verify(spyJack, Mockito.times(1)).doSomeThingB();
    }
```
结果：go home
doSomeThingA
doSomeThingB
> PS:doReturn方法直接返回，不会走方法体，thenRetrun会走方法体
```java
 // 当需要整体执行真正部分，只有少部分方法执行mock，选用这种方式
    @Test
    public void spyReturnTest() {
        Jerry spyJack = Mockito.spy(new Jerry());
        // 用thenReturn 会走go()方法体，然后将返回值Mock掉
        when(spyJack.go()).thenReturn(false);
        Assert.assertFalse(spyJack.go());
        // 用doReturn 不走go()方法体
        Mockito.doReturn(false).when(spyJack).go();
        Assert.assertFalse(spyJack.go());
    }
```
结果输出：I say go go go!!

---

在此输入正文




