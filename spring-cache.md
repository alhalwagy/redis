# Spring Cache Annotations: @Cacheable, @CacheEvict, and @CachePut

Spring provides a comprehensive caching abstraction that allows applications to store method results in a cache and retrieve them efficiently. This guide covers the key annotations: **@Cacheable**, **@CacheEvict**, and **@CachePut**.

---

## 1. **@Cacheable**

The `@Cacheable` annotation is used to store the result of a method in the cache. If the same method is called with the same parameters, the cached value is returned instead of executing the method again.

### Key Features:

- Automatically stores method results in a cache.
- Avoids redundant method executions for the same inputs.

### Example:

```java
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class ProductService {

    @Cacheable("products")
    public Product getProductById(Long id) {
        // Simulate a time-consuming operation
        System.out.println("Fetching product with ID: " + id);
        return new Product(id, "Product Name", 100.0);
    }
}
```

### Explanation:

- **Cache Name**: "products".
- When the `getProductById` method is called with a specific `id`, the result is cached under the "products" cache.
- Subsequent calls with the same `id` will retrieve the cached value without executing the method.

---

## 2. **@CacheEvict**

The `@CacheEvict` annotation is used to remove entries from the cache. This is useful when cached data becomes stale and needs to be updated.

### Key Features:

- Removes specific or all entries from the cache.
- Supports conditional eviction and bulk eviction.

### Example:

```java
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.stereotype.Service;

@Service
public class ProductService {

    @CacheEvict(value = "products", key = "#id")
    public void deleteProduct(Long id) {
        System.out.println("Deleting product with ID: " + id);
        // Perform deletion logic here
    }

    @CacheEvict(value = "products", allEntries = true)
    public void clearCache() {
        System.out.println("Clearing all cached products");
    }
}
```

### Explanation:

- **Specific Eviction**: `key = "#id"` removes the cached value associated with the given `id`.
- **Bulk Eviction**: `allEntries = true` clears all entries from the "products" cache.

---

## 3. **@CachePut**

The `@CachePut` annotation is used to update the cache with a new value. Unlike `@Cacheable`, this annotation ensures the method is always executed and the result is cached.

### Key Features:

- Updates the cache with a fresh value.
- Ensures the method execution always occurs.

### Example:

```java
import org.springframework.cache.annotation.CachePut;
import org.springframework.stereotype.Service;

@Service
public class ProductService {

    @CachePut(value = "products", key = "#product.id")
    public Product updateProduct(Product product) {
        System.out.println("Updating product: " + product);
        // Perform update logic here
        return product;
    }
}
```

### Explanation:

- The method `updateProduct` executes every time it is called, and the returned value is cached under the "products" cache with the specified key.

---

## Comparison Table

| Annotation      | Purpose                            | Executes Method | Updates Cache | Removes Cache |
| --------------- | ---------------------------------- | --------------- | ------------- | ------------- |
| **@Cacheable**  | Stores method results in cache     | No (if cached)  | Yes           | No            |
| **@CacheEvict** | Removes entries from the cache     | Yes             | No            | Yes           |
| **@CachePut**   | Updates the cache with a new value | Yes             | Yes           | No            |

---

## 4. **Other Annotations**

Spring provides additional annotations to enhance caching:

### **@Caching**

- Combines multiple cache annotations for complex scenarios.

### Example:

```java
import org.springframework.cache.annotation.Caching;
import org.springframework.stereotype.Service;

@Service
public class ProductService {

    @Caching(
        cacheable = @Cacheable(value = "products", key = "#id"),
        evict = @CacheEvict(value = "allProducts", allEntries = true)
    )
    public Product getProductWithCacheManagement(Long id) {
        System.out.println("Fetching product with ID: " + id);
        return new Product(id, "Product Name", 100.0);
    }
}
```

### **@EnableCaching**

- Enables caching in a Spring Boot application.

### Example:

```java
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableCaching
public class CacheConfig {
}
```

---

## Summary

- Use **@Cacheable** for caching method results.
- Use **@CacheEvict** to remove stale entries from the cache.
- Use **@CachePut** to update the cache with fresh values.
- Combine annotations with **@Caching** for complex scenarios.
- Ensure caching is enabled using **@EnableCaching**.

This guide provides a comprehensive overview of caching annotations in Spring Boot to help optimize your application performance.
