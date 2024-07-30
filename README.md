# spring-academy


# @DirtiesContext

- forces Spring to start with a clean slate, as if those other tests hadn't been run.
- add this annotation to all tests which change the data. If not, then these tests could affect the result of other tests

# Spring Web

- @RestController @RequestMapping("/cashcards")
    - @PostMapping, @PutMapping("/{id}"), @GetMapping("/{id}") @DeleteMapping"/{id}"


## ResponseEntity
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

# Spring Data

## PagingAndSortingRepository

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

## Pageable
allows Spring to parse out the 'page' number and 'size' query string parameters.
getSortOr() : no default so has to be specified

```java
PageRequest.of(
                   pageable.getPageNumber(),
                   pageable.getPageSize(),
                   pageable.getSortOr(Sort.by(Sort.Direction.DESC, "amount"))));
```
## Page
`page.getContent()`

# Security

## Authentication

- act of Principal proving its identity to the system
- `Principal` : user of an API, person or  program.
- Principal provides credentials:
    - eg a Basic Authentication (user/pwd) to get a `Session Token` (stored in a `cookie`) passed to next requests. (providing credentials at every request is inefficient)
- `Filter Chain`
    - Spring Security authentication implementation
    - Component called prior to the Controller
    - Checks the user’s authentication

## Authorization

- Spring Security provides Authorization via Role-Based Access Control (RBAC) #permissions
- RBAC configured at both a global level or a per-method basis

## Same Origin Policy

- Same Origin Policy (SOP)
- relax the SOP with Cross-Origin Resource Sharing (CORS)
    - `@CrossOrigin` annotation without any arguments allows all origins /!\ 

## Common Web Exploits

### Cross-Site Request Forgery
- pronounced “Sea-Surf”, and also known as `Session Riding`
-  CSRF Token for protection
- only actions that a user is authorized to do can be executed.
- Spring Security has built-in support for CSRF tokens which is enabled by default

### Cross-Site Scripting (XSS)
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
```

   CashCard findByIdAndOwner(Long id, String owner);
   Page<CashCard> findByOwner(String owner, PageRequest pageRequest);