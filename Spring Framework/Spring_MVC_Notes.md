# Spring MVC - Interview Questions & Answers

---

## 1. What is Model 1 Architecture?

**Model 1** is an older, **JSP-centric** web architecture where the **JSP page acts as both the controller and the view**. It directly handles HTTP requests, contains business logic, interacts with the database, and renders the response — all in one place.

| Aspect | Detail |
|---|---|
| **Request Handler** | JSP page itself |
| **Business Logic** | Embedded inside JSP (scriplets `<% %>`) |
| **Flow** | Browser → JSP → Database → Response |
| **Problem** | Tightly coupled; hard to maintain and test |

**Real World Analogy:** One person at a restaurant who takes your order, cooks the food, and serves it. Works for tiny setups, becomes a nightmare at scale.

---

## 2. What is Model 2 Architecture?

**Model 2** introduced the **MVC (Model-View-Controller)** pattern to web development, separating concerns into three distinct roles:

| Component | Role | Example |
|---|---|---|
| **Model** | Holds data / business logic | Java Bean, Service class |
| **View** | Renders the UI | JSP, Thymeleaf, HTML |
| **Controller** | Handles request, processes it, returns view | Servlet |

**Flow:**
```
Browser → Servlet (Controller) → Business Logic (Model) → JSP (View) → Browser
```

**Improvement over Model 1:** Business logic is separated from the view. Easier to maintain, test, and scale.

---

## 3. What is Model 2 Front Controller Architecture?

**Model 2 Front Controller** improves upon Model 2 by introducing a **single entry point (Front Controller)** — one central servlet that receives **all incoming requests** and dispatches them to the appropriate controller.

| Aspect | Model 2 | Model 2 Front Controller |
|---|---|---|
| Entry Point | Multiple Servlets | Single Front Controller Servlet |
| Request Routing | Each servlet handles its own URL | Front Controller delegates to specific controllers |
| Common Logic | Duplicated in each servlet | Centralized in Front Controller |
| Spring Implementation | ❌ | ✅ `DispatcherServlet` |

**Flow:**
```
Browser → DispatcherServlet (Front Controller) → Controller → Model → View → Browser
```

---

## 4. Example Controller Method in Spring MVC?

```java
@Controller                          // Marks this class as a Spring MVC Controller
@RequestMapping("/users")            // Base URL mapping for all methods in this controller
public class UserController {

    @Autowired
    private UserService userService;

    // Handle GET /users
    @GetMapping
    public String getAllUsers(Model model) {
        model.addAttribute("users", userService.findAll()); // Add data to model
        return "user-list";                                  // Return view name
    }

    // Handle GET /users/{id}
    @GetMapping("/{id}")
    public String getUserById(@PathVariable int id, Model model) {
        model.addAttribute("user", userService.findById(id));
        return "user-detail";
    }

    // Handle POST /users
    @PostMapping
    public String createUser(@ModelAttribute User user, BindingResult result) {
        if (result.hasErrors()) return "user-form";
        userService.save(user);
        return "redirect:/users"; // Redirect after successful save
    }
}
```

---

## 5. Explain Flow in Spring MVC?

The complete request-response flow in Spring MVC:

| Step | Component | What Happens |
|---|---|---|
| **1** | **Browser** | Sends HTTP request (e.g., `GET /users`) |
| **2** | **DispatcherServlet** | Receives all requests — the Front Controller |
| **3** | **HandlerMapping** | Determines which Controller method handles this URL |
| **4** | **Controller** | Executes business logic, builds the Model |
| **5** | **ModelAndView** | Controller returns view name + model data |
| **6** | **ViewResolver** | Translates view name (e.g., `"user-list"`) to actual view file (e.g., `/WEB-INF/views/user-list.jsp`) |
| **7** | **View (JSP/Thymeleaf)** | Renders HTML using model data |
| **8** | **Browser** | Receives final HTML response |

```
Browser
  ↓ HTTP Request
DispatcherServlet
  ↓ asks
HandlerMapping → returns Controller method
  ↓ calls
Controller → processes → returns ModelAndView
  ↓ sends to
ViewResolver → resolves view name to file path
  ↓ renders
View (JSP/Thymeleaf) → HTML
  ↓
Browser ← HTTP Response
```

---

## 6. What is a ViewResolver?

**ViewResolver** is a Spring MVC component that **translates the logical view name** returned by a controller into an **actual view file** (JSP, Thymeleaf, etc.). It eliminates hardcoding of file paths in controllers.

