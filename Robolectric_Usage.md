# Android单元测试记录
## JUnit
单元测试分三个步骤
1. setup，即new出待测试的类
2. 执行待测试的方法，得到结果
3. 验证结果

### 几个注解
|注解|说明|
| --- | ---- |
|@Test| 修饰测试方法|
|@Before　  |被该注解修饰的方法会在测试方法调用前调用
|@after   |   被该注解修饰的方法会在测试方法调用后调用
|@BeforeClass |被该注解修饰的方法在测试类的所有方法运行前调用，必须是静态方法
|@AfterClass |被该注解修饰的方法在测试类的所有方法运行后调用，必须是静态方法，可用于释放资源
|@ignore | 所修饰的测试方法运行时被忽略

### assert
| 名称 | 备注 |
| --- | --- |
|assertEquals(expected, actual)|验证expected的值跟actual是一样的
|assertEquals(expected, actual, tolerance)|如果两个数的差异在这个偏差值之内，则测试通过
|assertTrue(boolean condition)|验证contidion的值是true
|assertNull(Object obj)|验证obj的值是null
|assertSame(expected, actual)|验证expected和actual是同一个对象，即指向同一个对象
|fail()|让测试方法失败

## 单元测试测试什么方法
1. 所有的Model、Presenter/ViewModel、Api、Utils等类的public方法
2. Data类除了getter、setter、toString、hashCode等一般自动生成的方法之外的逻辑部分
自定义View的功能：比如setdata以后，text有没有显示出来等等，简单的交互，比如click事件，负责的交互一般不测，比如touch、滑动事件等等。
3. Activity的主要功能：比如view是不是存在、显示数据、错误信息、简单的点击事件等。比较复杂的用户交互比如onTouch，以及view的样式、位置等等可以不测。因为不好测。

## Robolectric
### 配置
>注意将菜单栏选择 Run -> Edit Configuration -> Defaults -> JUnit，在Configuration tab将working directory改成 \$MODULE_DIR$ 

- @config配置
1. 可以通过@Config定制Robolectric的运行时的行为。这个注解可以用来注释类和方法，如果类和方法同时使用了@Config，那么方法的设置会覆盖类的设置。你可以创建一个基类，用@Config配置测试参数，这样，其他测试用例就可以共享这个配置了
2. 配置Application，默认会根据manifest去实例化一个类，如果想在测试用例中重新指定，使用@config（application = CustomApplication.class）
3. 配置sdk @config（sdk = 23）
4. 指定resource路径
    ```java
    @Config(manifest = "some/build/path/AndroidManifest.xml",
            assetDir = "some/build/path/assetDir",
            resourceDir = "some/build/path/resourceDir")
    public class SandwichTest {
    
        @Config(manifest = "other/build/path/AndroidManifest.xml")
        public void getSandwich_shouldReturnHamSandwich() {
        }
    }
    ```
5. 使用限定的资源文件
Android会在运行时加载特定的资源文件，如根据设备屏幕加载不同分辨率的图片资源、根据系统语言加载不同的string.xml，在Robolectric测试当中，你也可以进行一个限定，让测试程序加载特定资源.多个限定条件可以用破折号拼接在在一起。
    ```java
       /**
         * 使用qualifiers加载对应的资源文件
         *
         * @throws Exception
         */
        @Config(qualifiers = "zh-rCN")
        @Test
        public void testString() throws Exception {
            final Context context = RuntimeEnvironment.application;
            assertThat(context.getString(R.string.app_name), is("单元测试Demo"));
        }
    ```
6. Properties
可以把配置写进同一个robolectric.properties文件里，放到test/src/resources目录下，通过@config指定

### 驱动Activity生命周期
```java
    @Test
    public void testLifeCycle() {
        final ActivityController<MainActivity> mainActivityActivityController = Robolectric.buildActivity(MainActivity.class);
        final MainActivity mainActivity = mainActivityActivityController.get();
        mainActivityActivityController.create();
        Assert.assertTrue(mainActivity.isActivityCreate());
    }
```
> ps:要遵循Activity的生命周期
### 给启动的activity传递Intent
```java
    @Test
    public void testStartActivityWithIntent() throws Exception {
        Intent intent = new Intent();
        intent.putExtra("test", "HelloWorld");
        final MainActivity mainActivity = Robolectric.buildActivity(MainActivity.class).withIntent(intent).create().get();
        assertEquals("HelloWorld", mainActivity.getIntent().getExtras().getString("test"));
    }
```
### onRestoreInstanceState回调中传递Bundle
```java
    @Test
    public void testSavedInstanceState() throws Exception {
        Bundle savedInstanceState = new Bundle();
        savedInstanceState.putString("state","rain");
        final ActivityController<MainActivity> controller = Robolectric.buildActivity(MainActivity.class);
        final MainActivity mainActivity = controller.create().restoreInstanceState(savedInstanceState).get();
        assertEquals("rain",mainActivity.getState());
    }
```
### 测试广播、toast
```java
    @Test
    public void restReceive() throws Exception {

        String action = "com.rain.unittest.receiver";
        Intent intent = new Intent(action);
        intent.putExtra("user", "hwanj");

        MyReceviver myReceiver = new MyReceviver();
        //确认没弹
        Toast latestToast = ShadowToast.getLatestToast();
        Assert.assertNull(latestToast);
        //调用onreceive
        myReceiver.onReceive(RuntimeEnvironment.application, intent);
        //确认弹了，并且传递的字符串正确
        latestToast = ShadowToast.getLatestToast();
        Assert.assertNotNull(latestToast);
        Assert.assertEquals("hwanj", ShadowToast.getTextOfLatestToast());
    }
```