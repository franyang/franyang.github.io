---
layout:     post
title:      springWbeFlux翻译
subtitle:   springWbeFlux翻译
date:       2019-02-19
author:     程序猿小哈
header-img: img/post-bg-coffee.jpeg
catalog: 	true
tags:
    - 翻译
    - spring
    - 框架
---

# <a href="https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux">spring WebFlux</a>学习   官方翻译 
 
## Web on Reactive Stack  web上的响应式技术栈
This part of the documentation covers support for reactive-stack web applications built on a <a href="http://www.reactive-streams.org/">Reactive Streams</a> API to run on non-blocking servers, such as Netty, Undertow, and Servlet 3.1+ containers. Individual chapters cover the Spring WebFlux framework, the reactive WebClient, support for testing, and reactive libraries. For Servlet-stack web applications, see <a href="https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#spring-web">Web on Servlet Stack</a>.<br>
这部分文档涵盖了对基于Reactive Streams API构建的reactive-stack Web应用程序的支持，以便在非阻塞服务器上运行，例如Netty，Undertow和Servlet 3.1+容器。各个章节涵盖Spring WebFlux框架，反应式WebClient，测试支持和反应库对于Servlet-stack Web应用程序的支持，请参阅Servlet堆栈上的Web。

## 1. Spring WebFlux
The original web framework included in the Spring Framework, Spring Web MVC, was purpose-built for the Servlet API and Servlet containers. The reactive-stack web framework, Spring WebFlux, was added later in version 5.0. It is fully non-blocking, supports Reactive Streams back pressure, and runs on such servers as Netty, Undertow, and Servlet 3.1+ containers.

Both web frameworks mirror the names of their source modules (spring-webmvc and spring-webflux) and co-exist side by side in the Spring Framework. Each module is optional. Applications can use one or the other module or, in some cases, both — for example, Spring MVC controllers with the reactive WebClient.<br>
Spring Framework中包含的原始Web框架Spring Web MVC是专为Servlet API和Servlet容器构建的。reactive-stack Web框架Spring WebFlux在5.0版后添加。它完全无阻塞，支持<a href="http://www.reactive-streams.org/">Reactive Streams</a>背压，并在Netty，Undertow和Servlet 3.1+容器等服务器上运行。 这两个Web框架都反映了其源模块的名称（spring-webmvc和spring-webflux），并在Spring Framework中并存。每个模块都是可选的。应用程序可以使用一个或另一个模块，或者在某些情况下，两者都使用 - 例如，带有反应式WebClient的Spring MVC控制器。

