---
title: Java Python 速度对比
categories :
- 技术
tags :
- Java
---

### 1.循环
Java  5ms
```
  public static void main(String[] args) {
        long t1 = System.currentTimeMillis();
        int count = 0;
        for (int i = 0; i < 100000000; i++) {

        }
        long t2 = System.currentTimeMillis();
        System.out.println(t2 - t1);
    }
```



python  12553ms
```
while i <100000000:
	i=i+1
end = int(round(1000*time.time()))
```

python循环特别慢

------------

### 2.循环处理字符串(循环次数少个0)

Java 21193ms
```
String str = "20492\tgame1\t3\tgame1#20492_3\t120\t0\t10.2.178.137\t10.13.4\t24013\t\t\t{\"data\":{\"rewardList\":[{\"amount\":5,\"code\":\"1\",\"currencyType\":\"DIAMOND\",\"subType\":0,\"tag\":\"DIAMOND\",\"type\":\"CURRENCY\",\"typeValue\":1},{\"amount\":1,\"code\":\"it_220001\",\"itemConfigId\":\"it_220001\",\"limit\":0,\"tag\":\"it_220001\",\"type\":\"ITEM\",\"typeValue\":2},{\"amount\":1,\"code\":\"it_209101\",\"itemConfigId\":\"it_209101\",\"limit\":0,\"tag\":\"it_209101\",\"type\":\"ITEM\",\"typeValue\":2}]},\"name\":\"6ZKp6ZWw6Zy45Li75LmU5qOu\",\"action\":\"TeamBattleHandler.endBattle\",\"client\":{\"isEscape\":0,\"battleData\":{\"blockId\":\"team_001\",\"scriptVersion\":\"dev_3\",\"operate\":{\"myOps\":[[[{\"a\":[\"f1\",\"e5\"],\"c\":1},{\"a\":[3,\"f1\"],\"c\":2}],[{\"a\":[\"f1\",\"e5\"],\"c\":1}]]],\"enemyOps\":[[[{\"a\":[\"e5\",\"f1\"],\"c\":1},{\"a\":[3,\"e5\"],\"c\":2}]]]},\"enemyInfo\":{\"monsterId\":\"team_001\",\"hero\":{\"5\":{\"bss\":{},\"blockdeepen\":0.5,\"angerbase\":1,\"hp\":12218.0,\"cure\":1.0,\"dp\":[\"sk_heroNormal_sp\"],\"uuid\":\"9F5rOHXh\",\"anger\":1,\"speed\":1.0,\"unhurt\":0.0,\"drain\":0.0,\"uncrit\":0.0,\"unblock\":0.0,\"dataId\":\"he_100001\",\"effecthit\":0.0,\"defense\":0.0,\"attack\":757.0,\"skill\":{\"1\":{\"level\":1,\"id\":\"sk_hero001_n\"},\"2\":{\"level\":1,\"id\":\"sk_hero001_f\"}},\"skillhurt\":0.0,\"block\":0.05,\"id\":\"he_100001\",\"angerMax\":2,\"antihurt\":0.0,\"sp\":{},\"critdeepen\":1.5,\"star\":1,\"level\":31,\"dotunhurt\":0.0,\"speeduprate\":0.0,\"effectdodge\":0.0,\"quality\":31,\"crit\":0.05,\"dothurt\":0.0,\"defrate\":1.0,\"hurt\":0.0,\"atkrate\":1.0,\"skillunhurt\":0.0}}},\"dataVersion\":\"dev_3\",\"randomseed\":781938,\"myInfo\":{\"hero\":{\"1\":{\"bss\":{},\"blockdeepen\":0.5,\"hp\":44054.0,\"fp\":{},\"cure\":1.0,\"dp\":[\"sk_heroNormal_sp\"],\"uuid\":\"Ql4JTCrg\",\"speed\":87.0,\"unhurt\":0.0,\"drain\":0.0,\"uncrit\":0.0,\"unblock\":0.0,\"effecthit\":0.0,\"defense\":1796.0,\"attack\":7442.0,\"rankLevel\":1,\"skill\":{\"1\":{\"level\":1,\"id\":\"sk_hero001_n\"},\"2\":{\"level\":1,\"id\":\"sk_hero001_f\"},\"3\":{\"level\":1,\"id\":\"sk_hero001_pp\"}},\"talent\":{\"1\":{\"level\":1,\"id\":\"sk_hero001_starUnlock_1\"},\"2\":{\"level\":1,\"id\":\"sk_hero001_starUnlock_3\"},\"3\":{\"level\":1,\"id\":\"sk_hero001_starUnlock_4\"}},\"skillhurt\":0.0,\"block\":0.05,\"id\":\"he_100001\",\"antihurt\":0.0,\"sp\":{},\"critdeepen\":1.5,\"star\":5,\"level\":120,\"dotunhurt\":0.0,\"speeduprate\":0.0,\"ep\":[],\"effectdodge\":0.0,\"quality\":40,\"crit\":0.17,\"dothurt\":0.0,\"defrate\":1.0,\"hurt\":0.0,\"atkrate\":1.0,\"skillunhurt\":0.0}}}},\"monsterId\":\"team_001\",\"isWin\":true},\"diff\":{\"wallet\":{\"moneys\":{\"diamond\":{\"amount\":10001035},\"challengevolume\":{\"amount\":2}}},\"teamBattleInfo\":{\"battle\":false,\"monsters\":{},\"battleCount\":1},\"record\":{\"map\":{\"2008\":1,\"2016\":1,\"2025\":1,\"1010\":10001055}},\"itemBag\":{\"items\":{\"it_209101\":{\"amount\":5002},\"it_220001\":{\"amount\":4810}}},\"lastRefreshTime\":1546839201057,\"taskProgress\":{\"workingTasks\":{\"Daily_TeamBattleComplete\":{\"cv\":1,\"s\":\"FINISH_NOT_DRAW\"}}}},\"time\":\"2019-01-07 13:33:21\"}\tD1497C49EC4756B78E0E1AD0DBFC6E83\t1546839201";
        for (int i = 0; i < 10000000; i++) {
            if (str.contains("it_220001")) {
                int index = str.indexOf("speeduprate");
                String str2 = str.substring(index, index + 10);
            }
            if (str.contains("_220001")) {
                int index = str.indexOf("speeduprate");
                String str2 = str.substring(index, index + 10);
            }
            if (str.contains("t_220001")) {
                int index = str.indexOf("speeduprate");
                String str2 = str.substring(index, index + 10);
            }

        }
```

