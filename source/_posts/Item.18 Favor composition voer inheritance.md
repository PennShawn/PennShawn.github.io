---
title: 《Effective Java》Item.18 Favor composition voer inheritance
categories :
- 技术
tags :
- Java
- Effective Java

---
Unlike method invocation,inheritance violates encapsulation. In other words, a subclass depend on the implementation details of its superclass for its proper funtion. The superclass's implementation may change from release to release, and if it does, the subclass may break, even though its code has not been touched.




