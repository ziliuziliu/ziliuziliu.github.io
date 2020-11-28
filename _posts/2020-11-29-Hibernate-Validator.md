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
- Hibernate-Validator is included in spring-boot-starter-web.

## 3. Declaring and Validating Bean Constraints

### 3.1 Field-level

