---
title: Mockito的另一种应用
categories :
- 技术
tags :
- Java
- Mock
---

除了模拟静态方法的返回值,如RewardConfigHelper.getAndRewardPlayer()在策划没配置奖励时也有返回值,mockito还可以模拟对象的方法.
例如,有一个需求是分享琦玉照片时领取奖励,需要编写测试用例模拟分享琦玉照片.但是新建分享之前需要玩家照片背包里有琦玉照片,当不方便手动给玩家加照片时,可以模拟判断玩家背包里是否有照片的方法,如`pr.getSaitama().getPhotoSys().hasPhoto(String picId);`
```
@Test
    //    @PrepareForTest(SaitamaPhotoSys.class)
    public void testShareSaitamaPic() {
        SaitamaPhotoSys saitamaPhotoSys = pr.getSaitama().getPhotoSys();
        SaitamaPhotoSys mock = spy(saitamaPhotoSys);
        PowerMockito.when(mock.hasPhoto(anyString())).thenReturn(true);

        pr.getSaitama().setPhotoSys(mock);
        pr.setLevel(119);
        pr.levelup();
        params.put("picId", KOSDataConfigService.getFirstId(SaitamaPicture.class));
        Assert.assertTrue(handler.shareSaitamaPic(pr, params).size() > 0);
    }
```
给mock设置好返回值后,` pr.getSaitama().setPhotoSys(mock);`,这样在`handler.shareSaitamaPic(pr, params)`时才会返回设置好的结果.


