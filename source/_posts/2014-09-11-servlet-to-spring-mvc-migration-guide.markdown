---
layout: post
title: "Servlet To Spring MVC Migration Guide"
date: 2014-09-11 18:44:01 -0600
comments: true
categories: 
- Java
- Servlet
- Spring MVC
---
##Introduction##
I recently worked on a migration from good-old servlets web-services project to a REST using Spring MVC. The main motivation for the migration was testability. Servlets made very hard to create a good test suite, not good enough for continious integration. On the other hand Spring MVC not only makes testing posible but enjoyable. I'll briefly layout my strategy and learnings from that project.

###Step 1: Create High-level Http tests ###
This step wil save you a lot of time in the long run. They will serve as a verification step. 
For these tests I used RestAssured. It lets you test web-services from the client's perspective. jsonPath makes it easy to verify responses.

###Step 2: Transform Servlets to Controllers###
####Step 2a: Create a one Cotroller per Servlet #####
Copy and paste the content of get and post###
In this step pass the HttpSevletRequest and HttpServletResponse as parameters

```java
public class HelloWorld extends HttpServlet {
    public void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException{
        ...
    }
    public void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException{
        ...
    }
}
```
Becomes
```java
@RequestMqpping("hello")
@Controller
public class HelloWorldController {
    
    @RequestStatus(RequestStatus.OK)
    public void doGet(HttpServletRequest request, HttpServletResponse response)
        throws Exception{
        ...
    }
    @RequestStatus(RequestStatus.OK)
    public void doPost(HttpServletRequest request, HttpServletResponse response)
            throws Exception{
        ...
    }
}
```

####Step 2b: Replace Dependencies to HttpServletRequest  ####
Pass the parameter types specific to your service.
This:
```java
@RequestMqpping("hello")
@Controller
public class HelloWorldController {
    
    @RequestStatus(RequestStatus.OK)
    public void doGet(HttpServletRequest request, HttpServletResponse response)
        throws Exception{
        	String name = request.getParameter("name");
        ...
    }
    
}
```

Becomes:

```java
@RequestMqpping("hello")
@Controller
public class HelloWorldController {
    
    @RequestStatus(RequestStatus.OK)
    public void doGet(@RequestParam String name, HttpServletResponse response)
        throws Exception{
        ...
    }
}
```

####Step 2c: Create appropriate return types and remove dependency to HttpServletResponse ####
Split Methods and return the appropriate Object Types
```java
@RequestMqpping("hello")
@Controller
public class HelloWorldController {
    
    @RequestStatus(RequestStatus.OK)
    public void doGet(@RequestParam String name,@RequestParam String action, HttpServletResponse response)
        throws Exception{
        ...
        PrintWriter writer = response.getWriter();
        if("sayhello".equals(action)){
            writer.write("{message:\"Hello " + name + "\"}");
        }else if("list".equals(action)){
            //write a list to writter or with Gson
        }
        ...
    }
}
```
Becomes:

```java
@RequestMqpping("hello")
@Controller
public class HelloWorldController {
    
    public static class HelloMessage{
    	public String message;
    }

    @RequestMapping("" params="action=sayhello")
    @RequestBody
    public HelloMessage sayHello(@RequestParam String name)
        throws Exception{
        	HelloMessage helloMessage = new HelloMessage()
            helloMessage.message ="Hello " + name );
            return helloMessage;
    }
    @RequestMapping("" params="action=list")
    @RequestStatus(RequestStatus.OK)
    public List<HelloMessage> list()
        throws Exception{
        List<HelloMessage> list = new ArrayList();
            //build list
        return list;
    }
}
```

###Step 3: Create Unit Test for Controllers ####
Use @RunWithSpring

###Step 4: Integrate Security###

###Step 5: Create Integration Tests###
Use MockMvc

