---
title: 《Effective Java》Item11.Always override hashCode when you override equals
categories :
- 技术
tags :
- Java
- Effective Java
---

You must override hashcode in every class that overrides equals. If you fail to do so, your class will violate the general contract for hashcode, which will prevent it from functioning properly in collection such as HashMap and HashSet.