```java
// Controller returns just the logical name:
return "user-list";

// ViewResolver maps it to:
// /WEB-INF/views/user-list.jsp
```

**Configuration Example (Spring Boot):**
```properties
# application.properties
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
```

**Java Config:**
```java
@Bean
public InternalResourceViewResolver viewResolver() {
    InternalResourceViewResolver resolver = new InternalResourceViewResolver();
    resolver.setPrefix("/WEB-INF/views/");
    resolver.setSuffix(".jsp");
    return resolver;
}
```

**Real World Analogy:** Like a receptionist who knows where every person's office is. You say "Take me to John" — they translate it to "Room 204, 2nd floor."

---

## 7. What is Model?

**Model** is an interface in Spring MVC that acts as a **container to pass data from the controller to the view**. It is a key-value map where you add attributes that the view (JSP/Thymeleaf) can access to render dynamic content.

```java
@GetMapping("/welcome")
public String welcome(Model model) {
    model.addAttribute("username", "Rahul");   // Add data
    model.addAttribute("role", "Admin");
    return "welcome-page";                      // View accesses ${username} and ${role}
}
```

In the JSP/Thymeleaf view:
```html
<h1>Welcome, ${username}!</h1>   <!-- Renders: Welcome, Rahul! -->
<p>Role: ${role}</p>             <!-- Renders: Role: Admin -->
```

---

## 8. What is ModelAndView?

**ModelAndView** is a Spring MVC class that **holds both the Model (data) and the View (name) together** in a single object returned from a controller method. It's an older approach compared to returning a `String` view name with a separate `Model`.

```java
@GetMapping("/user/{id}")
public ModelAndView getUserDetails(@PathVariable int id) {
    ModelAndView mav = new ModelAndView();

    mav.setViewName("user-detail");           // Set view name
    mav.addObject("user", userService.findById(id)); // Set model data
    mav.addObject("title", "User Details");

    return mav;
}
```

| Aspect | `Model` + `String` | `ModelAndView` |
|---|---|---|
| View Name | Returned as `String` | Set via `setViewName()` |
| Model Data | Added to `Model` parameter | Added via `addObject()` |
| Modern Usage | ✅ Preferred in modern Spring | Used in legacy/older Spring apps |

---

## 9. What is RequestMapping?

`@RequestMapping` is a Spring MVC annotation used to **map HTTP requests to controller methods** based on URL, HTTP method, request parameters, headers, and media types.

```java
// Maps ALL HTTP methods to /users
@RequestMapping("/users")

// Maps only GET requests to /users
@RequestMapping(value = "/users", method = RequestMethod.GET)

// Modern shorthand annotations (preferred):
@GetMapping("/users")       // GET
@PostMapping("/users")      // POST
@PutMapping("/users/{id}")  // PUT
@DeleteMapping("/users/{id}") // DELETE
@PatchMapping("/users/{id}")  // PATCH
```

**At Class Level + Method Level:**
```java
@Controller
@RequestMapping("/api/users")          // Base path
public class UserController {

    @GetMapping("/{id}")               // Full path: GET /api/users/{id}
    public String getUser(@PathVariable int id, Model model) { ... }

    @PostMapping                       // Full path: POST /api/users
    public String createUser(...) { ... }
}
```

---

## 10. What is Dispatcher Servlet?

**DispatcherServlet** is the **Front Controller** of Spring MVC. It is a single central servlet that **intercepts all incoming HTTP requests** and orchestrates the entire request processing flow — from finding the right controller to rendering the final view.

| Responsibility | Description |
|---|---|
| **Request Interception** | Receives all HTTP requests matching its URL pattern |
| **Handler Mapping** | Delegates to `HandlerMapping` to find the right controller |
| **Controller Invocation** | Calls the appropriate controller method |
| **View Resolution** | Delegates to `ViewResolver` to find the view file |
| **Exception Handling** | Delegates to `HandlerExceptionResolver` for error handling |

```
All Requests → DispatcherServlet → (routes to) → Right Controller
```

**Real World Analogy:** The DispatcherServlet is like the **main reception desk** of a large office building. Every visitor (request) comes through reception, which then directs them to the right department (controller).

---

## 11. How do you Set up Dispatcher Servlet?

### Spring Boot (Automatic — Zero Config)
Spring Boot **auto-configures** `DispatcherServlet` mapped to `/` by default. Nothing needed!

