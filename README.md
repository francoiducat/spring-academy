# Spring Boot
- opinionated view of spring fwk
- 3 files to get running
  - pom.xml (<parent>spring-boot-starter-parent</parent>)
  - application.properties
  - Application.class (`@SpringBootApplication`)
- JdbcTemplate bean automatically configured through auto-configuration

A Spring Boot starter:
- a pre-configured set of dependencies for specific tasks
  - such as web development
  - data access, etc.

## Dependency Management
in the pom

## Spring Boot Properties

### Property files

#### application.properties files
`application.properties` (or `.yml` or `.json) found by Spring Boot in this order

- /config from working directory
- the working directory
- config package in classpath
- classpath root

and creates a **PropertySource** based on these files.

#### application-{profile}.properties files

#### multiple profile specific in single file

![multiple profile specific](static/multipleProfileSpecific.png)

#### Precedence

1. Devtools settings
2. `@TestPropertySource` and `@SpringBootTest` props
3. Command line args
4. SPRING_APPLICATION_JSON (inline json props)
5. ServletConfig/Context params
6. JNDI attributes from java:comp/env
7. Java System props
8. OS env vars
9. Profile-specific app properties
10. App properties
11. `@PropertySource` files
12. SpringApplication.setDefaultProperties

### @ConfigurationProperties

It simplifies handling of large amount of properties
It maps properties into classes:

```java
@ConfigurationProperties(prefix="rewards.client")
public class ConnectionSettings {
    private String host;
    private int port;
}
```

This must be a Spring managed beans injected in configuration classes. 3 ways to do it:

```java
@SpringBootApplication
@EnableConfigurationProperties(ConnectionSettings.class)
public class RewardsApplication { }
```

```java
@SpringBootApplication
@ConfigurationPropertiesScan
public class RewardsApplication { }
```

```java
@Component
@ConfigurationProperties("rewards.client-connection")
public class ConnectionSettings {
    private String hostUrl;
}
```

**Relaxed binding**, valid matches:

- rewards.client-connection.host-url
- rewards.client_connection.host_url
- rewards.clientConnection.hostUrl
- REWARDS_CLIENTCONNECTION_HOSTURL

## Autoconfiguration

How? SpringBoot provides configuration classes with conditions.
It initialises components based on conditions like:
- do classpath contents include specific classes?
- are some props set?
- are some beans already configured (or not)?

SB auto-configuration is
- managed by a set of auto-configuration classes which uses @Conditional's
- enabled by `@EnableAutoConfiguration`

```java
@SpringBootConfiguration
@ComponentScan("example.config")
@EnableAutoConfiguration
```
equals
```java
@SpringBootApplication(scanBasePackages="example.config")
```

Note : Component Scan in current package onwards (=any subpackage.)

## Override default configuration

Overriding options :

1. Set some Spring Boot's props (`application.properties` files)
2. Explicitly define beans (so SB won't)

Example: A `DataSource.class` defined stops SB from autoconfiguring a default `DataSource`.

3. Explicitly disable some auto-config 
 
via annotation:
```java
@EnableAutoConfiguration(exclude="DataSourceAutoConfiguration.class")
```
or
```java
@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class }) // for multiple excludes
```
via props file:
```yml
spring:
  autoconfigure:
    exclude: org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration.class
```
4. Change dependencies or their versions

in `pom.xml` with properties or dependencies management sections
```xml
<properties>
  <spring-framework.versions>5.3.22</spring-framework.versions>
</properties>
```

```xml
<dependency>
  <groupID>org.springframework.boot</groupID>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
    <exclusion>
      <groupID>org.springframework.boot</groupID>
      <artifactId>spring-boot-starter-tomcat</artifactId> // Exclude tomcat
    </exclusion>
  </exclusions>
</dependency>
<dependency>
  <groupID>org.springframework.boot</groupID>
  <artifactId>spring-boot-starter-jetty</artifactId> //use Jetty
</dependency>
```

## Packaging and Runtime
### Spring Boot (Maven) Plugin

Makes "fat" executable jars/wars including dependencies and tomcat

- `spring-boot-maven-plugin` and `mvn package` (or gradle)
- 2 jars are created: fat JAR & JAR.original (light)
- run with `java -jar mySpringBootApp.jar`
## Integration Test
```java
@SpringBootTest(classes=Application.class)
class MyServiceTests {
}
```
@SpringBootTest searches for @SpringBootConfiguration
```java
@SpringBootConfiguration(classes=Application.class)
@EnableAutoConfiguration
@ComponentScan("example")
class MyServiceTests {

}
```

## Spring Boot JPA

Enabled with
 ```xml
