# Using RedisTemplate in Spring Boot

Spring Boot provides the `RedisTemplate` class to interact with Redis in a flexible and powerful way. This guide covers the basics of `RedisTemplate`, its configuration, and usage examples.

---

## 1. **What is RedisTemplate?**

`RedisTemplate` is a high-level abstraction that simplifies interaction with Redis data structures like Strings, Hashes, Lists, Sets, and Sorted Sets. It provides type-safe operations and allows working with custom objects.

---

## 2. **Configuring RedisTemplate**

To use `RedisTemplate`, it must be configured in the Spring context. The following example demonstrates how to configure it:

### Configuration Example

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);

        // Key Serializer
        template.setKeySerializer(new StringRedisSerializer());

        // Value Serializer
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());

        return template;
    }
}
```

### Explanation:

- **RedisConnectionFactory**: Manages connections to the Redis server.
- **StringRedisSerializer**: Serializes keys as strings.
- **GenericJackson2JsonRedisSerializer**: Serializes values as JSON, enabling easy handling of complex objects.

---

## 3. **Basic Operations with RedisTemplate**

Here are some common operations you can perform using `RedisTemplate`:

### Example: Storing and Retrieving Strings

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Service;

@Service
public class RedisService {

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    public void saveString(String key, String value) {
        redisTemplate.opsForValue().set(key, value);
    }

    public String getString(String key) {
        return (String) redisTemplate.opsForValue().get(key);
    }
}
```

### Explanation:

- **opsForValue()**: Provides operations for Redis String values.

### Example Usage:

```java
redisService.saveString("greeting", "Hello, Redis!");
String value = redisService.getString("greeting"); // Output: "Hello, Redis!"
```

---

## 4. **Working with Other Data Structures**

### Hashes

```java
public void saveHash(String key, String hashKey, Object value) {
    redisTemplate.opsForHash().put(key, hashKey, value);
}

public Object getHashValue(String key, String hashKey) {
    return redisTemplate.opsForHash().get(key, hashKey);
}
```

### Lists

```java
public void pushToList(String key, String value) {
    redisTemplate.opsForList().rightPush(key, value);
}

public String popFromList(String key) {
    return (String) redisTemplate.opsForList().leftPop(key);
}
```

### Sets

```java
public void addToSet(String key, String value) {
    redisTemplate.opsForSet().add(key, value);
}

public Set<Object> getSetMembers(String key) {
    return redisTemplate.opsForSet().members(key);
}
```

### Sorted Sets

```java
public void addToSortedSet(String key, String value, double score) {
    redisTemplate.opsForZSet().add(key, value, score);
}

public Set<Object> getSortedSet(String key) {
    return redisTemplate.opsForZSet().range(key, 0, -1);
}
```

---

## 5. **Working with Custom Objects**

You can store and retrieve custom objects by ensuring proper serialization.

### Example:

```java
import java.io.Serializable;

public class User implements Serializable {
    private String id;
    private String name;

    // Getters, Setters, Constructors
}
```

### Save and Retrieve

```java
public void saveUser(String key, User user) {
    redisTemplate.opsForValue().set(key, user);
}

public User getUser(String key) {
    return (User) redisTemplate.opsForValue().get(key);
}
```

---

## 6. **Expiration and TTL**

Redis supports time-to-live (TTL) for keys. Use `expire` to set expiration times.

### Example:

```java
import java.util.concurrent.TimeUnit;

public void saveWithExpiration(String key, String value, long timeout, TimeUnit unit) {
    redisTemplate.opsForValue().set(key, value, timeout, unit);
}

public void setExpiration(String key, long timeout, TimeUnit unit) {
    redisTemplate.expire(key, timeout, unit);
}
```

---

## 7. **Best Practices**

- Always use meaningful and unique keys to avoid conflicts.
- Configure appropriate serializers for performance and compatibility.
- Use TTL for cache entries to prevent memory overuse.
- Monitor Redis usage and tune configurations as needed.

---

## Summary

- **RedisTemplate** is a versatile tool for interacting with Redis in Spring Boot.
- It supports operations on Strings, Hashes, Lists, Sets, and Sorted Sets.
- Serialization and expiration configurations are essential for efficient usage.

This guide provides a foundation for working with `RedisTemplate`. Explore Redis data structures and Spring integrations further to unlock its full potential.