### 1.1 Overview 概况
Why was Spring WebFlux created?<br>
Part of the answer is the need for a non-blocking web stack to handle concurrency with a small number of threads and scale with fewer hardware resources. Servlet 3.1 did provide an API for non-blocking I/O. However, using it leads away from the rest of the Servlet API, where contracts are synchronous (Filter, Servlet) or blocking (getParameter, getPart). This was the motivation for a new common API to serve as a foundation across any non-blocking runtime. That is important because of servers (such as Netty) that are well-established in the async, non-blocking space.<br>
部分答案是需要非阻塞Web堆栈来处理少量线程的并发性，并使用较少的硬件资源进行扩展。 Servlet 3.1确实为非阻塞I / O提供了API。但是，使用它会远离Servlet API的其余部分，其中契约是同步的（Filter，Servlet）或阻塞（getParameter，getPart）。这是新的通用API作为跨任何非阻塞运行时的基础的动机。这很重要，因为在异步，非阻塞空间中已经建立了良好的服务器（例如Netty）。<br>
The other part of the answer is functional programming. Much as the addition of annotations in Java 5 created opportunities (such as annotated REST controllers or unit tests), the addition of lambda expressions in Java 8 created opportunities for functional APIs in Java. This is a boon for non-blocking applications and continuation-style APIs (as popularized by CompletableFuture and ReactiveX) that allow declarative composition of asynchronous logic. At the programming-model level, Java 8 enabled Spring WebFlux to offer functional web endpoints alongside annotated controllers.<br>
句子
答案的另一部分是函数式编程。就像在Java 5中添加注释一样创造了机会（例如带注释的REST控制器或单元测试），Java 8中添加lambda表达式为Java中的功能API创造了机会。这对于非阻塞应用程序和continuation-style API（由CompletableFuture和ReactiveX推广）来说是一个福音，它允许异步逻辑的声明性组合。在编程模型级别，Java 8使Spring WebFlux能够提供功能性Web端点以及带注释的控制器。<br>
#### 1.1.1. Define “Reactive”
We touched on “non-blocking” and “functional” but what does reactive mean?<br>
我们触及“非阻塞”和“功能性”但反应意味着什么？
The term, “reactive,” refers to programming models that are built around reacting to change — network components reacting to I/O events, UI controllers reacting to mouse events, and others. In that sense, non-blocking is reactive, because, instead of being blocked, we are now in the mode of reacting to notifications as operations complete or data becomes available.<br>
术语“反应性”是指围绕变化做出反应的编程模型 - 对I / O事件做出反应的网络组件，对鼠标事件做出反应的UI控制器等。从这个意义上说，非阻塞是被动的，因为我们现在处于一种模式，即在操作完成或数据可用时对通知作出反应。<br>
There is also another important mechanism that we on the Spring team associate with “reactive” and that is non-blocking back pressure. In synchronous, imperative code, blocking calls serve as a natural form of back pressure that forces the caller to wait. In non-blocking code, it becomes important to control the rate of events so that a fast producer does not overwhelm its destination.<br>
还有另一个重要的机制，我们在Spring团队中与“反应”相关联，即非阻塞背压。在同步，命令式代码中，阻塞调用是一种自然形式的背压，迫使调用者等待。在非阻塞代码中，控制事件的速率变得很重要，这样快速的生产者就不会压倒其目的地。<br>
Reactive Streams is a small spec (also adopted in Java 9) that defines the interaction between asynchronous components with back pressure. For example a data repository (acting as Publisher) can produce data that an HTTP server (acting as Subscriber) can then write to the response. The main purpose of Reactive Streams is to let the subscriber to control how quickly or how slowly the publisher produces data.<br>
Reactive Streams是一个小规范（在Java 9中也采用），用于定义具有背压的异步组件之间的交互。例如，数据存储库（充当发布者）可以生成HTTP服务器（充当订阅服务器）然后可以写入响应的数据。 Reactive Streams的主要目的是让订阅者控制发布者生成数据的速度或速度。<br>

>Common question: what if a publisher cannot slow down?
The purpose of Reactive Streams is only to establish the mechanism and a boundary. If a publisher cannot slow down, it has to decide whether to buffer, drop, or fail.<br>
常见问题：如果出版商不能放慢速度怎么办？ Reactive Streams的目的只是建立机制和边界。如果发布者不能减速，则必须决定是缓冲，丢弃还是失败。

### 1.1.2. Reactive API
Reactive Streams plays an important role for interoperability. It is of interest to libraries and infrastructure components but less useful as an application API, because it is too low-level. Applications need a higher-level and richer, functional API to compose async logic — similar to the Java 8 Stream API but not only for collections. This is the role that reactive libraries play.<br>
Reactive Streams在互操作性方面发挥着重要作用。它对库和基础架构组件很有用，但作为应用程序API不太有用，因为它太低级了。应用程序需要更高级别和更丰富的功能API来组成异步逻辑 - 类似于Java 8 Stream API，但不仅适用于集合。这是反应性图书馆所扮演的角色。<br>
Reactor is the reactive library of choice for Spring WebFlux. It provides the Mono and Flux API types to work on data sequences of 0..1 (Mono) and 0..N (Flux) through a rich set of operators aligned with the ReactiveX vocabulary of operators. Reactor is a Reactive Streams library and, therefore, all of its operators support non-blocking back pressure. Reactor has a strong focus on server-side Java. It is developed in close collaboration with Spring.<br>
Reactor是Spring WebFlux的首选反应库。它提供Mono和Flux API类型，通过与运算符的ReactiveX词汇表对齐的丰富运算符，处理0..1（Mono）和0..N（Flux）的数据序列。 Reactor是一个Reactive Streams库，因此，它的所有运营商都支持非阻塞背压。 Reactor非常关注服务器端Java。它是与Spring密切合作开发的。<br>
WebFlux requires Reactor as a core dependency but it is interoperable with other reactive libraries via Reactive Streams. As a general rule, a WebFlux API accepts a plain Publisher as input, adapts it to a Reactor type internally, uses that, and returns either a Flux or a Mono as output. So, you can pass any Publisher as input and you can apply operations on the output, but you need to adapt the output for use with another reactive library. Whenever feasible (for example, annotated controllers), WebFlux adapts transparently to the use of RxJava or another reactive library. See Reactive Libraries for more details.<br>
WebFlux要求Reactor作为核心依赖，但它可以通过Reactive Streams与其他反应库互操作。作为一般规则，WebFlux API接受普通Publisher作为输入，在内部使其适应Reactor类型，使用它，并返回Flux或Mono作为输出。因此，您可以将任何Publisher作为输入传递，并且可以对输出应用操作，但是您需要调整输出以与另一个反应库一起使用。只要可行（例如，带注释的控制器），WebFlux就会透明地适应RxJava或其他反应库的使用。有关详细信息，请参阅活动库。<br>