```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

### Traditional Spring (Java Config)
```java
public class WebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{RootConfig.class};
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{WebConfig.class};
    }

    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};  // Map DispatcherServlet to all URLs
    }
}
```

### Traditional Spring (web.xml)
```xml
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

---

## 12. What is a Form Backing Object?

A **Form Backing Object** (also called a **Command Object**) is a **plain Java class (POJO)** that is bound to an HTML form in Spring MVC. When a form is submitted, Spring automatically **maps form field values to the corresponding fields of this object**.

```java
// Form Backing Object
public class UserForm {
    private String name;
    private String email;
    private int age;
    // getters and setters
}
```

```java
// Controller
@GetMapping("/register")
public String showForm(Model model) {
    model.addAttribute("userForm", new UserForm()); // Bind empty object to form
    return "register";
}

@PostMapping("/register")
public String submitForm(@ModelAttribute UserForm userForm) {
    // userForm is auto-populated with submitted form data
    userService.save(userForm);
    return "success";
}
```

```html
<!-- JSP/Thymeleaf Form -->
<form:form modelAttribute="userForm" action="/register" method="post">
    <form:input path="name" />
    <form:input path="email" />
    <form:input path="age" />
    <button type="submit">Register</button>
</form:form>
```

---

## 13. How is Validation Done Using Spring MVC?

Spring MVC uses **JSR-303/380 Bean Validation** via annotations like `@NotNull`, `@Size`, `@Email`, etc., combined with `@Valid` in the controller to trigger validation automatically.

**Step 1: Add validation annotations to the model:**
```java
public class User {
    @NotNull(message = "Name is required")
    @Size(min = 2, max = 50, message = "Name must be 2-50 characters")
    private String name;

    @Email(message = "Invalid email format")
    @NotNull(message = "Email is required")
    private String email;

    @Min(value = 18, message = "Age must be at least 18")
    private int age;
    // getters and setters
}
```

**Step 2: Trigger validation in controller with `@Valid`:**
```java
@PostMapping("/register")
public String register(@Valid @ModelAttribute User user, BindingResult result) {
    if (result.hasErrors()) {
        return "register-form"; // Return to form if validation fails
    }
    userService.save(user);
    return "redirect:/success";
}
```

---

## 14. What is BindingResult?

**BindingResult** is a Spring MVC interface that **holds the results of data binding and validation** for a form-backing object. It captures all validation errors and binding errors (e.g., type mismatch) after `@Valid` is processed.

```java
@PostMapping("/register")
public String register(@Valid @ModelAttribute User user, BindingResult result) {
    // Check if there are any validation errors
    if (result.hasErrors()) {
        System.out.println("Total errors: " + result.getErrorCount());
        result.getAllErrors().forEach(e -> System.out.println(e.getDefaultMessage()));
        return "register-form"; // Stay on the form
    }
    return "redirect:/success";
}
```

> **Important:** `BindingResult` must **immediately follow** the `@ModelAttribute` / `@Valid` parameter in the method signature. Placing it elsewhere causes a runtime error.

---

## 15. How do you Map Validation Results to Your View?

Spring MVC automatically makes validation errors available in the view via **Spring Form Tags**. The errors are bound to the model attribute field names.

```java
// Controller
@PostMapping("/register")
public String register(@Valid @ModelAttribute("user") User user, BindingResult result) {
    if (result.hasErrors()) {
        return "register-form"; // Model with errors is automatically passed to view
    }
    return "redirect:/success";
}
```

```html
<!-- JSP View using Spring Form Tags -->
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>

<form:form modelAttribute="user" action="/register" method="post">
    Name: <form:input path="name" />
    <form:errors path="name" cssClass="error" />  <!-- Shows: "Name is required" -->

    Email: <form:input path="email" />
    <form:errors path="email" cssClass="error" /> <!-- Shows: "Invalid email format" -->

    Age: <form:input path="age" />
    <form:errors path="age" cssClass="error" />   <!-- Shows: "Age must be at least 18" -->

    <button type="submit">Submit</button>
</form:form>
```

---

## 16. What are Spring Form Tags?

**Spring Form Tags** are a JSP tag library (`spring-form.tld`) that provide **data-binding-aware HTML form elements** tightly integrated with Spring MVC's model and validation.

