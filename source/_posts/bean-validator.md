---
title: bean validator
toc: true
tags:
  - rest
date: 2018-07-18 10:30:57
---


bean validator 主要是验证一个 bean 的各字段是否满足一些约束，例如 `@NotNull`

bean validation 有个规范 [jsr 380](https://beanvalidation.org/2.0/)，里边定义了一堆 api。有很多规范的实现，最常用的是 [hibernate validator](http://hibernate.org/validator/)，jersey 出的 [jersey-bean-validation](https://jersey.github.io/documentation/latest/bean-validation.html) 也是基于 hibernate validator 做的。

bean validator 一般是应用在 web 框架（如 spring、jersey）上，框架在反序列化 rest 请求到 bean 对象时，框架会调用 validator 根据 bean 对象的 annotation 对 bean 进行验证。

这个过程也可以手动进行。可参考 [hibernate validator: get started](http://hibernate.org/validator/documentation/getting-started/)。

# 引入依赖

```groovy
// jsr 380 api
compile "javax.validation:validation-api:2.0.1.Final"

// hibernate vaidator 实现
testCompile "org.hibernate.validator:hibernate-validator:6.0.10.Final"

// hibernate validator 依赖的 JSR 341 实现
testCompile "org.glassfish:javax.el:3.0.0"
```

# 创建 bean，标明约束

```java
package org.hibernate.validator.referenceguide.chapter01;

import javax.validation.constraints.Min;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;

public class Car {

   @NotNull
   private String manufacturer;

   @NotNull
   @Size(min = 2, max = 14)
   private String licensePlate;

   @Min(2)
   private int seatCount;

   public Car(String manufacturer, String licencePlate, int seatCount){
      this.manufacturer = manufacturer;
      this.licensePlate = licencePlate;
      this.seatCount = seatCount;
   }
}
```

# 验证

1. 利用 `Validation` 获取默认的 `ValidatorFactory`：`Validation.buildDefaultValidatorFactory()`；
2. 从 `ValidatorFacotry` 获取 `Validator`：`factory.getValidator()`
3. 利用 `Validator` 验证 bean

```java
package org.hibernate.validator.referenceguide.chapter01;

import java.util.Set;
import javax.validation.ConstraintViolation;
import javax.validation.Validation;
import javax.validation.Validator;
import javax.validation.ValidatorFactory;

import org.junit.BeforeClass;
import org.junit.Test;

import static org.junit.Assert.assertEquals;

public class CarTest {

private static Validator validator;

   @BeforeClass
   public static void setUp() {
      ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
      validator = factory.getValidator();
   }

   @Test
   public void manufacturerIsNull() {
      Car car = new Car( null, "DD-AB-123", 4 );

      Set<ConstraintViolation<Car>> constraintViolations =
      validator.validate( car );

      assertEquals( 1, constraintViolations.size() );
      assertEquals(
         "may not be null",
         constraintViolations.iterator().next().getMessage()
      );
   }

   @Test
   public void licensePlateTooShort() {
      Car car = new Car( "Morris", "D", 4 );

      Set<ConstraintViolation<Car>> constraintViolations =
      validator.validate( car );

      assertEquals( 1, constraintViolations.size() );
      assertEquals(
         "size must be between 2 and 14",
         constraintViolations.iterator().next().getMessage()
      );
   }

   @Test
   public void seatCountTooLow() {
      Car car = new Car( "Morris", "DD-AB-123", 1 );

      Set<ConstraintViolation<Car>> constraintViolations =
      validator.validate( car );

      assertEquals( 1, constraintViolations.size() );
      assertEquals(
         "must be greater than or equal to 2",
         constraintViolations.iterator().next().getMessage()
      );
   }

   @Test
   public void carIsValid() {
      Car car = new Car( "Morris", "DD-AB-123", 2 );

      Set<ConstraintViolation<Car>> constraintViolations =
      validator.validate( car );

      assertEquals( 0, constraintViolations.size() );
   }
}
```