### 1.1.3. Programming Models
The spring-web module contains the reactive foundation that underlies Spring WebFlux, including HTTP abstractions, Reactive Streams adapters for supported servers, codecs, and a core WebHandler API comparable to the Servlet API but with non-blocking contracts.<br>
spring-web模块包含作为Spring WebFlux基础的反应基础，包括HTTP抽象，支持服务器的反应流适配器，编解码器，以及与Servlet API相当但具有非阻塞合同的核心WebHandler API。<br>
On that foundation, Spring WebFlux provides a choice of two programming models:
 + Annotated Controllers: Consistent with Spring MVC and based on the same annotations from the spring-web module. Both Spring MVC and WebFlux controllers support reactive (Reactor and RxJava) return types, and, as a result, it is not easy to tell them apart. One notable difference is that WebFlux also supports reactive @RequestBody arguments.
 + Functional Endpoints: Lambda-based, lightweight, and functional programming model. You can think of this as a small library or a set of utilities that an application can use to route and handle requests. The big difference with annotated controllers is that the application is in charge of request handling from start to finish versus declaring intent through annotations and being called back.

在此基础上，Spring WebFlux提供了两种编程模型供您选择： 
 + 带注释的控制器：与Spring MVC一致，并基于spring-web模块的相同注释。 Spring MVC和WebFlux控制器都支持反应式（Reactor和RxJava）返回类型，因此，要区分它们并不容易。一个值得注意的区别是WebFlux还支持反应式@RequestBody参数。 
 + 功能端点：基于Lambda，轻量级和函数式编程模型。您可以将其视为一个小型库或一组应用程序可用于路由和处理请求的实用程序。与带注释的控制器的最大区别在于，应用程序负责从开始到结束的请求处理，而不是通过注释声明意图并被回调。
 
### 1.1.4. Applicability 适用性
A natural question to ask but one that sets up an unsound dichotomy. Actually, both work together to expand the range of available options. The two are designed for continuity and consistency with each other, they are available side by side, and feedback from each side benefits both sides. The following diagram shows how the two relate, what they have in common, and what each supports uniquely:<br>
一个自然而然的问题，但是一个人提出了一个不合理的二分法。实际上，两者共同努力扩大可用选项的范围。两者的设计是为了保持连续性和一致性，它们可以并排使用，每一方的反馈都有利于双方。下图显示了两者之间的关系，它们的共同点以及各自的唯一支持：


We suggest that you consider the following specific points:<br>
我们建议您考虑以下具体要点:

+ If you have a Spring MVC application that works fine, there is no need to change. Imperative programming is the easiest way to write, understand, and debug code. You have maximum choice of libraries, since, historically, most are blocking.<br>
如果您的Spring MVC应用程序运行正常，则无需更改。命令式编程是编写，理解和调试代码的最简单方法。您有最多的库选择，因为从历史上看，大多数库都是阻塞的。

+ If you are already shopping for a non-blocking web stack, Spring WebFlux offers the same execution model benefits as others in this space and also provides a choice of servers (Netty, Tomcat, Jetty, Undertow, and Servlet 3.1+ containers), a choice of programming models (annotated controllers and functional web endpoints), and a choice of reactive libraries (Reactor, RxJava, or other).<br>
如果您已购买非阻塞Web堆栈，Spring WebFlux提供与此空间中的其他人相同的执行模型优势，并且还提供服务器选择（Netty，Tomcat，Jetty，Undertow和Servlet 3.1+容器），选择编程模型（带注释的控制器和功能Web端点），以及选择反应库（Reactor，RxJava或其他）。

