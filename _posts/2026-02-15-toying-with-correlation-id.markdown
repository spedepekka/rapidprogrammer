---
layout: post
title: "Toying with correlation ID"
date: "2026-02-15 16:22:00 +0300"
permalink: /:title
---

This sums it up nicely: Make correlation ID a first-class citizen in your microservice design.

I stole it from this [Medium post](https://medium.com/@anil.goyal0057/understanding-and-implementing-correlation-id-in-microservices-2900518954a0).

If you missed my [previous post]({% post_url 2026-02-13-from-interview-to-learn-aop-and-openapi %}) how I got here, check it out.

## What is correlation ID

It is just a unique string that can be followed across a log file or multiple log files.

So put that damn thing everywhere and if you cannot find it from the request in the request chain, create it. Of course if you can find it from headers or other meta data, use the existing id.

## Why should I do it?

Because these days the stuff is all over the place and it is nearly impossible to track requests across microservices. There are tools like grep and even some fancy stuff with UIs that can cobble these "threads" and give you real information how the system is working.

## How I can do it?

Just generate the UUID, if not found. When the service contacts another service, pass that id to that other service. To make it work, it doesn't matter what is the id/string and how you pass it. To make it work elegantly, generate and pass it in more automated fashion.

## Kotlin and Spring Boot example

The code is in this [repo](https://github.com/spedepekka/soon-i-know-aop).

In this example the first service A creates the correlation ID with request filter

```java
@Component
class CorrelationIdFilter : OncePerRequestFilter() {
    private val logger = LoggerFactory.getLogger(CorrelationIdFilter::class.java)

    @Throws(ServletException::class, IOException::class)
    override fun doFilterInternal(
        request: HttpServletRequest,
        response: HttpServletResponse,
        filterChain: FilterChain
    ) {
        // NOTE: User can inject correlationId as a header
        var correlationId = request.getHeader(HEADER_NAME)

        if (correlationId == null) {
            correlationId = UUID.randomUUID().toString()
            logger.debug("Generated new correlation ID $correlationId")
        } else {
            logger.debug("Using existing correlation ID $correlationId")
        }

        MDC.put("correlationId", correlationId)
        response.setHeader(HEADER_NAME, correlationId)

        try {
            filterChain.doFilter(request, response)
        } finally {
            MDC.clear()
        }
    }

    companion object {
        private const val HEADER_NAME = "X-Correlation-Id"
    }
}
```

Then service A passes the id to service B

```java
@PostMapping
fun handleUserTransaction(@RequestBody transaction: Transaction): Transaction {
    val correlationId = MDC.get("correlationId")
    logger.info("handleUserTransaction called with correlationId: $correlationId, transaction: $transaction")

    val headers = HttpHeaders()
    if (correlationId != null) {
        headers.set("X-Correlation-Id", correlationId)
    }

    val request = HttpEntity(transaction, headers)
    try {
        restTemplate.postForObject("http://service-b-lol:8080/internal-transaction", request, Transaction::class.java)
    } catch (e: Exception) {
        logger.error("Failed to call service-b", e)
    }

    return transaction
}
```

And service B has similar filter than A, so the logs can be followed for a single transaction between services A and B.

The pattern would be the same for service C. Feel free to do this the way you like, but the idea should be clear now.