<dependency>
  <groupID>org.springframework.boot</groupID>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```
If JPA is aon classpath autoconfigures:
- `Datasource`
- `EntityManagerFactoryBean`
- `JpaTransactionManager`

Note : use `@EntityScan` when Entities are in different package than SpringBootApplication class

 ```java
@SpringBootApplication
@EntityScan("rewards.internal")
public class Application {}
```

## Spring Data Repositories for JPA

### Spring Data

![Spring Data](static/springData.png)

- SpringBoot simplifies setup for data access (set up most JPA)
- Spring Data simplifies Repositories (just define an interface)
- ìt reduces boiler plate code.
- implements Instant repositories 
- Scans for interface extended `Repository<T,K>`
- CRUD methods (find, save, delete) auto-generated if using `CrudRepository<T,K>` (extends `Repository`)
- Paging, custom queries and sorting supported

Different Datasources:
- `@Entity` for SQL (map to a sql table). Every Entity needs an `@Id`
- @Document for MongoDB (map to a json document)
- @NodeEntity for Neo4j (map to a graph)
- @Region for Gemfire (map to a region)

```java
public class CustomerRepository extends Repository<Customer, Long> {
  @Query("SELECT c from Customer c WHERE c.emailAddress = ?1")
  Customer findByEmail(String email); // ?1 replace by method param
}
```

Accessing the Repository

```java
@Configuration
@EnableJpaRepositories(basePackages="com.acme.repository")
public class CustomerConfig {
  @Bean
  CustomerService cs(CustomerRepository repo) {
      return new CustomerService(repo);
  }; 
}
```

`@Transient` on a field of Entity = no persisted

## Spring MVC

implements MVC pattern

Enabled with
 ```xml
<dependency>
  <groupID>org.springframework.boot</groupID>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Ensures Spring Web and Spring MVC are on classpath

### Controllers

- `@Controller` on class 
- `@GetMapping("/accounts")` + `@ResponseBody`on method
- `@Controller` + `@ResponseBody` = `@RestController`
- `@RequestParam` on method's param
- `@PathVariable` on method's param
- `@RequestHeader` on method's param
- `@ResponseStatus(HttpStatus.NO_CONTENT)` on method returning void because default returning status code is 200.

Injected arguments by Spring : `Principal`, `HttpSession`, `Locale` etc.

### Message Converters

Multiple response formats available. `@ResponseBody` specifies the response must be converted.

Automatically accepts Request and converts Response accordingly: 
- application/xml
- application/json

- `ResponseEntity` for entire HTTP response (header, status code, body)

### JAR/WAR config

SB autoconfiguration for Web Applications:
- Sets up a `DispatcherServlet`
- Sets up default resource locations (images, css, js)
- Sets ups default message Converters
- Sets up internal configuration to support controllers

SB starts up en embedded web container (tomcat default) with one cmd `java -jar mySpringBootApp.jar`
 
- both JAR and WAR forms can run from the CLI based on embedded tomcat (or jetty) JARs
- Embedded tomcat JARs can cause conflicts whe running WAR inside traditional web container (ie when running with different version of tomcat)
- Best Practice: MArk Tomcat dependencies as provided when building WARs for traditional containers.

For `WAR` to be executed with CLI, must extend `SpringBootServletInitializer` :

```java
@SpringBootApplication
public class Application extends SpringBootServletInitializer {
  // Web Container Support, Specify the config class(es) to use
  protected SpringApplicationBuilder configure(SpringApplicationBuilder applicationBuilder) {
    return applicationBuilder.sources(Application.class);
  }
  // main() method from CLI
  public static void main(String args[]) {
    SpringApplication.run(Application.class, args);
  }
}
```
  
`spring-boot-dev-tools`

-avoid cold restart 
-automatic app restart


### MVC

Files to get running

- pom.xml : setup SB dependencies
- RewardController class: Basic Spring MVC controller
- application.properties: setup
- Application class: Application Launcher

Dependency : `spring-boot-starter-web`

## RESTful app with SpringBoot

REST principles:

- Expose resources through URIs (nouns, not verbs)
- Limited set of operations: GET, PUT, POST, DELETE for HTTP
- Resources support multiple representation (Json, XML ...)
- Representations link to other resources
- HATEOAS : Hypermedia As The Engine Of Application State: means that RESTful response contains the links we need.
- Stateless architecture
  - no HttpSessions
  - GETs can be cached on URL
  - clients keep track of the state
- Scalable
- Support for redirect, caching resource identification, different representations


- on method annotated `@PostMapping` return URI location of the resource with `ResponseEntity.created(build_URI_Here).build()`
  - option 1: UriComponentsBuilder
  - option 2: `ServletUriComponentsBuilder.fromCurrentRequestUri().path("/{itemId}").buildAndExpand("item A").toUri();`