+ If you are interested in a lightweight, functional web framework for use with Java 8 lambdas or Kotlin, you can use the Spring WebFlux functional web endpoints. That can also be a good choice for smaller applications or microservices with less complex requirements that can benefit from greater transparency and control.<br>
如果您对用于Java 8 lambdas或Kotlin的轻量级，功能性Web框架感兴趣，可以使用Spring WebFlux功能Web端点。对于较小的应用程序或具有较低复杂要求的微服务而言，这也是一个不错的选择，可以从更高的透明度和控制中受益。

+ In a microservice architecture, you can have a mix of applications with either Spring MVC or Spring WebFlux controllers or with Spring WebFlux functional endpoints. Having support for the same annotation-based programming model in both frameworks makes it easier to re-use knowledge while also selecting the right tool for the right job.<br>
在微服务架构中，您可以将应用程序与Spring MVC或Spring WebFlux控制器或Spring WebFlux功能端点混合使用。在两个框架中支持相同的基于注释的编程模型，可以更轻松地重用知识，同时为正确的工作选择合适的工具。

+ A simple way to evaluate an application is to check its dependencies. If you have blocking persistence APIs (JPA, JDBC) or networking APIs to use, Spring MVC is the best choice for common architectures at least. It is technically feasible with both Reactor and RxJava to perform blocking calls on a separate thread but you would not be making the most of a non-blocking web stack.<br>
评估应用程序的一种简单方法是检查其依赖性。如果您要使用阻塞持久性API（JPA，JDBC）或网络API，则Spring MVC至少是常见体系结构的最佳选择。从技术上讲，Reactor和RxJava都可以在单独的线程上执行阻塞调用，但是你不会充分利用非阻塞的Web堆栈。

+ If you have a Spring MVC application with calls to remote services, try the reactive WebClient. You can return reactive types (Reactor, RxJava, or other) directly from Spring MVC controller methods. The greater the latency per call or the interdependency among calls, the more dramatic the benefits. Spring MVC controllers can call other reactive components too.<br>
如果您有一个调用远程服务的Spring MVC应用程序，请尝试使用反应式WebClient。您可以直接从Spring MVC控制器方法返回反应类型（Reactor，RxJava或其他）。每次呼叫的延迟或呼叫之间的相互依赖性越大，其益处就越大。 Spring MVC控制器也可以调用其他无功组件。

+ If you have a large team, keep in mind the steep learning curve in the shift to non-blocking, functional, and declarative programming. A practical way to start without a full switch is to use the reactive WebClient. Beyond that, start small and measure the benefits. We expect that, for a wide range of applications, the shift is unnecessary. If you are unsure what benefits to look for, start by learning about how non-blocking I/O works (for example, concurrency on single-threaded Node.js) and its effects.<br>
如果你有一个庞大的团队，请记住转向非阻塞，功能和声明性编程时的陡峭学习曲线。在没有完整切换的情况下启动的实用方法是使用反应式WebClient。除此之外，从小处着手衡量效益。我们希望，对于广泛的应用，这种转变是不必要的。如果您不确定要查找哪些好处，请首先了解非阻塞I / O的工作原理（例如，单线程Node.js上的并发）及其影响。

### 1.1.5. Servers
Spring WebFlux is supported on Tomcat, Jetty, Servlet 3.1+ containers, as well as on non-Servlet runtimes such as Netty and Undertow. All servers are adapted to a low-level, common API so that higher-level programming models can be supported across servers.<br>
Spring WebFlux在Tomcat，Jetty，Servlet 3.1+容器以及非Servlet运行时（如Netty和Undertow）上受支持。所有服务器都适用于低级别的通用API，因此可以跨服务器支持更高级别的编程模型。

Spring WebFlux does not have built-in support to start or stop a server. However, it is easy to assemble an application from Spring configuration and WebFlux infrastructure and run it with a few lines of code.<br>
Spring WebFlux没有内置支持来启动或停止服务器。但是，从Spring配置和WebFlux基础架构组装应用程序并使用几行代码运行它很容易。

