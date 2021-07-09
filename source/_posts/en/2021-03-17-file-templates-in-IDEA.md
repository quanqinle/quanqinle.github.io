---
layout:       post
title:        "Java | Use IDEA's file template function to simplify Spring Boot class creation"
subtitle:     "When adding an entity to a Spring Boot project, you generally need to create multiple classes by the way. Use IntelliJ's `File and Code Templates` to create all classes in one click"
date:         2021-03-17 12:00:00
updated:      2021-06-21 17:07:00
author:       "Qinle Quan"
header-img:   "img/home-bg.webp"
lang:         en
catalog:      true
categories:
    - Code
tags:
    - Java
    - IDEA
    - Spring Boot

---

# Environment
+ IntelliJ IDEA Community 2020.3.3

# Introduction

In the Spring Boot projects, whenever a new entity/module object is added, such as `UserPO.java`, the next step is usually to create the corresponding repository, service, service implement, controller, etc., and the initial contents of these files are all similar. It is real a waste of time and a repetitive work to create the set of template classes.

So, I wondered if there was a way to create multiple class files at once with "one-click" from a class template or something. When I looked up `Settings`, I found the solution in `File and Code Templates`. This article describes how to create a batch class files from a file template step by step.

<!-- more -->

# Expectations and Results

Let's look at the effect of the final implementation first, that is, the initial requirement

1. On the root directory of the package, right click -> `New` -> select the new template,
   ![right click](/images/in-post/file-templates-in-IDEA/right-click-new-entity.webp)
2. Enter the `entity name`, such as `User`, with its first letter in uppercase
   ![input entity](/images/in-post/file-templates-in-IDEA/input-entity.webp)
3. Generate the following new files:
   + entity/po : `User.java`
   + dao : `UserRepository.java`
   + service : `UserService.java`
   + service/impl : `UserServiceImpl.java`
   + controller : `UserController.java`

The corresponding project structure is as follows, and the configuration below is based on this structure.
```text
src/main/java
└── com.github.quanqinle
        ├── Application.java
        ├── entity
        │   ├── po
        │   │   └── User.java
        │   └── vo
        ├── dao
        │   └── UserRepository.java
        ├── service
        │   ├── impl
        │   │   └── UserServiceImpl.java
        │   └── UserService.java
        └── controller
            └── UserController.java
```

# Setting

This is the final configuration, just as shown in (1) in the `Picture 1`:

![final settings](/images/in-post/file-templates-in-IDEA/final-settings.webp)
<center style="color:#C0C0C0;">Picture 1</center> 

## Setting po template
Open `Settings` dialog, find `Editor` -> `File and Code templates`, then under the 'Files' category, click <kbd>Create Template</kbd>, that is, button (2) in `Picture 1`.