python 41043ms
```
strs = "20492\tgame1\t3\tgame1#20492_3\t120\t0\t10.2.178.137\t10.13.4\t24013\t\t\t{\"data\":{\"rewardList\":[{\"amount\":5,\"code\":\"1\",\"currencyType\":\"DIAMOND\",\"subType\":0,\"tag\":\"DIAMOND\",\"type\":\"CURRENCY\",\"typeValue\":1},{\"amount\":1,\"code\":\"it_220001\",\"itemConfigId\":\"it_220001\",\"limit\":0,\"tag\":\"it_220001\",\"type\":\"ITEM\",\"typeValue\":2},{\"amount\":1,\"code\":\"it_209101\",\"itemConfigId\":\"it_209101\",\"limit\":0,\"tag\":\"it_209101\",\"type\":\"ITEM\",\"typeValue\":2}]},\"name\":\"6ZKp6ZWw6Zy45Li75LmU5qOu\",\"action\":\"TeamBattleHandler.endBattle\",\"client\":{\"isEscape\":0,\"battleData\":{\"blockId\":\"team_001\",\"scriptVersion\":\"dev_3\",\"operate\":{\"myOps\":[[[{\"a\":[\"f1\",\"e5\"],\"c\":1},{\"a\":[3,\"f1\"],\"c\":2}],[{\"a\":[\"f1\",\"e5\"],\"c\":1}]]],\"enemyOps\":[[[{\"a\":[\"e5\",\"f1\"],\"c\":1},{\"a\":[3,\"e5\"],\"c\":2}]]]},\"enemyInfo\":{\"monsterId\":\"team_001\",\"hero\":{\"5\":{\"bss\":{},\"blockdeepen\":0.5,\"angerbase\":1,\"hp\":12218.0,\"cure\":1.0,\"dp\":[\"sk_heroNormal_sp\"],\"uuid\":\"9F5rOHXh\",\"anger\":1,\"speed\":1.0,\"unhurt\":0.0,\"drain\":0.0,\"uncrit\":0.0,\"unblock\":0.0,\"dataId\":\"he_100001\",\"effecthit\":0.0,\"defense\":0.0,\"attack\":757.0,\"skill\":{\"1\":{\"level\":1,\"id\":\"sk_hero001_n\"},\"2\":{\"level\":1,\"id\":\"sk_hero001_f\"}},\"skillhurt\":0.0,\"block\":0.05,\"id\":\"he_100001\",\"angerMax\":2,\"antihurt\":0.0,\"sp\":{},\"critdeepen\":1.5,\"star\":1,\"level\":31,\"dotunhurt\":0.0,\"speeduprate\":0.0,\"effectdodge\":0.0,\"quality\":31,\"crit\":0.05,\"dothurt\":0.0,\"defrate\":1.0,\"hurt\":0.0,\"atkrate\":1.0,\"skillunhurt\":0.0}}},\"dataVersion\":\"dev_3\",\"randomseed\":781938,\"myInfo\":{\"hero\":{\"1\":{\"bss\":{},\"blockdeepen\":0.5,\"hp\":44054.0,\"fp\":{},\"cure\":1.0,\"dp\":[\"sk_heroNormal_sp\"],\"uuid\":\"Ql4JTCrg\",\"speed\":87.0,\"unhurt\":0.0,\"drain\":0.0,\"uncrit\":0.0,\"unblock\":0.0,\"effecthit\":0.0,\"defense\":1796.0,\"attack\":7442.0,\"rankLevel\":1,\"skill\":{\"1\":{\"level\":1,\"id\":\"sk_hero001_n\"},\"2\":{\"level\":1,\"id\":\"sk_hero001_f\"},\"3\":{\"level\":1,\"id\":\"sk_hero001_pp\"}},\"talent\":{\"1\":{\"level\":1,\"id\":\"sk_hero001_starUnlock_1\"},\"2\":{\"level\":1,\"id\":\"sk_hero001_starUnlock_3\"},\"3\":{\"level\":1,\"id\":\"sk_hero001_starUnlock_4\"}},\"skillhurt\":0.0,\"block\":0.05,\"id\":\"he_100001\",\"antihurt\":0.0,\"sp\":{},\"critdeepen\":1.5,\"star\":5,\"level\":120,\"dotunhurt\":0.0,\"speeduprate\":0.0,\"ep\":[],\"effectdodge\":0.0,\"quality\":40,\"crit\":0.17,\"dothurt\":0.0,\"defrate\":1.0,\"hurt\":0.0,\"atkrate\":1.0,\"skillunhurt\":0.0}}}},\"monsterId\":\"team_001\",\"isWin\":true},\"diff\":{\"wallet\":{\"moneys\":{\"diamond\":{\"amount\":10001035},\"challengevolume\":{\"amount\":2}}},\"teamBattleInfo\":{\"battle\":false,\"monsters\":{},\"battleCount\":1},\"record\":{\"map\":{\"2008\":1,\"2016\":1,\"2025\":1,\"1010\":10001055}},\"itemBag\":{\"items\":{\"it_209101\":{\"amount\":5002},\"it_220001\":{\"amount\":4810}}},\"lastRefreshTime\":1546839201057,\"taskProgress\":{\"workingTasks\":{\"Daily_TeamBattleComplete\":{\"cv\":1,\"s\":\"FINISH_NOT_DRAW\"}}}},\"time\":\"2019-01-07 13:33:21\"}\tD1497C49EC4756B78E0E1AD0DBFC6E83\t1546839201"; 
i=0
while i <10000000:
	i=i+1
	if 'it_220001' in strs:
		index = strs.find('speeduprate')
		str2 = strs[index:index+10]
	if '_220001' in strs:
		index = strs.find('speeduprate')
		str2 = strs[index:index+10]
	if 't_220001' in strs:
		index = strs.find('speeduprate')
		str2 = strs[index:index+10]
```