Spring Boot has a WebFlux starter that automates these steps. By default, the starter uses Netty, but it is easy to switch to Tomcat, Jetty, or Undertow by changing your Maven or Gradle dependencies. Spring Boot defaults to Netty, because it is more widely used in the asynchronous, non-blocking space and lets a client and a server share resources.<br>
Spring Boot有一个WebFlux启动器，可以自动执行这些步骤。默认情况下，启动程序使用Netty，但通过更改Maven或Gradle依赖项可以轻松切换到Tomcat，Jetty或Undertow。 Spring Boot默认为Netty，因为它在异步，非阻塞空间中使用得更广泛，并允许客户端和服务器共享资源。

Tomcat and Jetty can be used with both Spring MVC and WebFlux. Keep in mind, however, that the way they are used is very different. Spring MVC relies on Servlet blocking I/O and lets applications use the Servlet API directly if they need to. Spring WebFlux relies on Servlet 3.1 non-blocking I/O and uses the Servlet API behind a low-level adapter and not exposed for direct use.<br>
Tomcat和Jetty可以与Spring MVC和WebFlux一起使用。但请记住，它们的使用方式非常不同。 Spring MVC依赖于Servlet阻塞I / O，并允许应用程序在需要时直接使用Servlet API。 Spring WebFlux依赖于Servlet 3.1非阻塞I / O，并在低级适配器后面使用Servlet API，而不是直接使用。

For Undertow, Spring WebFlux uses Undertow APIs directly without the Servlet API.<br>
对于Undertow，Spring WebFlux直接使用Undertow API而不使用Servlet API。

### 1.1.6. Performance 性能
Performance has many characteristics and meanings. Reactive and non-blocking generally do not make applications run faster. They can, in some cases, (for example, if using the WebClient to execute remote calls in parallel). On the whole, it requires more work to do things the non-blocking way and that can increase slightly the required processing time.<br>
性能有许多特征和含义。反应性和非阻塞性通常不会使应用程序运行得更快。在某些情况下，它们可以（例如，如果使用WebClient并行执行远程调用）。总的来说，它需要更多的工作来以非阻塞的方式做事，并且可以略微增加所需的处理时间。

The key expected benefit of reactive and non-blocking is the ability to scale with a small, fixed number of threads and less memory. That makes applications more resilient under load, because they scale in a more predictable way. In order to observe those benefits, however, you need to have some latency (including a mix of slow and unpredictable network I/O). That is where the reactive stack begins to show its strengths, and the differences can be dramatic.<br>
反应和非阻塞的关键预期好处是能够使用少量固定数量的线程和更少的内存进行扩展。这使得应用程序在负载下更具弹性，因为它们以更可预测的方式扩展。但是，为了观察这些好处，您需要有一些延迟（包括缓慢和不可预测的网络I / O的混合）。这就是反应堆栈开始显示其优势的地方，差异可能是戏剧性的。

### 1.1.7. Concurrency Model 并发模型
Both Spring MVC and Spring WebFlux support annotated controllers, but there is a key difference in the concurrency model and the default assumptions for blocking and threads.<br>
Spring MVC和Spring WebFlux都支持带注释的控制器，但并发模型和阻塞和线程的默认假设存在关键差异。

In Spring MVC (and servlet applications in general), it is assumed that applications can block the current thread, (for example, for remote calls), and, for this reason, servlet containers use a large thread pool to absorb potential blocking during request handling.<br>
在Spring MVC（以及一般的servlet应用程序）中，假设应用程序可以阻塞当前线程（例如，用于远程调用），并且由于这个原因，servlet容器使用大型线程池来吸收请求期间的潜在阻塞处理。

In Spring WebFlux (and non-blocking servers in general), it is assumed that applications do not block, and, therefore, non-blocking servers use a small, fixed-size thread pool (event loop workers) to handle requests.<br>
在Spring WebFlux（以及一般的非阻塞服务器）中，假设应用程序不会阻塞，因此，非阻塞服务器使用小的固定大小的线程池（事件循环工作程序）来处理请求。

>“To scale” and “small number of threads” may sound contradictory but to never block the current thread (and rely on callbacks instead) means that you do not need extra threads, as there are no blocking calls to absorb.<br>"缩放“和”少量线程“可能听起来矛盾，但永远不会阻塞当前线程（并依赖于回调）意味着你不需要额外的线程，因为没有阻塞调用吸收。


