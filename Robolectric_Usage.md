# Android��Ԫ���Լ�¼
## JUnit
��Ԫ���Է���������
1. setup����new�������Ե���
2. ִ�д����Եķ������õ����
3. ��֤���

### ����ע��
|ע��|˵��|
| --- | ---- |
|@Test| ���β��Է���|
|@Before��  |����ע�����εķ������ڲ��Է�������ǰ����
|@after   |   ����ע�����εķ������ڲ��Է������ú����
|@BeforeClass |����ע�����εķ����ڲ���������з�������ǰ���ã������Ǿ�̬����
|@AfterClass |����ע�����εķ����ڲ���������з������к���ã������Ǿ�̬�������������ͷ���Դ
|@ignore | �����εĲ��Է�������ʱ������

### assert
| ���� | ��ע |
| --- | --- |
|assertEquals(expected, actual)|��֤expected��ֵ��actual��һ����
|assertEquals(expected, actual, tolerance)|����������Ĳ��������ƫ��ֵ֮�ڣ������ͨ��
|assertTrue(boolean condition)|��֤contidion��ֵ��true
|assertNull(Object obj)|��֤obj��ֵ��null
|assertSame(expected, actual)|��֤expected��actual��ͬһ�����󣬼�ָ��ͬһ������
|fail()|�ò��Է���ʧ��

## ��Ԫ���Բ���ʲô����
1. ���е�Model��Presenter/ViewModel��Api��Utils�����public����
2. Data�����getter��setter��toString��hashCode��һ���Զ����ɵķ���֮����߼�����
�Զ���View�Ĺ��ܣ�����setdata�Ժ�text��û����ʾ�����ȵȣ��򵥵Ľ���������click�¼�������Ľ���һ�㲻�⣬����touch�������¼��ȵȡ�
3. Activity����Ҫ���ܣ�����view�ǲ��Ǵ��ڡ���ʾ���ݡ�������Ϣ���򵥵ĵ���¼��ȡ��Ƚϸ��ӵ��û���������onTouch���Լ�view����ʽ��λ�õȵȿ��Բ��⡣��Ϊ���ò⡣

## Robolectric
### ����
>ע�⽫�˵���ѡ�� Run -> Edit Configuration -> Defaults -> JUnit����Configuration tab��working directory�ĳ� \$MODULE_DIR$ 

- @config����
1. ����ͨ��@Config����Robolectric������ʱ����Ϊ�����ע���������ע����ͷ����������ͷ���ͬʱʹ����@Config����ô���������ûḲ��������á�����Դ���һ�����࣬��@Config���ò��Բ������������������������Ϳ��Թ������������
2. ����Application��Ĭ�ϻ����manifestȥʵ����һ���࣬������ڲ�������������ָ����ʹ��@config��application = CustomApplication.class��
3. ����sdk @config��sdk = 23��
4. ָ��resource·��
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
5. ʹ���޶�����Դ�ļ�
Android��������ʱ�����ض�����Դ�ļ���������豸��Ļ���ز�ͬ�ֱ��ʵ�ͼƬ��Դ������ϵͳ���Լ��ز�ͬ��string.xml����Robolectric���Ե��У���Ҳ���Խ���һ���޶����ò��Գ�������ض���Դ.����޶��������������ۺ�ƴ������һ��
    ```java
       /**
         * ʹ��qualifiers���ض�Ӧ����Դ�ļ�
         *
         * @throws Exception
         */
        @Config(qualifiers = "zh-rCN")
        @Test
        public void testString() throws Exception {
            final Context context = RuntimeEnvironment.application;
            assertThat(context.getString(R.string.app_name), is("��Ԫ����Demo"));
        }
    ```
6. Properties
���԰�����д��ͬһ��robolectric.properties�ļ���ŵ�test/src/resourcesĿ¼�£�ͨ��@configָ��

### ����Activity��������
```java
    @Test
    public void testLifeCycle() {
        final ActivityController<MainActivity> mainActivityActivityController = Robolectric.buildActivity(MainActivity.class);
        final MainActivity mainActivity = mainActivityActivityController.get();
        mainActivityActivityController.create();
        Assert.assertTrue(mainActivity.isActivityCreate());
    }
```
> ps:Ҫ��ѭActivity����������
### ��������activity����Intent
```java
    @Test
    public void testStartActivityWithIntent() throws Exception {
        Intent intent = new Intent();
        intent.putExtra("test", "HelloWorld");
        final MainActivity mainActivity = Robolectric.buildActivity(MainActivity.class).withIntent(intent).create().get();
        assertEquals("HelloWorld", mainActivity.getIntent().getExtras().getString("test"));
    }
```
### onRestoreInstanceState�ص��д���Bundle
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
### ���Թ㲥��toast
```java
    @Test
    public void restReceive() throws Exception {

        String action = "com.rain.unittest.receiver";
        Intent intent = new Intent(action);
        intent.putExtra("user", "hwanj");

        MyReceviver myReceiver = new MyReceviver();
        //ȷ��û��
        Toast latestToast = ShadowToast.getLatestToast();
        Assert.assertNull(latestToast);
        //����onreceive
        myReceiver.onReceive(RuntimeEnvironment.application, intent);
        //ȷ�ϵ��ˣ����Ҵ��ݵ��ַ�����ȷ
        latestToast = ShadowToast.getLatestToast();
        Assert.assertNotNull(latestToast);
        Assert.assertEquals("hwanj", ShadowToast.getTextOfLatestToast());
    }
```