| Tag | HTML Equivalent | Purpose |
|---|---|---|
| `<form:form>` | `<form>` | Binds form to a model attribute |
| `<form:input>` | `<input type="text">` | Text input bound to model field |
| `<form:password>` | `<input type="password">` | Password input |
| `<form:checkbox>` | `<input type="checkbox">` | Checkbox bound to boolean field |
| `<form:radiobutton>` | `<input type="radio">` | Radio button |
| `<form:select>` | `<select>` | Dropdown list |
| `<form:textarea>` | `<textarea>` | Multi-line text input |
| `<form:errors>` | N/A | Displays validation error messages |
| `<form:hidden>` | `<input type="hidden">` | Hidden field |

```html
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>

<form:form modelAttribute="user" action="/save" method="post">
    <form:input path="name" placeholder="Enter name" />
    <form:errors path="name" />

    <form:select path="role">
        <form:option value="ADMIN">Admin</form:option>
        <form:option value="USER">User</form:option>
    </form:select>

    <form:checkbox path="active" /> Active
    <button type="submit">Save</button>
</form:form>
```

---

## 17. What is a Path Variable?

`@PathVariable` is a Spring MVC annotation used to **extract a value from the URI path** and bind it to a method parameter. It is commonly used in RESTful APIs where resource identifiers are part of the URL.

```java
// URL: GET /users/42
@GetMapping("/users/{id}")
public String getUserById(@PathVariable int id, Model model) {
    model.addAttribute("user", userService.findById(id)); // id = 42
    return "user-detail";
}

// Multiple path variables
// URL: GET /users/42/orders/7
@GetMapping("/users/{userId}/orders/{orderId}")
public String getOrder(@PathVariable int userId, @PathVariable int orderId, Model model) {
    model.addAttribute("order", orderService.find(userId, orderId));
    return "order-detail";
}
```

| Aspect | `@PathVariable` | `@RequestParam` |
|---|---|---|
| Location in URL | Part of the path: `/users/42` | Query string: `/users?id=42` |
| Usage | RESTful resource identification | Filtering, sorting, pagination |
| Required by default | ✅ Yes | ❌ No (can be optional) |

---

## 18. What is a Model Attribute?

`@ModelAttribute` is a Spring MVC annotation with **two uses**:

### Use 1: Bind form data to a method parameter
```java
@PostMapping("/register")
public String register(@ModelAttribute User user) {
    // Spring auto-maps form fields to User object
    userService.save(user);
    return "redirect:/success";
}
```

### Use 2: Pre-populate model data for all methods in a controller
```java
@Controller
public class UserController {

    @ModelAttribute("roles")  // Added to model BEFORE every request in this controller
    public List<String> populateRoles() {
        return Arrays.asList("ADMIN", "USER", "MANAGER");
    }

    @GetMapping("/register")
    public String showForm(Model model) {
        // "roles" is already in the model — no need to add it manually
        return "register-form";
    }
}
```

---

## 19. What is a Session Attribute?

`@SessionAttributes` is a Spring MVC annotation used to **store specific model attributes in the HTTP session** across multiple requests. It is useful for **multi-step forms** where data needs to persist across pages.

```java
@Controller
@SessionAttributes("user")   // Store "user" in session
public class RegistrationController {

    @GetMapping("/step1")
    public String step1(Model model) {
        model.addAttribute("user", new User()); // Added to both model and session
        return "step1";
    }

    @PostMapping("/step2")
    public String step2(@ModelAttribute User user, Model model) {
        // "user" is retrieved from session, updated with step2 form data
        return "step2";
    }

    @PostMapping("/complete")
    public String complete(@ModelAttribute User user, SessionStatus status) {
        userService.save(user);
        status.setComplete(); // Clear the session attribute
        return "redirect:/success";
    }
}
```

---

## 20. What is Init Binder?

`@InitBinder` is a Spring MVC annotation that marks a method to **customize the data binding process** for a specific controller. It is used to register custom editors or formatters — most commonly to define **date formats**, restrict binding to specific fields, or apply custom type conversion.

```java
@Controller
public class UserController {

    @InitBinder
    public void initBinder(WebDataBinder binder) {
        // Register custom date format for this controller
        SimpleDateFormat dateFormat = new SimpleDateFormat("dd/MM/yyyy");
        dateFormat.setLenient(false);
        binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));

        // Optionally restrict which fields can be bound (security)
        binder.setAllowedFields("name", "email", "age");
    }
}
```

---

## 21. How do you Set Default Date Format with Spring?

There are multiple ways to configure a default date format in Spring MVC:

### Using @InitBinder (Controller-level)
```java
@InitBinder
public void initBinder(WebDataBinder binder) {
    SimpleDateFormat dateFormat = new SimpleDateFormat("dd-MM-yyyy");
    binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
}
```

### Using @DateTimeFormat (Field-level)
```java
public class User {
    @DateTimeFormat(pattern = "dd/MM/yyyy")
    private Date birthDate; // Spring parses "25/03/1995" → Date object
}
```

### Global Format via Spring Boot (application.properties)
```properties
spring.mvc.format.date=dd/MM/yyyy
spring.mvc.format.date-time=dd/MM/yyyy HH:mm:ss
```

---

## 22. How do you Implement Common Logic for Controllers?

Common logic (e.g., adding common model data, logging, date binding) that applies to **multiple controllers** can be centralized using:

### @ControllerAdvice (Recommended)
```java
@ControllerAdvice
public class GlobalControllerAdvice {

    // Adds "appName" to the model of EVERY controller method
    @ModelAttribute("appName")
    public String addAppName() {
        return "My Spring App";
    }

    // Applies date binding to ALL controllers
    @InitBinder
    public void globalInitBinder(WebDataBinder binder) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("dd/MM/yyyy");
        binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
    }
}
```

### Base Controller Class (Inheritance approach)
```java
public abstract class BaseController {
    @ModelAttribute("currentYear")
    public int addCurrentYear() {
        return LocalDate.now().getYear();
    }
}

@Controller
public class UserController extends BaseController { ... }
```

---

## 23. What is Controller Advice?

`@ControllerAdvice` is a Spring MVC annotation that marks a class as a **global component that applies cross-cutting concerns to all controllers** (or a subset). It centralizes:

1. **Global Exception Handling** — via `@ExceptionHandler`
2. **Global Model Attributes** — via `@ModelAttribute`
3. **Global Init Binder** — via `@InitBinder`

```java
@ControllerAdvice                        // Applies to ALL controllers
// @ControllerAdvice("com.example.web") // Applies to specific package only
public class GlobalControllerAdvice {

    // Global exception handler
    @ExceptionHandler(UserNotFoundException.class)
    public String handleUserNotFound(UserNotFoundException ex, Model model) {
        model.addAttribute("errorMessage", ex.getMessage());
        return "error-page";
    }

    // Global model attribute
    @ModelAttribute("appVersion")
    public String appVersion() {
        return "v2.0";
    }

    // Global init binder
    @InitBinder
    public void initBinder(WebDataBinder binder) {
        binder.registerCustomEditor(Date.class,
            new CustomDateEditor(new SimpleDateFormat("dd/MM/yyyy"), true));
    }
}
```

> **`@RestControllerAdvice`** = `@ControllerAdvice` + `@ResponseBody` — used for REST APIs where exception handlers return JSON instead of view names.

---

## 24. What is @ExceptionHandler?

`@ExceptionHandler` is a Spring MVC annotation that marks a method to **handle specific exceptions** thrown by controller methods. It intercepts the exception and returns a custom error response (view or JSON) instead of showing a generic error page.

```java
@Controller
public class UserController {

    // Handles exception only within THIS controller
    @ExceptionHandler(UserNotFoundException.class)
    public String handleNotFound(UserNotFoundException ex, Model model) {
        model.addAttribute("error", ex.getMessage());
        return "error/not-found";
    }

    @ExceptionHandler(Exception.class) // Fallback for any other exception
    public String handleGeneric(Exception ex, Model model) {
        model.addAttribute("error", "Something went wrong!");
        return "error/generic";
    }
}
```

For **REST APIs**, return JSON error response:
```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public Map<String, String> handleNotFound(UserNotFoundException ex) {
        return Map.of("error", ex.getMessage(), "status", "404");
    }
}
```

---

## 25. How to Handle Exceptions for Web Applications?

Spring MVC provides **three levels** of exception handling:

| Level | Mechanism | Scope |
|---|---|---|
| **Controller-level** | `@ExceptionHandler` inside controller | Only that controller |
| **Global-level** | `@ExceptionHandler` inside `@ControllerAdvice` | All controllers |
| **Servlet-level** | `SimpleMappingExceptionResolver` or `error pages in web.xml` | Entire web app |

