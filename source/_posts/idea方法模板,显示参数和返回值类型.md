---
title: idea方法模板,显示参数和返回值类型
categories :
- 技术
tags :
- Java
- idea
---

之前配置的自定义模板方法参数和返回值类型一直显示不出来,如下:
```
/**
     * 通过怪物类型和属性基础值计算最终值
     *
     * @param
     * @return
     * @Author: shenpeng
     * @Date: 2019/1/14
     */
     private Map<String, Integer> getASHSfinByCfg(Map<String, Double> baseVals, String type,
            TKEquipEnemyType enemyType) {
```
是由于idea的模板设置时以`/**`开头,修改模板为
```
**
 * 
 * 
 * @param $params$
 * @return $returns$
 * @Author: shenpeng
 * @Date: $date$ 
 */ 
```
之后,快捷键还是设置为`cmt`,但是用的时候用`/cmt`把开头缺少的`/`补上就行了.
结果如下:
```
/**
     * 
     * 
     * @param [baseVals, type, enemyType]
     * @return java.util.Map<java.lang.String,java.lang.Integer>
     * @Author: shenpeng
     * @Date: 2019-02-26 
     */ 
```




