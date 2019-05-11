---
title: 《Effective Java》Item17. Minimize mutablity
categories :
- 技术
tags :
- Java
- Effective Java
---
####To Make a class immutable, follow this five rules:
 - 1. Don't provide methods that modify the object's state(known as mutators)
 - 2. Ensure that the class can't be extended.
 - 3. Make all fields final.
 - 4. Make all fields private.
 - 5. Ensure exclusive access to any mutable components.
