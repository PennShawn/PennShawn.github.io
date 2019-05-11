---
title: PowerMock
categories :
- 技术
tags :
- Java
- Mock
---

###1.为什么要用mock
在做单元测试的时候，我们会发现我们要测试的方法会引用很多外部依赖的对象，比如：（发送邮件，网络通讯，远程服务, 文件系统等等）。 而我们没法控制这些外部依赖的对象，为了解决这个问题，我们就需要用到Mock工具来模拟这些外部依赖的对象，来完成单元测试。
例如策划表没有配完,需要在调用如`RewardConfigHelper.getAndRewardPlayer()`这样的方法时不报错,模拟返回一个rewardList,就可以用mock.
###2.PowerMock
&nbsp;&nbsp;&nbsp;&nbsp;现如今比较流行的Mock工具如EasyMock、Mockito等都有一个共同的缺点：不能mock静态、final、私有方法等。而PowerMock能够完美的弥补以上三个Mock工具的不足。
&nbsp;&nbsp;&nbsp;&nbsp;<b>PowerMock是一个扩展了其它如EasyMock等mock框架的、功能更加强大的框架。PowerMock使用一个自定义类加载器和字节码操作来模拟静态方法，构造函数，final类和方法，私有方法，去除静态初始化器等等。通过使用自定义的类加载器，简化采用的IDE或持续集成服务器不需要做任何改变。熟悉PowerMock支持的mock框架的开发人员会发现PowerMock很容易使用，因为对于静态方法和构造器来说，整个的期望API是一样的。PowerMock旨在用少量的方法和注解扩展现有的API来实现额外的功能。目前PowerMock支持EasyMock和Mockito。</b>
###3.使用
PowerMock有两个重要的注解：

      –@RunWith(PowerMockRunner.class)

      –@PrepareForTest( { YourClassWithEgStaticMethod.class })

      如果你的测试用例里没有使用注解@PrepareForTest，那么可以不用加注解@RunWith(PowerMockRunner.class)，反之亦然。当你需要使用PowerMock强大功能（Mock静态、final、私有方法等）的时候，就需要加注解@PrepareForTest。
###3.原理
       •  当某个测试方法被注解@PrepareForTest标注以后，在运行测试用例时，会创建一个新的org.powermock.core.classloader.MockClassLoader实例，然后加载该测试用例使用到的类（系统类除外）。

       •   PowerMock会根据你的mock要求，去修改写在注解@PrepareForTest里的class文件（当前测试类会自动加入注解中），以满足特殊的mock需求。例如：去除final方法的final标识，在静态方法的最前面加入自己的虚拟实现等。

       •   如果需要mock的是系统类的final方法和静态方法，PowerMock不会直接修改系统类的class文件，而是修改调用系统类的class文件，以满足mock需求。
       
###4.Maven依赖
```
<dependency> 
<groupId>org.powermock</groupId> 
<artifactId>powermock-api-mockito</artifactId> 
<version>1.6.1</version> 
<scope>test</scope> 
</dependency> 

<dependency> 
<groupId>org.powermock</groupId> 
<artifactId>powermock-module-junit4</artifactId> 
<version>1.6.1</version> 
<scope>test</scope> 
</dependency> 
```
  

###5.示例
```
 @Test
    @PrepareForTest(RewardConfigHelper.class)
    public void testSettFrag() {
        mockStatic(RewardConfigHelper.class);
        List<RewardObject> rewardList = Lists.newArrayList();
        rewardList.add(new RewardObject());
        when(RewardConfigHelper.getAndRewardPlayer(any(Player.class), anyString(), anyInt(),
                eq(AddEnum.SELL_HERO_FRAG), eq(""))).thenReturn(rewardList);
        JSONObject param = new JSONObject();
        JSONObject frags = new JSONObject();
        String fragId = "it_220006";
        pr.getItemBag().addItem(pr, fragId, 3, AddEnum.TEST, "");
        frags.put(fragId, 3);
        param.put("frags", frags);
        JSONObject ret = (JSONObject) heroHandler.sellHeroFrag(pr, param);
        List<RewardObject> rewardObjects = (List<RewardObject>) ret.get("rewards");
        Assert.assertEquals(1, rewardObjects.size());
    }
```