+ Name: template name, which will be used when right-clicking, such as `Create whole classes in package root`
+ Extension: java default
+ File Name: file path and file name (Don't add .java), `./entity/po/${Subject}`
+ Enter the content of the template as below:

```java
#set($SubjectOfLowerFirst = ${Subject.substring(0,1).toLowerCase()} + $Subject.substring(1))
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME}.entity.po;#end

import lombok.Data;
import javax.persistence.*;
import java.time.LocalDateTime;

#parse("File Header.java")
@Data
@Entity
@Table(name = "${SubjectOfLowerFirst}")
public class ${Subject} {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    /**
     * name
     */
    private String name;

    @Column(name = "create_time")
    private LocalDateTime createTime;
    @Column(name = "update_time")
    private LocalDateTime updateTime;
}
```

## Setting dao template
Select `po Template` first which was created in the step 1, then click <kbd>Create Child Template File</kbd>，that is, button (3) in `Picture 1`.

+ Extension: java default
+ File Name: file path and file name (Don't add .java), `./dao/${Subject}Repository`
+ Enter the content of the template as below:

```java
#set($SubjectOfLowerFirst = ${Subject.substring(0,1).toLowerCase()} + $Subject.substring(1))
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME}.dao;#end

import ${PACKAGE_NAME}.entity.po.${Subject};
import org.springframework.stereotype.Repository;
import org.springframework.data.jpa.repository.JpaRepository;

#parse("File Header.java")
@Repository
public interface ${Subject}Repository extends JpaRepository<${Subject}, Long> {
}
```

## Setting service template

Select `po Template` first which was created in the step 1, then click <kbd>Create Child Template File</kbd>，that is, button (3) in `Picture 1`.

+ Extension: java default
+ File Name: file path and file name (Don't add .java), `./service/${Subject}Service`
+ Enter the content of the template as below:

```java
#set($SubjectOfLowerFirst = ${Subject.substring(0,1).toLowerCase()} + $Subject.substring(1))
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME}.service;#end

import ${PACKAGE_NAME}.entity.po.${Subject};
import java.util.Optional;

#parse("File Header.java")
public interface ${Subject}Service {

    /**
     * insert
     * @param ${SubjectOfLowerFirst} a ${Subject} object
     * @return
     */
    ${Subject} insert(${Subject} ${SubjectOfLowerFirst});

    /**
     * update
     * @param ${SubjectOfLowerFirst} a ${Subject} object
     * @return
     */
    ${Subject} update(${Subject} ${SubjectOfLowerFirst});
    
    /**
     * query by id
     * @param id ${Subject} id
     * @return
     */
    Optional<${Subject}> queryById(Long id);
    
    /**
     * delete by id
     * @param id ${Subject} id
     * @return
     */
    boolean deleteById(Long id);
    
}
```

## Setting service implement template
Select `po Template` first which was created in the step 1, then click <kbd>Create Child Template File</kbd>，that is, button (3) in `Picture 1`.

+ Extension: java default
+ File Name: file path and file name (Don't add .java), `./service/impl/${Subject}ServiceImpl`
+ Enter the content of the template as below:

```java
#set($SubjectOfLowerFirst = ${Subject.substring(0,1).toLowerCase()} + $Subject.substring(1))
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME}.service.impl;#end

import ${PACKAGE_NAME}.dao.${Subject}Repository;
import ${PACKAGE_NAME}.entity.po.${Subject};
import ${PACKAGE_NAME}.service.${Subject}Service;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.util.Optional;

#parse("File Header.java")
@Service
@Transactional(rollbackFor = Exception.class)
public class ${Subject}ServiceImpl implements ${Subject}Service {

    private Logger log = LoggerFactory.getLogger(${Subject}Service.class);

    @Autowired
    private ${Subject}Repository repository;

    public ${Subject}ServiceImpl() {
    }

    @Override
    public ${Subject} insert(${Subject} ${SubjectOfLowerFirst}) {
        return repository.save(${SubjectOfLowerFirst});
    }

    @Override
    public ${Subject} update(${Subject} ${SubjectOfLowerFirst}) {
        return repository.save(${SubjectOfLowerFirst});
    }

    @Override
    public boolean deleteById(Long id) {
        boolean result = true;
        try {
            repository.deleteById(id);
        } catch (Exception e) {
            result = false;
        }
        return result;
    }

    @Override
    public Optional<${Subject}> queryById(Long id) {
        return repository.findById(id);
    }

}
```

## Setting controller template
Select `po Template` first which was created in the step 1, then click <kbd>Create Child Template File</kbd>，that is, button (3) in `Picture 1`.

+ Extension: java default
+ File Name: file path and file name (Don't add .java), `./controller/${Subject}Controller`
+ Enter the content of the template as below:

```java
#set($SubjectOfLowerFirst = ${Subject.substring(0,1).toLowerCase()} + $Subject.substring(1))
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME}.controller;#end

import ${PACKAGE_NAME}.entity.Result;
import ${PACKAGE_NAME}.entity.po.${Subject};
import ${PACKAGE_NAME}.service.${Subject}Service;
import org.springframework.web.bind.annotation.*;
import javax.annotation.Resource;

#parse("File Header.java")
@RestController
@RequestMapping("api/${SubjectOfLowerFirst}")
public class ${Subject}Controller {

    @Resource
    private ${Subject}Service ${SubjectOfLowerFirst}Service;

    @PostMapping
    public Result<${Subject}> create(@RequestBody ${Subject} record) {
        ${Subject} ${SubjectOfLowerFirst} = ${SubjectOfLowerFirst}Service.insert(record);
        return Result.success(${SubjectOfLowerFirst});
    }

    @PutMapping
    public Result<${Subject}> update(@RequestBody ${Subject} record) {
        ${Subject} ${SubjectOfLowerFirst} = ${SubjectOfLowerFirst}Service.update(record);
        return Result.success(${SubjectOfLowerFirst});
    }

    @DeleteMapping("{id}")
    public Result<Void> deleteById(@PathVariable Long id) {
        boolean success = ${SubjectOfLowerFirst}Service.deleteById(id);
        return success ? Result.success() : Result.fail();
    }

    @GetMapping("{id}")
    public Result<${Subject}> queryById(@PathVariable Long id) {
        var optional = ${SubjectOfLowerFirst}Service.queryById(id);
        if (optional.isPresent()) {
            return Result.success(optional.get());
        } else {
            return Result.success();
        }
    }

}
```

Save the configuration.
