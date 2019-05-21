---
title: JsonField标签
categories:
- 技术
tags:
- Java

---

Test.java
```
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.annotation.JSONField;
public class Test {
    @JSONField(name = "mingzi")
    String name;
    @JSONField(name = "ling")
    int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return JSONObject.toJSONString(this);
    }
}
```
--------
Test3.java
```
import com.alibaba.fastjson.JSONObject;

public class Test3 {

    public static void main(String[] args) {
        Test test = new Test();
        test.setAge(20);
        test.setName("tom");

        JSONObject jsonObject = (JSONObject) JSONObject.toJSON(test);

        System.out.println(jsonObject.get("mingzi"));
        System.out.println(jsonObject.get("name"));
    }
}
```
--------
输出：
tom
null