###6.报错
```
java.lang.NoSuchMethodError: org.mockito.internal.handler.MockHandlerFactory.createMockHandler(Lorg/mockito/mock/MockCreationSettings;)Lorg/mockito/internal/InternalMockHandler;

	at org.powermock.api.mockito.internal.mockcreation.DefaultMockCreator.createMethodInvocationControl(DefaultMockCreator.java:114)
	at org.powermock.api.mockito.internal.mockcreation.DefaultMockCreator.createMock(DefaultMockCreator.java:69)
	at org.powermock.api.mockito.internal.mockcreation.DefaultMockCreator.mock(DefaultMockCreator.java:46)
	at org.powermock.api.mockito.PowerMockito.mock(PowerMockito.java:138)
	at com.playcrab.kos.shenpeng.HeroHandlerTest.testSettFrag(HeroHandlerTest.java:61)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.junit.internal.runners.TestMethod.invoke(TestMethod.java:68)
	at org.powermock.modules.junit4.internal.impl.PowerMockJUnit44RunnerDelegateImpl$PowerMockJUnit44MethodRunner.runTestMethod(PowerMockJUnit44RunnerDelegateImpl.java:326)
	at org.junit.internal.runners.MethodRoadie$2.run(MethodRoadie.java:88)
	at org.junit.internal.runners.MethodRoadie.runBeforesThenTestThenAfters(MethodRoadie.java:96)
	at org.powermock.modules.junit4.internal.impl.PowerMockJUnit44RunnerDelegateImpl$PowerMockJUnit44MethodRunner.executeTest(PowerMockJUnit44RunnerDelegateImpl.java:310)
	at org.powermock.modules.junit4.internal.impl.PowerMockJUnit47RunnerDelegateImpl$PowerMockJUnit47MethodRunner.executeTestInSuper(PowerMockJUnit47RunnerDelegateImpl.java:131)
	at org.powermock.modules.junit4.internal.impl.PowerMockJUnit47RunnerDelegateImpl$PowerMockJUnit47MethodRunner.access$100(PowerMockJUnit47RunnerDelegateImpl.java:59)
	at org.powermock.modules.junit4.internal.impl.PowerMockJUnit47RunnerDelegateImpl$PowerMockJUnit47MethodRunner$TestExecutorStatement.evaluate(PowerMockJUnit47RunnerDelegateImpl.java:147)
	at org.powermock.modules.junit4.internal.impl.PowerMockJUnit47RunnerDelegateImpl$PowerMockJUnit47MethodRunner.evaluateStatement(PowerMockJUnit47RunnerDelegateImpl.java:107)
	at org.powermock.modules.junit4.internal.impl.PowerMockJUnit47RunnerDelegateImpl$PowerMockJUnit47MethodRunner.executeTest(PowerMockJUnit47RunnerDelegateImpl.java:82)
	at org.powermock.modules.junit4.internal.impl.PowerMockJUnit44RunnerDelegateImpl$PowerMockJUnit44MethodRunner.runBeforesThenTestThenAfters(PowerMockJUnit44RunnerDelegateImpl.java:298)
	at org.junit.internal.runners.MethodRoadie.runTest(MethodRoadie.java:86)
	at org.junit.internal.runners.MethodRoadie.run(MethodRoadie.java:49)
	at org.powermock.modules.junit4.internal.impl.PowerMockJUnit44RunnerDelegateImpl.invokeTestMethod(PowerMockJUnit44RunnerDelegateImpl.java:218)
	at org.powermock.modules.junit4.internal.impl.PowerMockJUnit44RunnerDelegateImpl.runMethods(PowerMockJUnit44RunnerDelegateImpl.java:160)
	at org.powermock.modules.junit4.internal.impl.PowerMockJUnit44RunnerDelegateImpl$1.run(PowerMockJUnit44RunnerDelegateImpl.java:134)
	at org.junit.internal.runners.ClassRoadie.runUnprotected(ClassRoadie.java:33)
	at org.junit.internal.runners.ClassRoadie.runProtected(ClassRoadie.java:45)
	at org.powermock.modules.junit4.internal.impl.PowerMockJUnit44RunnerDelegateImpl.run(PowerMockJUnit44RunnerDelegateImpl.java:136)
	at org.powermock.modules.junit4.common.internal.impl.JUnit4TestSuiteChunkerImpl.run(JUnit4TestSuiteChunkerImpl.java:121)
	at org.powermock.modules.junit4.common.internal.impl.AbstractCommonPowerMockRunner.run(AbstractCommonPowerMockRunner.java:57)
	at org.powermock.modules.junit4.PowerMockRunner.run(PowerMockRunner.java:59)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:160)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:68)
	at com.intellij.rt.execution.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:47)
	at com.intellij.rt.execution.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:242)
	at com.intellij.rt.execution.junit.JUnitStarter.main(JUnitStarter.java:70)
```
是由于mockito版本为2.19.1
powermock版本1.7.4
需要提高powermock版本到2.0以上
```
https://github.com/mockito/mockito/issues/1207
@mockitoguy
 Member
mockitoguy commented on 15 Oct 2017
Sorry for the issue!

Please report it to PowerMock tracker. Mockito team does not support PowerMock integration.

The only way Mockito team can help Java developers for this use case is by providing comprehensive public API for framework integrations. This way, PowerMock can cleanly integrate with Mockito public API and avoid various method not found errors. We have added new public API in Mockito 2.10.0, now PowerMock needs to release a new version with this API.

Hope that helps clarifying the use case!
```


