---
layout:       post
title:        "UT | Controller层单元测试"
subtitle:     "Spring Boot 2.2.0开始支持Junit 5，之前的版本需要手动引入依赖"
date:         2020-06-22
author:       "权芹乐"
header-img:   "img/post-bg-beach2.webp"
catalog:      true
tags:
    - Automated Testing
    - 自动化测试
    - Junit
    - Spring Boot
    - Unit Testing
    - 单元测试
---

单元测试一般分为四个步骤：setup、exercise、verify、teardown。用上面的代码举例，setup 阶段，我们创建了 mock 对象，并且设置了 queryForInt 的行为；exercise 阶段调用了测试函数；verify 阶段做了两件事情：1. 确认 queryForInt 函数被正确调用，2. count() 返回值符合预期；teardown 阶段一般用来清理和释放资源，我们这里不需要，直接跳过了。

[toc]

我的环境：
+ Java 8+
+ Maven 3
+ Spring Boot 2
+ Junit 5

Controller的单元测试很多时候都不容易做到隔离，不得不依赖下层类的逻辑、数据库、网络、第三方服务等等，所以，有时人们又将它归为接口测试，类似于HTTP API测试。

# 测试的几种类型


## 1. [发HTTP请求]启动服务，监听端口，发送真实的http请求
start the application up and listen for a connection like it would do in production, and then send an HTTP request and assert the response.
```Java
import org.springframework.boot.test.web.client.TestRestTemplate;

// @RunWith(SpringRunner.class) // needed if junit 4
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class HttpRequestTest {

  @LocalServerPort
  private int port; // bind the above RANDOM_PORT

  @Autowired
  private TestRestTemplate restTemplate;

  @Test
  public void greetingShouldReturnDefaultMessage() throws Exception {
    assertThat(this.restTemplate.getForObject("http://localhost:" + port + "/",
        String.class)).contains("Hello World");
  }
}
```

## 2. [发HTTP请求]不启动服务（但实例化整个上下文），却类似发送真实http请求
the full Spring application context is started but without the server.

Another useful approach is to not start the server at all, but test only the layer below that, where Spring handles the incoming HTTP request and hands it off to your controller. That way, almost the full stack is used, and your code will be called exactly the same way as if it was processing a real HTTP request, but without the cost of starting the server.

另一种有用的方法是根本不启动服务器，而是只测试其下的层，Spring在这一层处理传入的HTTP请求并将其传递给控制器。这样，几乎使用了整个堆栈，代码调用的方式与处理实际HTTP请求的方式完全相同，但是不需要启动服务器。

```Java
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest
@AutoConfigureMockMvc
class MockMvcExampleTests {
  
  @Autowired
  private MockMvc mvc;

  @Test
  void exampleTest() throws Exception {
    mvc.perform(get("/")).andExpect(status().isOk()).andExpect(content().string("Hello World"));
  }

}
```

^ Spring Boot is instantiating the whole context.（`@SpringBootTest`可以理解成用来集成测试？quanql）  

==注意上下两段代码，在类注释处的区别==  

V Spring Boot is only instantiating the web layer.（If you want to focus **_only_ on the web layer** and **_not start_ a complete ApplicationContext**, consider using `@WebMvcTest` instead.）

## 3. [发HTTP请求]不启动服务（仅实例化web层），却类似发送真实http请求

In this test, the full Spring application context is started, but without the server. We can narrow down the tests to just the web layer by using @WebMvcTest.  
在这个测试中，启动了完整的Spring应用程序上下文，但是没有服务器。我们可以使用@WebMvcTest将测试范围缩小到web层。
如果应用含有多个controller，可以使用@WebMvcTest(homecontroler.class)来只实例化一个控制器。
```Java
@WebMvcTest(GreetingController.class)
public class WebMockTest {

  @Autowired
  private MockMvc mockMvc;

  @MockBean
  private GreetingService service;

  @Test
  public void greetingShouldReturnMessageFromService() throws Exception {
    when(service.greet()).thenReturn("Hello Mock");

    mockMvc.perform(get("/greeting"))
        .accept(MediaType.APPLICATION_JSON_UTF8)
        .andDo(print()) // 添加 ResultHandler 结果处理器，比如调试时打印结果到控制台，更多处理器可查阅
        .andExpect(status().isOk()) // 添加 ResultMatcher 验证规则，验证请求结果是否正确
        .andExpect(content().string(containsString("Hello Mock")));
        // andReturn：返回执行请求的结果，该结果是一个恩 MvcResult 实例对象
  }
}
```

## Mock下层函数，调用

```Java
public class MockitoControllerTest {

    @InjectMocks
    private UserController userController;

    @Mock
    private UserRepository userRepository;

    @Before
    public void init() {
        MockitoAnnotations.initMocks(this); //【必须！】初始化@Mock
    }

    @Test
    public void testGetUserById() {
        User u = new User();
        u.setId(1l);
        when(userRepository.findOne(1l)).thenReturn(u);

        User user = userController.get(1L);
        verify(userRepository).findOne(1l);

        assertEquals(1l, user.getId().longValue());
    }
}
```


# 重要注解
## `@SpringBootTest`
The `@SpringBootTest` annotation tells Spring Boot to go and look for a main configuration class (one with `@SpringBootApplication` for instance), and use that to ==start a Spring application context==.

注释告诉SpringBoot去寻找一个主配置类(例如带有`@SpringBootApplication`的配置类)，并使用它来启动一个Spring应用程序上下文。

> Springboot默认只启动一次应用，并缓存它。对于使用配置相同的多个tc，或者一个tc中的多个method，就可以共用它。当然，可以使用`@DirtiesContext`来控制缓存。

它有两个属性：
+ webEnvironment：指定Web应用环境，它可以是以下值
    - MOCK：提供一个模拟的 Servlet 环境，内置的 Servlet 容器没有启动，配合可以与@AutoConfigureMockMvc 结合使用，用于基于 MockMvc 的应用程序测试。
    - RANDOM_PORT：加载一个 EmbeddedWebApplicationContext 并提供一个真正嵌入式的 Servlet 环境，随机端口。
    - DEFINED_PORT：加载一个 EmbeddedWebApplicationContext 并提供一个真正嵌入式的 Servlet 环境，默认端口 8080 或由配置文件指定。
    - NONE：使用 SpringApplication 加载 ApplicationContext，但不提供任何 servlet 环境。
+ classes：指定应用启动类，通常情况下无需设置，因为 SpringBoot 会自动搜索，直到找到 @SpringBootApplication 或 @SpringBootConfiguration 注解。

> PS: `@RunWith(SpringRunner.class)`废弃

## `@Transactional` 注解，它会在每个测试方法结束时会进行回滚操作。

但是，如果使用 `RANDOM_PORT` 或 `DEFINED_PORT` 这种真正的 Servlet 环境，HTTP 客户端和服务器将在不同的线程中运行，从而分离事务。 在这种情况下，在服务器上启动的任何事务都不会回滚。


# 参考

[1]: asdsa

[Offical > Testing the Web Layer](https://spring.io/guides/gs/testing-web/)
