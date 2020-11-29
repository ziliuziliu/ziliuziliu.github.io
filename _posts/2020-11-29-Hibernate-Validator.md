---
layout: article
title: Hibernate Validator
tags: Articles
---

## 1. Why Hibernate Validator?
- Developers implement validation accross layers, which is time consuming and error-prone.
![avatar](https://docs.jboss.org/hibernate/validator/7.0/reference/en-US/html_single/images/application-layers.png)
- Validation code is really metadata of a class itself. Instead of bundle validation directly to code, we use **Annotations**.
![avatar](https://docs.jboss.org/hibernate/validator/7.0/reference/en-US/html_single/images/application-layers2.png)
## 2. Dependencies
- spring-boot-starter-validation
## 3. Declaring Bean Constraints
### 3.1 Field-level
```java
    @NotNull
    private String manufacturer;
    @AssertTrue
    private boolean isRegistered;
```
### 3.2 Property-level
- Only ***getter*** method can be annotated. Choose either field or property for annotation, not both.
```java
    @NotNull
    public String getManufacturer() {
        return manufacturer;
    }
```
### 3.3 Container Elements
- For List, Map, Set, Iterable things...
```java
    private Map<@NotNull FuelConsumption, @MaxAllowedFuelConsumption Integer> fuelConsumption = new HashMap<>();
```
### 3.4 Class-level
```java
@ValidPassengerCount
public class Car {
    private int seatCount;
    private List<Person> passengers;
}
```
## 4. Declaring Method Constraints
### 4.1 Parameter
```java
    public void rentCar(
            @NotNull Customer customer,
            @NotNull @Future Date startDate,
            @Min(1) int durationInDays) {
        //...
    }
```
### 4.2 Return Value
```java
    @NotNull
    @Size(min = 1)
    public List<@NotNull Customer> getCustomers() {
        //...
        return null;
    }
```
## 5. Common Annotations
- @Null, @NotNull, @AssertTrue, @AssertFalse
- @Min(value), @Max(value), @DecimalMin(value), @DecimalMax(value)
- @Past, @Future
- @Email
- @Pattern(regex)
- @Length(min,max), @Range(min,max)
## 6. Validating
### 6.1 Validating Bean Constraints
- Obtaining a ***Validator*** instance.
```java
    ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
    validator = factory.getValidator();
```
- Validate: use the ***validate()*** method to perform validation. It returns a ***Set\<ConstraintViolation>***, for each violation it add a ***ConstraintViolation***.
```java
    Car car = new Car( null, true );
    Set<ConstraintViolation<Car>> constraintViolations = validator.validate( car );
    assertEquals( 1, constraintViolations.size() );
    assertEquals( "must not be null", constraintViolations.iterator().next().getMessage() );
```
### 6.2 Validating Method Constraints
- Obtaining a ***ExecutableValidator*** instance.
- Validate: use ***validateParameters()*** and ***validateReturnValue()***.
## 7. Integration With Springboot Controller
- It's impossible to totally decouple validation code with bussiness logic. However, it's useful in Springboot Controller: lightweight, minimum modification of code, and maximum effect.
- ***@Validated*** at the start of controller class.
### 7.1 @RequestBody
- Use annotation metioned in part 2. Then:
```java
    @PostMapping("/User")
    public ReturnVO register(@RequestBody @Validated UserDTO userDTO) {...}
```
### 7.2 @RequestParam
```java
    @RequestMapping("/Register")
    public ReturnVO register(@Email @RequestParam(name = "email") String email,
                             @Pattern(regexp = "^1([38][0-9]|4[579]|5[0-3,5-9]|6[6]|7[0135678]|9[89])\\d{8}$", message = "205")
                             @RequestParam(name = "phone") String phone)
```
### 7.3 Handle Exception
- When violating constraint, the hibernate validator throws ***ConstraintViolationException***, which should be processed, and return user-friendly messages. A ***GlobalExceptionHandler*** would do the trick.
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ConstraintViolationException.class)
    public Msg<Boolean> handle(ConstraintViolationException e) {
        Set<ConstraintViolation<?>> violations = e.getConstraintViolations();
        for (ConstraintViolation<?> item : violations) {
            switch (item.getMessage()) {
                ...
            }
        }
    }
}
```
## 8. References
- [Hibernate-Validator-7.0.0.Alpha6-Doc](https://docs.jboss.org/hibernate/validator/7.0/reference/en-US/html_single/#_return_value_constraints)