--------



```
org.mockito.exceptions.misusing.InvalidUseOfMatchersException: 
Invalid use of argument matchers!
5 matchers expected, 3 recorded:
-> at com.playcrab.kos.shenpeng.HeroHandlerTest.testSettFrag(HeroHandlerTest.java:64)
-> at com.playcrab.kos.shenpeng.HeroHandlerTest.testSettFrag(HeroHandlerTest.java:64)
-> at com.playcrab.kos.shenpeng.HeroHandlerTest.testSettFrag(HeroHandlerTest.java:64)

This exception may occur if matchers are combined with raw values:
    //incorrect:
    someMethod(anyObject(), "raw String");
When using matchers, all arguments have to be provided by matchers.
For example:
    //correct:
    someMethod(anyObject(), eq("String by matcher"));
```
-----
```
java.lang.LinkageError: loader constraint violation: loader (instance of org/powermock/core/classloader/MockClassLoader) previously initiated loading for a different type with name "javax/management/MBeanServer"
	at java.lang.ClassLoader.defineClass1(Native Method)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:763)
	at org.powermock.core.classloader.MockClassLoader.loadUnmockedClass(MockClassLoader.java:262)
	at org.powermock.core.classloader.MockClassLoader.loadModifiedClass(MockClassLoader.java:206)
	at org.powermock.core.classloader.DeferSupportingClassLoader.loadClass1(DeferSupportingClassLoader.java:89)
	at org.powermock.core.classloader.DeferSupportingClassLoader.loadClass(DeferSupportingClassLoader.java:79)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	at org.apache.commons.pool2.impl.BaseGenericObjectPool.jmxRegister(BaseGenericObjectPool.java:957)
	at org.apache.commons.pool2.impl.BaseGenericObjectPool.<init>(BaseGenericObjectPool.java:133)
	at org.apache.commons.pool2.impl.GenericObjectPool.<init>(GenericObjectPool.java:107)
	at redis.clients.util.Pool.initPool(Pool.java:44)
	at redis.clients.util.Pool.<init>(Pool.java:23)
	at redis.clients.jedis.JedisPool.<init>(JedisPool.java:185)
	at redis.clients.jedis.JedisPool.<init>(JedisPool.java:162)
	at redis.clients.jedis.JedisPool.<init>(JedisPool.java:144)
	at com.playcrab.cache.redis.RedisInstance.init(RedisInstance.java:48)
	at com.playcrab.cache.CacheModule.buildRedis(CacheModule.java:55)
	at com.playcrab.cache.CacheModule.init(CacheModule.java:34)
	at com.playcrab.common.agent.BaseAgent.init(BaseAgent.java:29)
	at com.playcrab.kos.agents.KOSGSAgent.init(KOSGSAgent.java:89)
	at com.playcrab.kos.shenpeng.shenpengTestBase.initServer(shenpengTestBase.java:35)
	at com.playcrab.kos.shenpeng.HeroHandlerTest.beforeClass(HeroHandlerTest.java:38)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.junit.internal.runners.ClassRoadie.runBefores(ClassRoadie.java:57)
	at org.junit.internal.runners.ClassRoadie.runProtected(ClassRoadie.java:44)
	at org.powermock.modules.junit4.internal.impl.PowerMockJUnit44RunnerDelegateImpl.run(PowerMockJUnit44RunnerDelegateImpl.java:136)
	at org.powermock.modules.junit4.common.internal.impl.JUnit4TestSuiteChunkerImpl.run(JUnit4TestSuiteChunkerImpl.java:121)
	at org.powermock.modules.junit4.common.internal.impl.AbstractCommonPowerMockRunner.run(AbstractCommonPowerMockRunner.java:57)
	at org.powermock.modules.junit4.PowerMockRunner.run(PowerMockRunner.java:59)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:160)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:68)
	at com.intellij.rt.execution.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:47)
	at com.intellij.rt.execution.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:242)
	at com.intellij.rt.execution.junit.JUnitStarter.main(JUnitStarter.java:70)
```
需要加上`@PowerMockIgnore("javax.management.*")`防止类加载器冲突

-----


<b>如果用matcher,所有参数都要用matcher</b>
```
when(RewardConfigHelper.getAndRewardPlayer(any(Player.class), anyString(), anyInt(),eq(AddEnum.SELL_HERO_FRAG), eq(""))).thenReturn(rewardList);
```

------

<b>不要忘了
`@RunWith(PowerMockRunner.class)   `
和
`@PrepareForTest(RewardConfigHelper.class)`
</b>



参考来源:https://www.cnblogs.com/dengshihuang/p/7903642.html