### Global Handler (Recommended)
```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public String handleNotFound(ResourceNotFoundException ex, Model model) {
        model.addAttribute("message", ex.getMessage());
        return "error/404";
    }

    @ExceptionHandler(AccessDeniedException.class)
    @ResponseStatus(HttpStatus.FORBIDDEN)
    public String handleForbidden(Model model) {
        model.addAttribute("message", "You don't have permission.");
        return "error/403";
    }

    @ExceptionHandler(Exception.class) // Catch-all
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public String handleAll(Exception ex, Model model) {
        model.addAttribute("message", "An unexpected error occurred.");
        return "error/500";
    }
}
```

### Spring Boot — Custom Error Page
```properties
# application.properties
server.error.whitelabel.enabled=false  # Disable default error page
```
Create `/resources/templates/error/404.html`, `500.html` — Spring Boot auto-maps them.

---

## 26. Important Things in Exception Handling?

| Best Practice | Why it Matters |
|---|---|
| **Always have a catch-all handler** | `@ExceptionHandler(Exception.class)` prevents unhandled exceptions from leaking stack traces to users |
| **Use `@ResponseStatus`** | Sets the correct HTTP status code (404, 500, etc.) — important for REST APIs and SEO |
| **Separate concerns** | Use `@ControllerAdvice` to keep exception handling out of business logic |
| **Never expose stack traces to users** | Log internally, show user-friendly messages |
| **Use custom exception classes** | `UserNotFoundException extends RuntimeException` — makes handlers more specific and readable |
| **Log all exceptions** | Use `Logger` to log exceptions with context (user, URL, etc.) for debugging |
| **Return consistent error structure for REST** | Always return a standard error response body: `{ "status", "message", "timestamp" }` |
| **Handle validation errors separately** | `MethodArgumentNotValidException` for `@Valid` failures — return field-level error messages |

```java
// Custom Exception
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(int id) {
        super("User not found with id: " + id);
    }
}

// Standard Error Response for REST
public class ErrorResponse {
    private int status;
    private String message;
    private LocalDateTime timestamp;
    // constructor, getters
}
```

---

## 27. How to Implement Specific Error Handling in Controller?

For **controller-specific** exception handling, place `@ExceptionHandler` directly inside the controller class. It handles exceptions **only thrown from that controller**.

```java
@Controller
@RequestMapping("/products")
public class ProductController {

    @GetMapping("/{id}")
    public String getProduct(@PathVariable int id, Model model) {
        Product product = productService.findById(id);
        if (product == null) throw new ProductNotFoundException(id);
        model.addAttribute("product", product);
        return "product-detail";
    }

    @PostMapping
    public String createProduct(@Valid @ModelAttribute Product product,
                                 BindingResult result) {
        if (result.hasErrors()) return "product-form";
        productService.save(product);
        return "redirect:/products";
    }

    // Handles ProductNotFoundException ONLY from this controller
    @ExceptionHandler(ProductNotFoundException.class)
    public String handleProductNotFound(ProductNotFoundException ex, Model model) {
        model.addAttribute("errorMessage", ex.getMessage());
        return "error/product-not-found";
    }

    // Handles any other exception from this controller
    @ExceptionHandler(Exception.class)
    public String handleGeneralError(Exception ex, Model model) {
        model.addAttribute("errorMessage", "Error in product module: " + ex.getMessage());
        return "error/general";
    }
}
```

> **Rule:** Controller-level `@ExceptionHandler` takes priority over `@ControllerAdvice` global handler for that controller.

---

## 28. Why is Spring MVC Popular?

| Reason | Explanation |
|---|---|
| **Clean MVC Architecture** | Clear separation of Controller, Model, and View — easy to maintain and scale |
| **Annotation-driven** | `@Controller`, `@GetMapping`, `@PostMapping` reduce boilerplate significantly |
| **Flexible View Support** | Works with JSP, Thymeleaf, FreeMarker, JSON, XML, PDF — not tied to one view technology |
| **Powerful Data Binding** | Automatically maps HTTP request parameters to Java objects (`@ModelAttribute`) |
| **Built-in Validation** | Seamless integration with JSR-303/380 Bean Validation (`@Valid`, `BindingResult`) |
| **REST API Support** | `@RestController`, `@ResponseBody`, `@PathVariable` make REST API development effortless |
| **Centralized Exception Handling** | `@ControllerAdvice` + `@ExceptionHandler` for clean, global error management |
| **Spring Ecosystem Integration** | Works seamlessly with Spring Security, Spring Data, Spring Boot |
| **Testability** | `MockMvc` enables full controller testing without deploying to a server |
| **Massive Community** | Extensive documentation, tutorials, and enterprise adoption worldwide |

---

*Happy Learning Spring MVC! 🌱*
