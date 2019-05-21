---
title: jinja 配置文件渲染模板
categories :
- 技术
tags :
- python
- jinja
---

##### 通过meta.yaml里的配置文件,渲染模板文件gs.tpl生成gs.json
 - meta.yaml配置文件内容如下:
```
secs:
 - id: 1
   user: user1
   pwd: pwd1
   info:
       gender: male

 - id: 2
   user: user2
   pwd: pwd2
   info:
       gender: female
```
 - gs.tpl模板文件内容:
```
{
    {%-  for sec in cfg.secs    %}
    "{{sec.id}}":{
        "user":"{{sec.user}}",
        "pwd":"{{sec.pwd}}",
        "info":"{{sec.info}}"
    }
    {%- endfor %}
}
```
 - 最终生成的gs.json文件:
```
{
    "1":{
        "user":"user1",
        "pwd":"pwd1",
        "info":"{'gender': 'male'}"
    }
    "2":{
        "user":"user2",
        "pwd":"pwd2",
        "info":"{'gender': 'female'}"
    }
}
```
 - python代码:
```
#!/usr/bin/python
#. -*- coding: UTF-8 -*-
#读取meta.yaml  并且渲染gs.tpl

import ConfigParser
import os
import yaml
from jinja2 import Template
from jinja2 import Environment
from jinja2 import FileSystemLoader

#os.chdir() 方法用于改变当前工作目录到指定的路径。
#os.getcwd()  获取文件路径
#对于本例,文件都在同一文件夹下,下面这行代码其实没意义
os.chdir(os.getcwd())

yamlpath = 'meta.yaml'
config = yaml.load(file(yamlpath,'r'))

env = Environment( loader = FileSystemLoader(os.getcwd()))
template = env.get_template('gs.tpl')
result =   template.render(cfg=config)

fp = open('gs.json','w')
fp.write(result)
```