python 字符串处理比java快特别多

--------

### 3.读取文件

Java 228707
```
String path = "/Volumes/macwin/action日志/action_log_2018-12-05.log";
        long t1 = System.currentTimeMillis();
        String str;
        BufferedReader br = new BufferedReader(new FileReader(path));
        while ((str = br.readLine()) != null) {

        }
```

python 228838
```
path = "/Volumes/macwin/action日志/action_log_2018-12-05.log"
with open(path,'r') as f:
	for line in f:
		pass
```

-----

### 4.NIO MappedBuffer
```
   private static final int LEN = 2047483646;

    public static void main(String[] args) {
        int line = 0;
        try {

            FileChannel fc = new FileInputStream(path).getChannel();

            BufferedWriter bw = new BufferedWriter(new FileWriter(path2));
            long t1 = System.currentTimeMillis();
            byte[] bs;
            byte[] temp = new byte[0];
            String str;
            Map<String, Integer> map = Maps.newHashMap();

            long prePos = 0;

            int count = (int) (fc.size() / LEN + 1);
            int offset = LEN;
            long size = fc.size();

            for (int c = 0; c < count; c++) {
                prePos += offset;
                if (size - prePos < LEN) {
                    offset = (int) (size - prePos - 1);
                    System.out.println("=====" + offset);
                }
                MappedByteBuffer mappedByteBuffer = fc.map(FileChannel.MapMode.READ_ONLY, prePos,
                        offset);
                bs = new byte[offset];
                mappedByteBuffer.get(bs);
                mappedByteBuffer.clear();
                int start = 0;
                for (int i = 0; i < bs.length; i++) {
                    if (bs[i] == 10) {
                        byte[] toTemp = new byte[temp.length + i - start];
                        System.arraycopy(temp, 0, toTemp, 0, temp.length);
                        System.arraycopy(bs, start, toTemp, temp.length, i - start);
                        start = i;
                        temp = new byte[0];
                        str = new String(toTemp);
                        line++;
                        if (line >= 500000) {
                            throw new RuntimeException();
                        }

                    }
                }
                //将最后不是一行的字符存起来下一次读取的时候用
                if (start < bs.length - 1) {
                    temp = new byte[bs.length - 1 - start];
                    System.arraycopy(bs, start, temp, 0, bs.length - 1 - start);
                }
            }
            long t2 = System.currentTimeMillis();
            System.out.println("\n" + (t2 - t1));
```
-Xmx2048m -Xms2048m
无论怎么调缓冲区大小,最快是800000时需要两百多秒读完
可能是因为得重复将文件映射到内存里

但是,换了个200m的文件,mappedBuffer 一次读fc.size()时1.7秒 ;一次读10000左右时最快1.5秒
bufferredReader.readline():  1.4~1.7秒
python 1秒! 偶尔2秒


-------
###Tips:

1.python 的 str in list 数据量大时比较慢
