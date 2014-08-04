---
layout: post
title: "Objectify's OnLoad can be tricky"
date: 2014-08-01 08:18:30 -0600
comments: true
categories: 
- Google App Engine
- Objectify
- JDO
---
When creating a `Ref<?>` to another object can be tricky since, here is the scenario:

I have an Entity `UserPost` that has a reference to `UserProfile` like so:
```java
@Entity
public class UserPost {
    @Id
    public Long id;

    @JsonIgnore
    @Index
    @Load(All.class)
    private Ref<UserProfile> user;

    @Ignore
    public String userNickname;

    @OnLoad
    public void onLoad(){
        if(user!=null){
            UserProfile userProfile = user.get();
            if (userProfile != null) {
                this.userNickname = userProfile.userNickname;
            }
        }
    }
	
	public void setUserRef(Ref<UserProfile> userRef) {
        this.user = userRef;
    }

    public static class All {
    }
```
The problem then was that `userNickname` was null sometimes even when `user` was being set. After some trial and error I found this post [Google Groups](https://groups.google.com/forum/#!topic/objectify-appengine/TT27QyLGYPI) where they mention the fact that @OnLoad only gets called when the Entity is not found in the session. 

My solution: call onLoad directly when setting the reference. That makes that instance to be usable right away. But at the same it will be loaded when needed.
```java
@Entity
public class UserPost {
    @Id
    public Long id;

    @JsonIgnore
    @Index
    @Load(All.class)
    private Ref<UserProfile> user;

    @Ignore
    public String userNickname;

    @OnLoad
    public void onLoad(){
        if(user!=null){
            UserProfile userProfile = user.get();
            if (userProfile != null) {
                this.userNickname = userProfile.userNickname;
            }
        }
    }
	
	public void setUserRef(Ref<UserProfile> userRef) {
        this.user = userRef;
		onLoad();
    }

    public static class All {
    }
```