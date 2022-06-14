---
title: 使用Hibernate建立一个带有分页的全文搜索的Spring Boot REST API
date: 2022-06-14 19:42:55
categories:
- Spring
- Hibernate
tags:
- Spring
- Hibernate
- API
---

{% asset_img 1.png 示意图 width="400" %}

在[之前的文章](https://pangz.fun/Build-a-Spring-Boot-REST-API-with-Full-Text-Search-using-Hibernate-Search.html)中，我们学习了如何使用Hibernate Search为Spring Boot Rest API添加全文搜索。

在这篇文章中，我们将在此基础上，学习如何向现有的REST API添加分页搜索。

<!--more-->

### 项目设置
你可以查看之前的博文，以获得关于如何使用Spring Initializer设置项目的详细攻略。

### 扩展数据模型
首先要解决的是找到一种方法来接收添加分页所需的新数据。

为此，我们可以扩展SearchRequestDTO。

```java
package com.mozen.springbootpaginatedsearch.model;

import lombok.Data;
import lombok.EqualsAndHashCode;

import javax.validation.constraints.Min;

@Data
@EqualsAndHashCode(callSuper = true)
public class PageableSearchRequestDTO extends SearchRequestDTO{

    @Min(0)
    private int pageOffset;
}
```
我们只需要定义一个新字段，pageOffset。这个字段是用来控制我们要查询的页面的索引。

我们还定义了一个新的PageDTO。这个数据结构用来保存我们分页搜索的结果。

```java
package com.mozen.springbootpaginatedsearch.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.List;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class PageDTO<T> {

    private List<T> content;
    private long total;
}
```

### 扩展数据层
我们在SearchRepository接口中声明一个新的searchPageBy函数。

```java
package com.mozen.springbootpaginatedsearch.repository;

import com.mozen.springbootpaginatedsearch.model.PageDTO;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.repository.NoRepositoryBean;

import java.io.Serializable;
import java.util.List;

@NoRepositoryBean
public interface SearchRepository<T, ID extends Serializable> extends JpaRepository<T, ID> {

    List<T> searchBy(String text, int limit, String... fields);

    PageDTO<T> searchPageBy(String text, int limit, int offset, String... fields);
}
```

这个签名与现有的 searchBy 函数非常相似。我们只是添加了新的偏移量参数，表示要查询的页面。

我们把这个变化复制到SearchRepositoryImpl类中。

```java
package com.mozen.springbootpaginatedsearch.repository;

import com.mozen.springbootpaginatedsearch.model.PageDTO;
import org.hibernate.search.engine.search.query.SearchResult;
import org.hibernate.search.mapper.orm.Search;
import org.hibernate.search.mapper.orm.session.SearchSession;
import org.springframework.data.jpa.repository.support.JpaEntityInformation;
import org.springframework.data.jpa.repository.support.SimpleJpaRepository;
import org.springframework.transaction.annotation.Transactional;

import javax.persistence.EntityManager;
import java.io.Serializable;
import java.util.List;

@Transactional
public class SearchRepositoryImpl<T, ID extends Serializable> extends SimpleJpaRepository<T, ID>
        implements SearchRepository<T, ID> {

    private final EntityManager entityManager;

    public SearchRepositoryImpl(Class<T> domainClass, EntityManager entityManager) {
        super(domainClass, entityManager);
        this.entityManager = entityManager;
    }

    public SearchRepositoryImpl(
            JpaEntityInformation<T, ID> entityInformation, EntityManager entityManager) {
        super(entityInformation, entityManager);
        this.entityManager = entityManager;
    }

    @Override
    public List<T> searchBy(String text, int limit, String... fields) {

        SearchResult<T> result = getSearchResult(text, limit, 0, fields);

        return result.hits();
    }

    @Override
    public PageDTO<T> searchPageBy(String text, int limit, int offset, String... fields) {
        SearchResult<T> result = getSearchResult(text, limit, offset, fields);

        return new PageDTO<T>(result.hits(), result.total().hitCount());
    }

    private SearchResult<T> getSearchResult(String text, int limit, int offset, String[] fields) {
        SearchSession searchSession = Search.session(entityManager);

        SearchResult<T> result =
                searchSession
                        .search(getDomainClass())
                        .where(f -> f.match().fields(fields).matching(text).fuzzy(2))
                        .fetch(offset, limit);
        return result;
    }
}
```
我们可以通过添加一个新的 "offset "参数来重新使用现有的 getSearchResult 方法。然后我们在Hibernate Search fetch()方法中使用这个参数，该方法已经提供了一个签名，接受offset参数用于分页的目的。

PageDTO是使用搜索查询的结果建立的。

### 扩展业务层
我们可以在现有逻辑的基础上，提取处理字段的部分进行搜索，以避免重复，然后根据我们使用searchPlant()方法或searchPlantPage()方法，在有或没有分页的情况下调用资源库函数。

```java
package com.mozen.springbootpaginatedsearch.service;

import com.mozen.springbootpaginatedsearch.model.PageDTO;
import com.mozen.springbootpaginatedsearch.model.Plant;
import com.mozen.springbootpaginatedsearch.repository.PlantRepository;
import org.springframework.stereotype.Service;

import java.util.Arrays;
import java.util.List;

@Service
public class PlantService {

    private PlantRepository plantRepository;

    private static final List<String> SEARCHABLE_FIELDS = Arrays.asList("name","scientificName","family");

    public PlantService(PlantRepository plantRepository) {
        this.plantRepository = plantRepository;
    }

    public List<Plant> searchPlants(String text, List<String> fields, int limit) {

        List<String> fieldsToSearchBy = getFieldsToSearchBy(fields);

        return plantRepository.searchBy(
                text, limit, fieldsToSearchBy.toArray(new String[0]));
    }

    public PageDTO<Plant> searchPlantPage(String text, List<String> fields, int limit, int pageOffset) {
        List<String> fieldsToSearchBy = getFieldsToSearchBy(fields);

        return plantRepository.searchPageBy(
                text, limit, pageOffset, fieldsToSearchBy.toArray(new String[0]));
    }

        // We extract the common logic in a separate function
    private List<String> getFieldsToSearchBy(List<String> fields) {
        List<String> fieldsToSearchBy = fields.isEmpty() ? SEARCHABLE_FIELDS : fields;

        boolean containsInvalidField = fieldsToSearchBy.stream(). anyMatch(f -> !SEARCHABLE_FIELDS.contains(f));

        if(containsInvalidField) {
            throw new IllegalArgumentException();
        }
        return fieldsToSearchBy;
    }
}
```

### 扩展网络层
在这里面没有什么可做的。

我们只需要一个新的端点，通过使用我们新的PageableSearchRequestDTO来接收分页的搜索请求，并返回一个PageDTO。

```java
package com.mozen.springbootpaginatedsearch.controller;

import com.mozen.springbootpaginatedsearch.model.PageDTO;
import com.mozen.springbootpaginatedsearch.model.PageableSearchRequestDTO;
import com.mozen.springbootpaginatedsearch.model.Plant;
import com.mozen.springbootpaginatedsearch.model.SearchRequestDTO;
import com.mozen.springbootpaginatedsearch.service.PlantService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@Slf4j
@RestController
@RequestMapping("/plant")
public class PlantController {

    private PlantService plantService;

    public PlantController(PlantService plantService) {
        this.plantService = plantService;
    }

    @GetMapping("/search")
    public List<Plant> searchPlants(SearchRequestDTO searchRequestDTO) {

        log.info("Request for plant search received with data : " + searchRequestDTO);

        return plantService.searchPlants(searchRequestDTO.getText(), searchRequestDTO.getFields(), searchRequestDTO.getLimit());
    }

    @GetMapping("/search/page")
    public PageDTO<Plant> searchPlantPage(PageableSearchRequestDTO pageableSearchRequestDTO) {

        log.info("Request for plant page search received with data : " + pageableSearchRequestDTO);

        return plantService.searchPlantPage(pageableSearchRequestDTO.getText(), pageableSearchRequestDTO.getFields(), pageableSearchRequestDTO.getLimit(), pageableSearchRequestDTO.getPageOffset());
    }
}
```

我们记录收到的请求数据并调用我们在PlantService中定义的新函数。

### 把它们放在一起
是时候测试我们的代码了!

我们可以用命令行启动我们的应用程序。

```
mvn spring-boot:run
```

{% asset_img 3.png 示意图 width="400" %}

与第一篇文章类似，我们可以使用Postman ...

或者我们可以使用一个简单的cUrl命令。

```
// Request page 1 with 2 items per page on all fields 
curl -X GET 'http://localhost:9000/plant/search?text=cherry&limit=2&pageOffset=1'
 
// Request page 2 with 3 items per page on scientificName field 
curl -X GET 'http://localhost:9000/plant/search?text=asian&limit=3&fields=name&fields=scientificName&pageOffset=2'
```

我们已经完成了! 我们的全文搜索实现现在支持分页了。

<!-- https://medium.com/javarevisited/build-a-spring-boot-rest-api-with-paginated-full-text-search-using-hibernate-search-1143438b6a1d -->