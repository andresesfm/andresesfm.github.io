---
layout: post
title: Using Objectify alongside JDO
categories:
- Google App Engine
- Objectify
- JDO
---
When migrating from JDO to Objectify, It's often the case where you need them to co-exist during the transition. Here is my case:

Suppose that you have an entity called `UserProfile` defined as follows:
```java
@PersistenceCapable
public class UserProfile implements Serializable {
    
    @PrimaryKey
    @Persistent(valueStrategy = IdGeneratorStrategy.IDENTITY)
    private Key key;
    @Persistent
    public String photoPath;
    @Persistent
    public String userNickname;
}
    
```
A pretty normal JDO-annotated entity. Now reading on-line people say that Ofy and JDO can co-exist. So I tried something like:
```java
@Entity
@PersistenceCapable
public class UserProfile implements Serializable {
    @Ignore
    @PrimaryKey
    @Persistent(valueStrategy = IdGeneratorStrategy.IDENTITY)
    private Key key;
    
    @Id
    public Long oid;
    
    @Persistent
    public String photoPath;
    @Indexed
    @Persistent
    public String userNickname;
}
    
```

Notice that I'm explicitly ignoring the key. However I get the following vey cryptic exception:
<pre><code>
java.lang.NoClassDefFoundError: Could not initialize class com.x.model.OfyService
</code></pre>

The solution I found was to leave UserProfile intact and create a new class:
```java
@Entity(name="UserProfile")
public class UserProfileOfy implements Serializable {
    @Id
    public Long oid;
    
    public String photoPath;
    @Indexed
    public String userNickname;
}
    
```
Notice the `name` parameter in the `@Entity` annotation. 
