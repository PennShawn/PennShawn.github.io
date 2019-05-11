---
title: 《Effective Java》Item5. Perfer dependency injection to hardwiring resources
categories :
- 技术
tags :
- Java
- Effective Java
---

Many classes depend on one or more underlying resources. For example, a spell checker depends on a dictionary. It is not uncommon to see such classes implemented as static utility classes.

```
//inflexible & untestable 
public clsss SpellChecker {
    private static final Lexicon dictionart = ,,,;
    
    private SpellChecker(){}//nonistantiable
    public static INSTANCE = new SpelChecke(....);
    
    public boolean  isValid(String words){...}
    ...
}
```
It's not satisfactory,because is assume that there is only one dictionary worth using.In practive, each language has its own dictionary, and special dictionaries are use for special vocavularies. Also, it may be desirable to use a special dictionary for testing.  

A simple pattern that saitisfies this requirement is to pass the resources into the constructor when creating a new instance.
```
//Dependency injection provides flexibility and testablity
public class SpellChecker {
    private final Lexicon dictionary;
    
    public SpellChecker(Lexicon dictionary){
        this.dictionary = Objects.requireNonNull(dictionary);
    }
    
    public boolean isValid(String words){...}
    ...
}
```
&nbsp;A useful variant is to pass aresource factory to the constructor.A factory is an object that can be called repeadly to create instance of a type.Such factories embody the *Factory Method* pattern.The Supplier<T> interface is perfect for representing factories.
&nbsp; Although dependency injection greatly improve flexibility and testablity, it can clutter up large projects, which typically contain thounsands of dependencies. This iclutter can be all but eliminated by using a **dependency injection framwark**,such as Dagger, Guice, or Spring.





