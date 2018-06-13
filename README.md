This application is a proof of concept for how to support functional
bean registration in Spring Boot. See
https://github.com/spring-projects/spring-boot/issues/8115 for
background.

The approach here is to provide a modified version of Spring Boot's
(internal) `BeanDefinitionLoader`. Example app:

```java
@SpringBootApplication
public class InitializerApplication
		implements ApplicationContextInitializer<GenericApplicationContext> {

	public static void main(String[] args) {
		SpringApplication.run(InitializerApplication.class, args);
	}

	@Override
	public void initialize(GenericApplicationContext context) {
		context.registerBean(RouterFunction.class, this::userEndpoints);
	}

	private RouterFunction<?> userEndpoints() {
		return route(GET("/"), request -> ok().body(Mono.just("Hello"), String.class));
	}

}
```

This works as a main application class and in integration tests
because the same `BeanDefinitionLoader` is used.

There is also a custom `AutoConfigurationImportSelectorAutoConfigurationImportSelector` 
which looks at autoconfig classes to see if they are `ApplicationContextInitializer`. If 
so the initializer is applied to the current context and the autoimport is cancelled.