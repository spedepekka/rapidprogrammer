---
layout: post
title: "More about AOP"
date: "2026-02-16 10:55:00 +0300"
permalink: /:title
---

Here is the original [AOP post]({% post_url 2026-02-13-from-interview-to-learn-aop-and-openapi %}).

And here is couple words about [correlation ID]({% post_url 2026-02-15-toying-with-correlation-id %}).

The code is in this [repo](https://github.com/spedepekka/soon-i-know-aop).

## What AOP is usually used for

Logging is one aspect AOP can tackle nicely.

Anothed good on is user authentication

```java
@Aspect
@Component
class UserAuthenticationAspect {

    private val logger = LoggerFactory.getLogger(UserAuthenticationAspect::class.java)

    @Before("@annotation(fi.kranu.servicea.aop.CustomAuthentication)")
    fun authenticate() {
        val request = (RequestContextHolder.getRequestAttributes() as? ServletRequestAttributes)?.request
        val userId = request?.getHeader("X-User-Id")

        if (userId.isNullOrBlank()) {
            logger.warn("Authentication failed: X-User-Id header is missing")
            throw UnauthorizedException()
        } else if (!userId.equals("dirty-hack")) {
            // This is just a dirty hack for testing purposes. Don't do this in production.
            logger.warn("Wrong secret")
            throw UnauthorizedException()
        }

        logger.info("User $userId authenticated successfully")
    }
}
```

### Example 1. Transaction Management

One of the most important and widely used applications of AOP.

It ensures that database operations are automatically wrapped in a transaction boundary.

Typical use cases:

* Atomic business operations
* Automatic rollback on failure
* Consistent data integrity

Example:

```java
@Transactional
public void transferFunds(Account from, Account to, BigDecimal amount) {
    debit(from, amount);
    credit(to, amount);
}
```

### Example 2. Performance Monitoring & Metrics

AOP is commonly used to measure execution time and feed observability systems.

Typical use cases:

* API latency tracking
* Slow query detection
* SLA/SLO monitoring

Example:

```java
@Timed("payment.processing.time")
public Payment processPayment(Request request) {
    ...
}
```

### Example 3. Retry Logic for Resilience

Essential in distributed systems where downstream services can fail temporarily.

Typical use cases:

* Network timeouts
* Transient database failures
* External API instability

Example:

```java
@Retryable(maxAttempts = 3, backoff = @Backoff(delay = 200))
public PaymentResponse callExternalService() {
    ...
}
```

### Example 4. Rate Limiting / Throttling

Used to prevent abuse and control system load.

Typical use cases:

* Public APIs
* Multi-tenant SaaS quotas
* DoS protection

Example:

```java
@RateLimited("user-api")
public UserProfile getProfile(String userId) {
    ...
}
```

### Example 5. Multi-Tenancy Enforcement

Ensures data isolation between tenants automatically.

Typical use cases:

* SaaS applications
* Tenant-specific data filtering
* Automatic query scoping

Example:

```java
@TenantScoped
public List<Order> findOrders() {
    ...
}
```

### Example 6. Audit Trail Generation

Critical for compliance and traceability.

Typical use cases:

* Financial systems
* Healthcare applications
* Regulatory reporting

Example:

```java
@Auditable(action = "UPDATE_CUSTOMER")
public void updateCustomer(Customer customer) {
    ...
}
```

### Example 7. Idempotency Enforcement

Prevents duplicate processing of the same request.

Typical use cases:

* Payment processing
* Order creation
* Event handling

Example:

```java
@Idempotent(key = "#request.transactionId")
public PaymentResult processPayment(PaymentRequest request) {
    ...
}
```

### Example 8. Caching

Used to avoid repeated expensive computations or database calls.

Typical use cases:

* Read-heavy endpoints
* Reference data lookup
* External API results

Example:

```java
@Cacheable("products")
public Product getProduct(String id) {
    ...
}
```

### Example 9. Distributed Tracing

Automatically creates trace spans for observability platforms.

Typical use cases:

* Microservice request tracking
* Performance bottleneck analysis
* Dependency mapping

Example:

```java
@NewSpan("order-processing")
public void processOrder(Order order) {
    ...
}
```

### Example 10. Method Validation Enforcement

Ensures input constraints before method execution.

Typical use cases:

* DTO validation
* Service-level input checks
* Contract enforcement

Example:

```java
@Validated
public void createUser(@NotNull @Email String email) {
    ...
}
```

### Example 11. Feature Flags / Conditional Execution

Allows dynamic behavior toggling without changing business logic.

Typical use cases:

* A/B testing
* Gradual rollout
* Experimental features

Example:

```java
@FeatureToggle("new-pricing-engine")
public Price calculatePrice(Order order) {
    ...
}
```

### Example 12. Automatic Event Publishing

Triggers domain events after method execution.

Typical use cases:

* Event-driven architecture
* CQRS systems
* Asynchronous workflows

Example:

```java
@PublishEvent("ORDER_CREATED")
public Order createOrder(CreateOrderRequest request) {
    ...
}
```

## Afterwords

When you are learning these, there is no need to test and see how you could do all of these. In my opinion you just need to practice couple of them and then the rest are easy enough to implement when needed. Learn ideas behind the system.