Invoking a Blocking API<br>
What if you do need to use a blocking library? Both Reactor and RxJava provide the publishOn operator to continue processing on a different thread. That means there is an easy escape hatch. Keep in mind, however, that blocking APIs are not a good fit for this concurrency model.<br>
调用阻止API 如果您确实需要使用阻止库，该怎么办？ Reactor和RxJava都提供了publishOn运算符以继续在不同的线程上进行处理。这意味着有一个简单的逃生舱口。但请记住，阻塞API不适合这种并发模型。

Mutable State<br>
In Reactor and RxJava, you declare logic through operators, and, at runtime, a reactive pipeline is formed where data is processed sequentially, in distinct stages. A key benefit of this is that it frees applications from having to protect mutable state because application code within that pipeline is never invoked concurrently.<br>
可变状态<br> 在Reactor和RxJava中，您通过运算符声明逻辑，并且在运行时，形成一个反应流水线，其中数据在不同的阶段按顺序处理。这样做的一个主要好处是它可以使应用程序免于必须保护可变状态，因为该管道中的应用程序代码永远不会同时被调用。

Threading Model 线程模型<br> 
What threads should you expect to see on a server running with Spring WebFlux?<br>
您希望在运行Spring WebFlux的服务器上看到哪些线程？<br>

+ On a “vanilla” Spring WebFlux server (for example, no data access nor other optional dependencies), you can expect one thread for the server and several others for request processing (typically as many as the number of CPU cores). Servlet containers, however, may start with more threads (for example, 10 on Tomcat), in support of both servlet (blocking) I/O and servlet 3.1 (non-blocking) I/O usage.<br>
在“vanilla”Spring WebFlux服务器上（例如，没有数据访问或其他可选依赖项），您可以期望服务器有一个线程，而其他几个用于请求处理（通常与CPU核心数一样多）。但是，Servlet容器可以从更多线程开始（例如，在Tomcat上为10），以支持servlet（阻塞）I / O和servlet 3.1（非阻塞）I / O使用。<br>

+ The reactive WebClient operates in event loop style. So you can see a small, fixed number of processing threads related to that (for example, reactor-http-nio- with the Reactor Netty connector). However, if Reactor Netty is used for both client and server, the two share event loop resources by default.<br>
被动WebClient以事件循环方式运行。因此，您可以看到与此相关的少量固定数量的处理线程（例如，reactor-http-nio-与Reactor Netty连接器）。但是，如果Reactor Netty同时用于客户端和服务器，则默认情况下两者共享事件循环资源。<br>

+ Reactor and RxJava provide thread pool abstractions, called Schedulers, to use with the publishOn operator that is used to switch processing to a different thread pool. The schedulers have names that suggest a specific concurrency strategy — for example, “parallel” (for CPU-bound work with a limited number of threads) or “elastic” (for I/O-bound work with a large number of threads). If you see such threads, it means some code is using a specific thread pool Scheduler strategy.<br>
Reactor和RxJava提供线程池抽象，称为调度程序，与publishOn运算符一起使用，该运算符用于将处理切换到不同的线程池。调度程序具有建议特定并发策略的名称 - 例如，“并行”（对于具有有限数量线程的CPU绑定工作）或“弹性”（对于具有大量线程的I / O绑定工作）。如果您看到这样的线程，则意味着某些代码正在使用特定的线程池调度程序策略。

+ Data access libraries and other third party dependencies can also create and use threads of their own.<br>
数据访问库和其他第三方依赖项也可以创建和使用自己的线程。

Configuring
The Spring Framework does not provide support for starting and stopping <a href="https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-server-choice">servers</a>. To configure the threading model for a server, you need to use server-specific configuration APIs, or, if you use Spring Boot, check the Spring Boot configuration options for each server. You can <a href="https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-client-builder">configure</a> The WebClient directly. For all other libraries, see their respective documentation.<br>
Spring Framework不支持启动和停止服务器。要为服务器配置线程模型，需要使用特定于服务器的配置API，或者，如果使用Spring Boot，请检查每个服务器的Spring Boot配置选项。您可以直接配置WebClient。对于所有其他库，请参阅其各自的文档。

## 2.WebClient