### RestTemplateBuilder

```java
RestTemplate template = new RestTemplate();
String uri : "http://example.com/store/orders/{id}/items";
OrderItem[] items = template.getForObject(uri, OrderItem[].class, "1");
URI itemLocation = template.postForLocation(uri,item,"1");
template.put(itemLocation,item);
template.delete(itemLocation);

RequestEntity<OrdrItem> request = RequestEntity.post(new URI(itemUrl)).getHeaders().add(..basic...).body();
ResponseEntity<Void> response = template.exchange(request, Void.class);
```

`WebClient` is preferred for streaming (#reactive,#nonBlocking)


## Spring Boot Testing

 - Dependency `spring-boot-starter-test` with scope *test*

### `@SpringBootTest`:

   - for integration testing
   - automatically searches for `@SpringBootConfiguration`
   - embedded servers gets started (#takesTimeToStart!)
   - auto-configures a **TestRestTemplate**
   - provides support for different `webEnvironment` modes (RANDOM_PORT, DEFINED_PORT, MOCK, NONE)

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class MyClassTest {

  @Autowired
  private TestRestTemplate restTemplate;

  @Test
  void test() {
    restTemplate.getForObject("url", MyDto.class);
  }
}
```
 
### `@ContextConfiguration` for slice testing
 
### TestRestTemplate

- alternative to `RestTemplate`
- takes a relative path (vs absolute)
- fault tolerant
- configured to ignore cookies and redirects
- customize with `RestTemplateBuilder` (message converters, timeouts etc.)

### MockMVC Test 

- provides a mock web environment: no need for running an external app server
- `@SpringBootTest(webEnvironement = WebEnvironment.MOCK)` + `@AutoConfigureMockMvc` = `@WebMvcTest`
- `mockMvc.perform(get().accept().header().content().content-type())` etc.
- `perform(get()).andDo(print())` to debug tests

### Slice Testing

- to perform isolated testing (web, repository, caching)

### @WebMvcTest

- disables full auto-configuration
- apply auto-config mvc testing fwk only (`@Autowired MockMvc)
- dependencies need to be mocked with `@MockBean`

`@Mock`

- from Mockito
- used when spring context not needed

`@MockBean`
- from spring boot
- used when spring context not needed
  - creates a new mock bean when not present in the spring context OR 
  - replaces a bean with a mock bean when it is present

```java
@WebMvcTest
myControllerTest() {
  @Autowired
  private MockMvc mockMvc;
  @MockBean
  private myService myService;
  
  @Test
  void test(){
      mockMvc.perform(get()); //
  }
}

```
### @DataJpaTest

- loads @Repository beans, excludes all other @Components
- auto-configures `TestEntityManager`
  - alternative to `EntityManager`
  - provides a subset of `EntityManager` methods
    - just those useful for tests
    - helper methods for common testing tasks : `persistFlushFind()`, `persistAndFlush()`
    - Uses an embedded in-memory database
      - replaces any explicit or auto-configured DataSource
      - `@AutoConfigureTestDatabase` can be used to override these settings.


## Spring Security

#### Principal
user, device or system that performs an action
#### Authentication
- establishing that a principal's credentials are valid. 
- authentication mechanisms: Basic, Digest, Form, X.509, OAuth 2.0 / OIDC
- storage options for credential: in-memory (dev only), Database, LDAP
#### Authorization
- Deciding if a principal is allowed to access a resource
- Depends on authentication : user identity must be established
#### Authority
- Permission or credential enabling access (such as a role like admin/member/guest)
- A Role is simply a commonly used type of Authority
#### Secured Resource
Resource that is being secured

### Why Spring Security?
#### Portable
can be used on any Spring project
#### SOC
- Business logic is decoupled from security concerns
- Athenthication & Authorization are decoupled
#### Felixible & Extensible
- Authentication: Basic, Digest, Form, X.509, OAuth 2.0 / OIDC, Cookies etc.
- Storage : LDAP, RDBMS, Props file, custom DAOs etc
- Highly customizable

 
### Setup and Configuration
#### 1. Setup Filter chain
- SecurityContextPersistenceFilter
- LogoutFilter
- UsernamePasswordAuthenticationFilter
- ExceptionTranslationFilter
- AuthorizationFilter

#### 2. Configure security (authorization) rules

Url Authorization & By-passing Security:

```java
@Configuration
public class SpringSecurityConfig {
    
  @Bean
  public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests((authz) -> authz
            .requestMatchers("/signup").permitAll() // allows open-access but still processed 
            // by Spring Security Filter Chain
            .requestMatchers("/accounts/edit").hasRole("ADMIN")
            .requestMatchers("/accounts").hasAnyRole("ADMIN", "MEMBER")
            .anyRequest().authenticated());
    return http.build();
  }

  // By-passing Security completely
  @Bean
  public WebSecurityCustomizer webSecurityCustomizer() {
    return web -> web.ignoring().requestMatchers("/swagger-ui/");
  }
}
```

Note: antMatchers and mvcMatchers are deprecated in SpringSecurity 5.8

#### 3. Setup Web Authentication

- Observing the default security behavior:
password printed in the console. 
- Run "curl -i localhost:8080/accounts" and observe 401 response
- Run "curl -i -u user:<Spring-Boot-Generated-Password> localhost:8080/accounts"

// - Configuring authorization based on roles

// - Configuring authentication using in-memory storage

```java
@Configuration
public class SpringSecurityConfig {

    @Bean
    public InMemoryUserDetailsManager userDetailsService(PasswordEncoder passwordEncoder) {

        // TODO-05: Add three users with corresponding roles:
        // - "user"/"user" with "USER" role (example code is provided below)
        // - "admin"/"admin" with "USER" and "ADMIN" roles
        // - "superadmin"/"superadmin" with "USER", "ADMIN", and "SUPERADMIN" roles
        // (Make sure to store the password in encoded form.)
        // - pass all users in the InMemoryUserDetailsManager constructor
        UserDetails user = User.withUsername("user").password(passwordEncoder.encode("user")).roles("USER").build();
        UserDetails user = User.withUsername("admin").password(passwordEncoder.encode("admin")).roles("USER", "ADMIN").build();
        UserDetails user = User.withUsername("superadmin").password(passwordEncoder.superadmin("user")).roles("USER","ADMIN", "SUPERADMIN").build();

        return new InMemoryUserDetailsManager(user, admin, superadmin /* Add new users comma-separated here */);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }
}
```

// - Configuring method-level security
// - Adding custom UserDetailsService
// - Adding custom AuthenticationProvider
// - Writing test code for security


#### Method Security

### Security Testing

```java
@Configuration
public class SpringSecurityConfig {
    
  @Test
  @WithMockUser(roles = {"ADMIN", "SUPERADMIN"})
  void test() {
      
  }
        
}
```

## Spring Boot Actuator

`/actuator` is the base path (can be changed)

### info
general info: name, version...
### health
- a composite status (db, diskSpace, custom health status etc): `{"status" : "UP"}
- possible to group health indicators
- 

### metrics
- jvm, cup, connection pool etc. 
- collected with micrometer

#### Custom metrics:
- Counter, Gauge, Timer, DistributionSummary (ie amount of an order)
- created/registred with a `MeterRegistry` bean
- listed on /actuator/metrics
- fetched at /actuator/metrics/custom-metric-name

#### Hierarchical vs Dimensional Metrics

- Hierarchical
  - key/value pair. Ex: http.method.get.status.200
- Dimensional
  - metrics are tagged. Ex: http?tag=method:get&tag=status:200&tag=region:us-east
  - better naming convention
  - add new attribute to a query is easy


### setup
- `spring-boot-starter-actuator` dependency
### endpoints
- beans
- conditions
- env
- health
- configprops
- info
- loggers
- mappings
- metrics
- session, shutdown, threaddump, jolokia

#### Enabled endpoints
- its bean exists in the application context
- all enabled except shutdown

#### Exposed endpoints
Accessible via:
- JMX : all enabled endpoints exposed (only when spring.jmx.enabled=true)
- HTTP : health (only when using spring MVC, WebFlux or Jersey)

#### Custom expose
management.endpoints.web.exposure.include=beans,metrics
=> health not exposed in this config

#### Secure endpoints
![Secure endpoints with Spring Security](static/actuator.png)


# Spring Framework

- opensource framework: helps focus on business logic (pojo programing model)
- lightweight
- DI container (IOC) : spring instantiates and injects dependencies into objects (lifecycle management)
- no need for Java EE application server
- Key principles: DRY, SOC, Convention over config, Testability
- Spring provides templates: JdbcTemplate, JmsTemplate, RestTemplate, WebServiceTemplate ...

## Configuration

- Spring separates application config from application objects (beans)
- Spring manages Objects
  - creates and initialize them 
- Spring gives unique id/name to beans
- Spring Application Context represents Spring DI Container
  - can be created in any env (standalone, web app, JUnit test)
- multiple config with `@Import(MyClass.class)`
- Bean scopes
  - `singleton` (default): single instance
  - `prototype`: new instance created each time the bean is referenced
  - `session`: new instance created once per user session (web only)
  - `request`: new instance created once per request (web only)
  - web socket scope
  - refresh scope
  - thread scope
  - custom scope
- Accessing **external properties**:
  - `@PropertySource("file.properties")` on class
  - or `@Value` on parameter
- Spring **profiles**:
  - can represent 
    - an env (dev, test prod)
    - an impl (jdbc, jpa)
    - a deployment platform (one-premise, cloud)
  - beans are grouped into profiles
  - `@Profile("dev")` or `@Profile("!prod")` can be at
    - class level
    - method level
  - activated at run time with
    - command line : `-Dspring.profiles.active=dev`
    - programmatically: `System.setProperty("spring.profiles.active","dev")` before `SpringApplication.run(AppConfig.class);`
    - Integration Test only: `@ActiveProfiles`
    - with `@Value("${maxAttempts}")`
- Spring **Expression Languages** (SpEL):
   - with `@Value("#{maxAttempts}")` 
     - accessing system properties
     - accessing spring beans
   - can provide a fallback with `@Value("#{maxAttempts: 5}")`

### Component Scanning
- Annotation-based configuration within bean-class `@Configuration @Bean` vs component-scanning with `@Component`
- `@Autowired(required = false)` default is true
- `@Qualifier("beanName")` in parameter or property (after @Autowired )
- `@Component` name auto-generated de-capitalized
- `@Lazy @Component` bean created when dependency injected or when `ApplicationContext.getBean` (default true)
- `@PostConstruct` invoked during bean-creation process (javax not Spring)
- `@PreDestroy` called when ConfigurableApplicationContext is closed. (javax not Spring)
- or alternatively`@Bean(initMethod="populateCache",destroyMethod=""flushCache")`
 
## Spring Container

### LifeCycle

![Spring Beans Initialization Steps](static/BeanInitSteps.png)
*Spring Beans Initialization Steps*
### Initialization: beans created, DI occurs
#### A.Load & Process : #What should we create ? (Keeps Beans Definition In Memory)
- Load Beans (javaconfig, xml, annotations,component scan ) 
    - gather information about the application context (AC is a BeanFactory)
        - AC BeanFactory : gather bean name, type, scope 
- Post Process Beans Definition 
    - interface BeanFactoryPostProcessor
        - can modify any bean definition before any objects are created
        - some implementation provided by spring : reading properties (`@Value("${max.retries}")`), registering custom scope...
        - Use STATIC for custom BFPP: `@Bean public static BFPP mycustomConfigurer() {}`
#### B. Create and initialize each bean
- Bean created 
    - in right order based on dependencies
    - force dependency order  with `@DependsOn("beanName)`
- Bean initialized *eagerly* (unless marked *lazy*)
    - DI: Instantiate + Setter Injection
- Post Process at bean level BPP(before init & after init)
- Bean ready to use
### Usage: beans available
During previous BPP, bean can be:
1. Just a bean
2. a Proxy
![Bean Or Proxy](static/beanOrProxy.png)
*Bean Or Proxy*

Spring suppports 
- JDK proxy
    - interface based (#implements)
- *CGLib Proxy* 
    - subclass based (#extends)

### Destruction: beans clean up 

1. Any registered `@PreDestroy` methods are invoked
=> Don't happen if application is killed or fails

2. Beans released for Garbage Collection

## Spring AOP (Aspect Oriented Programming)

AOP enables modularization of cross-cutting concerns

**cross-cutting concerns**: generic functionnality needed in many place in the app
- logging and tracing
- transaction management
- security
- caching
- error handling
- perf monitoring
- custom business rules

AOP avoids:
- code tangling: coupling of concers (hard to test)
- code scattering: same concern spread across modules (code duplicates)

AOP technologies:
- Aspect J (JDK proxy)
- Spring AOP (`@Configuration @EnableAspectJAutoProxy`)
 AOP happens during initialization phase: spring wraps the component in a proxy  

### AOP Concepts
#### Join Point
method call or exception trown
#### Pointcut
expression that select one or more join points

![Expression](static/expression.png)

- `*` matches only once
- `..` matches zero or more

Any class annnotated *@Cacheable*:

```java
@Before(value = "execution(@example.Cacheable * rewards.. *.*(..))")
```

#### Advice
code to be executed at each join point with:
- `@Before(value= "pointcut execution")`
- `@AfterReturning(value = "", returning="reward")`
- `@AfterThrowing(value="", throwing= "ex")` can return a different type of exception
- `@After()`
- `@Around()` 
    - for before and/or after 
    - need to implement `proceed()` method

#### Aspect 
module (java class) that encapsulates pointcuts and advice annotated with `@Aspect`
#### Weaving
technique by which aspects are combined with main code
#### Proxy
someone else, web proxy etc. 
#### AOP Proxy
class that stand in place (transaction, caching etc.)

### AOP Limits
- only non-private method
- only aspects to spring beans
- weaving proxies inner calls: suppose method @() calls method b() on the same class/interface then advice will never be executed for method b()

## Spring Testing

- Spring TestContext framework provides explicit support for JUnit 4, JUnit Jupiter (AKA JUnit 5), and TestNG
- External dependencies should be minimized
- Isolate with stubs and mocks
- NO need Spring for unit test but YES for integration test
- With Junit5 and Spring'extension via `@ExtendWith(SpringExtension.class)`
- Define spring config to use via `@ContextConfiguration(classes={TestConfig.class})`

`@SpringJUnitConfig(TestConfig.class)` is a composed annotation for:
```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes={TestConfig.class})
```

- `@SpringJunitConfig` without config class specified in () is possible but need to add an inner static class with the config in this test class.
- `@TestPropertySource(properties = "", locations = "")`
- `@ActiveProfiles({"test","jdbc"})`: bean associated to that profil + not associated to any profile
- `@Sql({"schema.sql","data.sql"})`
    - at class runs before each `@Test`
    - method level runs before `@Test` 
    - `@Sql(scripts="cleanup.sql",executionPhase=Sql.ExecutionPhase.AFTER_TEST_METHOD)` runs after `@Test` method
    - `@Sql(scripts= "data.sql", config = @SqlConfig(errorMode = ErrorMode.FAIL_ON_°ERROR, commentPrefix = "//", separator = "@@") )` controls what to do if scripts fails
        - `FAIL_ON_ERROR`
        - `CONTINUE_ON_ERROR`
        - `IGNORE_FAILED_DROPS`
        - `DEFAULT` = whatever `@Sql` defines at class level  otherwise `FAIL_ON_ERROR`


### @DirtiesContext

- forces context to be closed (allows testing of `@PreDestroy`)
- forces Spring to start with a clean slate, as if those other tests hadn't been run.
- add this annotation to all tests which change the data. If not, then these tests could affect the result of other tests
- cached context destroyed, 


## Spring JDBC

### JDBC API limits
- boilerplate code, error prone code
- must check SQLException

### JdbcTemplate
- Rod Johson "Life is too short to write JDBC"
- JDBC template requires a DataSource : `JdbcTemplate template = new JdbcTemplate(datasource);`
- Do not create one for each thread (Inject it in constructor class and re-use it in methods)
- Thread safe after construction  
- JdbcTemplate can query for Simple types (int, long String Date ...), Generic Maps, Domain Objects
- jdbcTemplate.queryForObject
    - connection acquisition
    - transaction participation
    - statement execution
    - resultset process (results.add(rowMapper.mapRow(resultset,rowNumber)))
    - exception handling
    - connection release
```java
int count = jdbcTemplate.queryForObject("SELECT blabla", Integer.class);
```
- Query with bind variable using `?`
```java
String sql = "SELECT count(1) from PERSON WHERE age > ? and education = ?";
int count = jdbcTemplate.queryForObject(sql, Integer.class, age, education.toString());
```
- jdbcTemplate.insert DOES NOT EXIST. Any non-SELECT SQL is run using `jdbcTemplate.update()`:
```java
jdbcTemplate.update("INSERT INTO PERSON (name,age) VALUES (?,?)", person.getName(), person.getAge());
``` 

- JdbcTemplate can return each row of a rs as a Map
    - expecting single row: use `queryForMap`
    - expecting multiple rows: use `queryForList` returns a List of Map

```java
Map<String,Object> getPersonDetails(String id) {
    return jdbcTemplate.queryForMap("SELECT * FROM PERSON where ID = ? ", id);
}
// Returns Map of key (column name) value (column values)
// Map { ID = 1, FIRST_NAME="John", LAST_NAME="Doe"}
```

```java
List<Map<String,Object>> getAllPersons() {
    return jdbcTemplate.queryForList("SELECT * FROM PERSON");
}
// Returns List of Maps  
// List {
// 0 - Map { ID = 1, FIRST_NAME="John", LAST_NAME="Doe"}
// 1 - Map { ID = 2, FIRST_NAME="Bod", LAST_NAME="Smith"}
// 2 - Map { ID = 3, FIRST_NAME="Sarah", LAST_NAME="Connor"}
// }
//}
```

#### RowMapper functional interface 
```java
@FunctionalInterface
public interface RowMapper<T> {
  T mapRow(ResultSet rs, int rowNum) throws SQLException;
  }
```

- for mapping a single row of a rs to an object
- used for both single or multiple row queries
- return type paramatized
##### Query for single row
- Map rs with one domain object with jdbcTemplate.queryForObject (with lambda or callback)
```java
Customer customer = jdbcTemplate.queryForObject("SELECT * PERSON where ID = ?",
(rs, rowNum) -> new Customer(rs.getString(name)), id);
```
or
```java
	public Restaurant findByMerchantNumber(String merchantNumber) {
		String sql = "select MERCHANT_NUMBER, NAME, BENEFIT_PERCENTAGE, BENEFIT_AVAILABILITY_POLICY"
				+ " from T_RESTAURANT where MERCHANT_NUMBER = ?";
		return jdbcTemplate.queryForObject(sql, new
        RowMapper<Restaurant>() {
          public Restaurant mapRow(ResultSet rs, int rowNum) throws SQLException {
            return mapRestaurant(rs);
          }
        }, merchantNumber);
	}
```
##### Query for multiple rows
- Map rs with list of domain objects with jdbcTemplate.query (with callback with explicit RowMapper)
```java
List<Customer> customers = jdbcTemplate.query("SELECT blabla", 
new RowMapper<Customer>(){
    public Customer mapRow(ResultSet rs, int rowNum) throws SQLException ¶
    // do the mapping from rs
})
```

#### ResultSetExtracor functional interface
```java
@FunctionalInterface
public interface ResultSetExtractor<T> {
  T extractData(ResultSet rs) throws SQLException, DataAccessException;
}
```
- for mapping an entire rs to an object
    - n to one Mapping
    - #sqlWithJoin
- iterating on rs not automatic => `while(rs.next()) {rs.getString("name") blabla}`
- 

#### RowCallbackHandler functional interface
```java
@FunctionalInterface
public interface RowCallbackHandler<T> {
  void processRow(ResultSet rs) throws SQLException, DataAccessException;
}
```

### Exception handling

#### Checked Exceptions
- force developers to handle erros
- ex: SQLException 
#### Unchecked Exceptions
- Spring always throws Runtime (unchecked) Exceptions 
- `DataAccessException` hierarchy of sub-exceptions that hides whether using JPA, Hibernate, JDBC etc.
     - BadSqlGrammarException (column not found eg)
     - DataIntegrityViolationException
     - DataAccessResourceFailureException
     - CleanupFailureDataAccessException
     - OptimisticLocking Exception


## Spring Transaction Management

### Non-Transactional
When a unit-of-work containes 4 data access operations (eg SELECT, SELECT, UPDATE, INSERT), each acquires, uses and releases a distinct Connection. 

### Transactional
- Enable concurrent access to a shared resource.
- A set of tasks which take place as a single, indivisible action
- A unit of work that should be ATOMIC
#### Atomic 
Each unit of work is an All-or-nothing operation
#### Consistent
Database integrity Constraints are never violated
#### Isolated
Isolating transactions from each other
#### Durable
Committed changes are permanent (#final )

### Implementation
1. Declare a `PlatformTransactionManager` bean. Several implementation available:
  - DataSourceTransactionManager
  - JmsTransactionManager
  - JpaTransactionManager
  - JtaTransactionManager
  - WebLogicJtaTransactionManager
  - WebSphereUowTransactionManager
```java
@Bean
public PlatformTransactionManager transactionManager(DataSource dataSource) {
  return new DataSourceTransactionManager(datasource);
}
```
Declare `@Transactional` above relevant class or method(props overrided )
- wrapped in a proxy with *around* advice
- Rollback if method throws a Runtime Exception. Checked exceptions don't. (unless config specifies the contrary)
- commit at the end of the method
- transaction started before entering the method
- all controlled by config
2. Add `@EnableTransactionManagement` in config class

Transaction bound to current thread.
- holds underlying JDBC connection
- access manually with `DataSourceUtils.getConnection(datasource)`
- different from javax.transaction.Transactional (fewer options, supported by Spring)

### Transaction Propagation
Wether Connection to datasource gets opened/closed.
#### REQUIRED (default)
`@Transactional(propagation=Propagation.REQUIRED)`
Needs a connection. Takes the current if exists or create a new one.
#### REQUIRES_NEW
`@Transactional(propagation=Propagation.REQUIRES_NEW)`

Since it uses an around advice, no new connection acquired because the *update2()* method is into a method with around advice.
The propagation rule does not get applied because the call is internal (the inner call does not go through a proxy.)
*update1()* and *update2()* will participate in the same transaction
```java
class ClientService {
  @Transactional(propagation=Propagation.REQUIRED)
  void update1() {
    update2()
  }
    @Transactional(propagation=Propagation.REQUIRES_NEW)
  void update2() {
  }
}
```
### Rollback Rules
#### Default
Rollback if method throws a Runtime Exception. Checked exceptions don't. (unless config specifies the contrary)
#### Overridden
```java
  @Transactional(rollbackFor=MyCheckedException.class,
              noRollbackFor={JmxException.class, MailException.class})
  public RewardConfirmation rewardAccount() throws Exception {
    
  }
```
### Testing Transaction
Annotate test method with `@Transactional` 
- Transaction rolled back afterwards
Annotate class with `@Transactional` 
- all tests transactional
- add `@Commit` for test method we want to commit data. 

## Spring Web

- @RestController @RequestMapping("/cashcards")
- @PostMapping, @PutMapping("/{id}"), @GetMapping("/{id}") @DeleteMapping"/{id}"
- `@GetMapping("/store/orders")` is equivalent to `@RequestMapping(path="/stores/orders", method=RequestMethod.GET)`
- `@RequestMethod` enumerators: GET,POST,PUT,PATCH,DELETE and HEAD,OPTIONS,TRACE must use this annotation.


### ResponseEntity
- Spring Web (or http) provides the .created()
    - ResponseEntity.created(uriOfCashCard).build(); => returns the uri in the Header Location
- UriComponentsBuilder ucb: method argument to POST handler method automatically passed in (injected from our friend, Spring's IoC Container.)
- restTemplate
    - withBasicAuth(user, pwd) 
    - getForEntity(url, responseType) => returns ResponsEntity
    - postForEntity(url, request, responseType) => returns ResponsEntity
    - putForEntity() DOES NOT EXIST!
    - exchange(url, method, request, responseType) => returns ResponsEntity
    - delete => returns void

## Spring Data
### PagingAndSortingRepository
- comes from Spring Data Pagination API
    - Provides `PageRequest` and `Sort` classes for pagination
- implements CrudRepository
- Spring Data's CrudRepository provides methods that support creating, reading, updating, and deleting data from a data store. cashCardRepository.save returns the saved object with a unique id provided by the database.

```java
Page<CashCard> page2 = cashCardRepository.findAll(
    PageRequest.of(
        1,  // page index for the second page - indexing starts at 0
        10, // page size (the last page might have fewer items)
        Sort.by(new Sort.Order(Sort.Direction.DESC, "amount"))));
```

### Pageable
allows Spring to parse out the 'page' number and 'size' query string parameters.
getSortOr() : no default so has to be specified

```java
PageRequest.of(
                   pageable.getPageNumber(),
                   pageable.getPageSize(),
                   pageable.getSortOr(Sort.by(Sort.Direction.DESC, "amount"))));
```
### Page
`page.getContent()`

## Spring Security

### Authentication

- act of Principal proving its identity to the system
- `Principal` : user of an API, person or  program.
- Principal provides credentials:
    - eg a Basic Authentication (user/pwd) to get a `Session Token` (stored in a `cookie`) passed to next requests. (providing credentials at every request is inefficient)
- `Filter Chain`
    - Spring Security authentication implementation
    - Component called prior to the Controller
    - Checks the user’s authentication

### Authorization

- Spring Security provides Authorization via Role-Based Access Control (RBAC) #permissions
- RBAC configured at both a global level or a per-method basis

### Same Origin Policy

- Same Origin Policy (SOP)
- relax the SOP with Cross-Origin Resource Sharing (CORS)
    - `@CrossOrigin` annotation without any arguments allows all origins /!\ 

### Common Web Exploits

#### Cross-Site Request Forgery
- pronounced “Sea-Surf”, and also known as `Session Riding`
-  CSRF Token for protection
- only actions that a user is authorized to do can be executed.
- Spring Security has built-in support for CSRF tokens which is enabled by default

#### Cross-Site Scripting (XSS)
- even more malicious since any script could be run

### Configuration

Adding org.springframework.boot:spring-boot-starter-security without any config locks down the app. (need to specify how authentication and authorization are performed)

`@Configuration` tells Spring to use this class to configure Spring and Spring Boot itself.
Any Beans specified in this class will now be available to Spring's <b>Auto Configuration engine</b>.
Spring Security expects a Bean to configure its Filter Chain

```java
@Configuration
class SecurityConfig {

    @Bean
    SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
             .authorizeHttpRequests(request -> request
                     .requestMatchers("/cashcards/**")
                     .authenticated())
             .httpBasic(Customizer.withDefaults()) //user+pwd
             .csrf(csrf -> csrf.disable());
        return http.build();
    }
}
```

   CashCard findByIdAndOwner(Long id, String owner);
   Page<CashCard> findByOwner(String owner, PageRequest pageRequest);
