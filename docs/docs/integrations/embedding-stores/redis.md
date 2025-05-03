---
sidebar_position: 22
---

# Redis

https://redis.io/


## Maven Dependency

You can use Redis with LangChain4j in plain Java or Spring Boot applications.

### Plain Java

:::note
Since `1.0.0-beta1`, `langchain4j-redis` has migrated to `langchain4j-community` and is renamed to
`langchain4j-community-redis`.
:::

Before `1.0.0-beta1`:

```xml

<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-redis</artifactId>
    <version>${previous version here}</version>
</dependency>
```

`1.0.0-beta1` and later:

```xml

<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-community-redis</artifactId>
    <version>${latest version here}</version>
</dependency>
```

### Spring Boot

:::note
Since `1.0.0-beta1`, `langchain4j-redis-spring-boot-starter` has migrated to `langchain4j-community` and is renamed
to `langchain4j-community-redis-spring-boot-starter`.
:::

Before `1.0.0-beta1`:

```xml

<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-redis-spring-boot-starter</artifactId>
    <version>${previous version here}</version>
</dependency>
```

`1.0.0-beta1` and later:

```xml

<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-community-redis-spring-boot-starter</artifactId>
    <version>${latest version here}</version>
</dependency>
```

Or, you can use BOM to manage dependencies consistently:

```xml

<dependencyManagement>
    <dependency>
        <groupId>dev.langchain4j</groupId>
        <artifactId>langchain4j-community-bom</artifactId>
        <version>${latest version here}</version>
        <typ>pom</typ>
        <scope>import</scope>
    </dependency>
</dependencyManagement>
```


## APIs

- `RedisEmbeddingStore` - Store and query vector embeddings using Redis Stack
- `RedisSemanticCache` - Cache LLM responses based on semantic similarity
- `RedisEmbeddingCache` - Cache embedding vectors to avoid regeneration
- `RedisSemanticRouter` - Route queries to specialized handlers based on semantic similarity
- `RedisFilterBuilder` - Type-safe API for building Redis filters

## Features

### Embedding Store

Redis integration in LangChain4j provides a robust vector store implementation that leverages Redis Stack's vector search capabilities. You can:

- Store and retrieve embeddings with associated metadata
- Perform similarity searches based on vector distance
- Filter results using metadata
- Easily configure index settings and similarity metrics

Example:

```java
RedisEmbeddingStore embeddingStore = RedisEmbeddingStore.builder()
        .redis(jedis)
        .indexName("my-index")
        .build();
```

### Semantic Caching

The `RedisSemanticCache` provides efficient caching of LLM responses based on the semantic similarity of queries:

- Cache chat completions from OpenAI and other LLMs
- Configurable similarity threshold to control cache hits
- TTL for cache entries
- Filtering by LLM model ID

Example:

```java
RedisSemanticCache cache = RedisSemanticCache.builder()
        .redis(jedis)
        .similarityThreshold(0.85f)
        .ttl(Duration.ofHours(24))
        .build();

// Add to a chat model
ChatLanguageModel chatModel = OpenAiChatModel.builder()
        .apiKey(System.getenv("OPENAI_API_KEY"))
        .cache(cache)
        .build();
```

### Embedding Cache

The `RedisEmbeddingCache` prevents redundant embedding generation by caching vector embeddings:

- Wrap any embedding model for transparent caching
- Configurable TTL and max cache size
- Custom key prefixes for organization

Example:

```java
EmbeddingModel baseModel = new AllMiniLmL6V2EmbeddingModel();

RedisEmbeddingCache cachedModel = RedisEmbeddingCache.builder()
        .redis(jedis)
        .embeddingModel(baseModel)
        .keyPrefix("embeddings:")
        .ttl(Duration.ofDays(7))
        .maxKeys(10000)
        .build();
```

### Semantic Router

The `RedisSemanticRouter` routes queries to specialized handlers based on semantic similarity:

- Route queries to different handlers/assistants
- Configure reference examples for each route
- Customize similarity thresholds

Example:

```java
Route techRoute = Route.builder()
        .name("tech")
        .references(
                "How do I fix my computer?",
                "What's the latest smartphone?",
                "How do I set up my wifi network?"
        )
        .build();

RedisSemanticRouter router = RedisSemanticRouter.builder()
        .redis(jedis)
        .embeddingModel(embeddingModel)
        .defaultRouteName("general")
        .addRoute(techRoute)
        .build();

// Route a query
String routeName = router.route("My iPhone won't turn on");
```

### Advanced Filtering

The Redis integration provides a type-safe API for building complex filters with tag, numeric, and text conditions:

```java
// Tag filters for exact matching
FilterExpression categoryFilter = RedisFilter.tag("category").equalTo("electronics");
FilterExpression languageFilter = RedisFilter.tag("language").in("English", "Spanish");

// Numeric filters for ranges
FilterExpression priceFilter = RedisFilter.numeric("price").between(50, 200);
FilterExpression ratingFilter = RedisFilter.numeric("rating").greaterThanOrEqualTo(4.0);

// Text filters for content search
FilterExpression textFilter = RedisFilter.text("description").contains("premium");

// Combine filters with AND/OR/NOT logic
FilterExpression combinedFilter = categoryFilter
        .and(priceFilter)
        .and(languageFilter.or(textFilter))
        .and(RedisFilter.tag("author").notEqualTo("Unknown"));

// Use in search
Filter filter = new RedisFilterExpression(combinedFilter);
List<EmbeddingMatch<TextSegment>> matches = embeddingStore.search(searchRequest).matches();
```

The filters support various operations:
- Tag fields: `equalTo()`, `notEqualTo()`, `in()`, `notIn()`
- Numeric fields: `equalTo()`, `lessThan()`, `greaterThan()`, `between()` 
- Text fields: `exactMatch()`, `contains()`, `startsWith()`, `fuzzyMatch()`
- Timestamp fields: `before()`, `after()`, `between()`, `onDate()`
- Geo fields: `withinRadius()`, `withinBox()`

## Examples

- [RedisEmbeddingStoreExample](https://github.com/langchain4j/langchain4j-examples/blob/main/redis-example/src/main/java/RedisEmbeddingStoreExample.java) - Basic embedding store operations
- [SimpleRedisStoreTest](https://github.com/langchain4j/langchain4j-examples/blob/main/redis-example/src/main/java/SimpleRedisStoreTest.java) - Embedding store with metadata and custom schema
- [RedisSemanticCacheExample](https://github.com/langchain4j/langchain4j-examples/blob/main/redis-example/src/main/java/RedisSemanticCacheExample.java) - Caching LLM responses based on semantic similarity using OpenAI or a mock model
- [RedisEmbeddingCacheExample](https://github.com/langchain4j/langchain4j-examples/blob/main/redis-example/src/main/java/RedisEmbeddingCacheExample.java) - Caching embedding vectors for performance optimization
- [RedisSemanticRouterExample](https://github.com/langchain4j/langchain4j-examples/blob/main/redis-example/src/main/java/RedisSemanticRouterExample.java) - Creating a comprehensive routing system with handlers based on semantic similarity
- [RedisAdvancedFilteringExample](https://github.com/langchain4j/langchain4j-examples/blob/main/redis-example/src/main/java/RedisAdvancedFilteringExample.java) - Advanced filtering with tag, numeric, and text filters in a product catalog scenario
