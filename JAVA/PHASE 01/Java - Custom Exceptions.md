---
title: "Java - Custom Exceptions"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - exceptions
  - custom-exceptions
  - error-handling
status: "not-started"
---

# Java - Custom Exceptions

> [!abstract] Overview
> Custom exceptions are user-defined exception classes that represent specific error conditions in your application's domain. While Java provides many built-in exceptions (`NullPointerException`, `IllegalArgumentException`, etc.), they are too generic for backend systems that need to distinguish between "order not found," "payment declined," "insufficient stock," and "rate limit exceeded." Custom exceptions carry domain-specific error codes, HTTP status codes, and contextual data that enable the global exception handler to produce precise, structured API responses. This note covers how to design, implement, and organize a production-grade exception hierarchy for a Spring Boot backend.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Inheritance - Single Multilevel Hierarchical]]
> - [[Java - Exception Handling - Try Catch Finally Throw Throws]]
> - [[Java - Encapsulation - Getters Setters Access Modifiers]]
> - [[Java - Final Keyword]]

---

## Theory

### Why Custom Exceptions?

Java's built-in exceptions are designed for general-purpose programming. They tell you **what went wrong at the language level** (null reference, bad index, illegal argument) but not **what went wrong at the business level** (order already shipped, coupon expired, account suspended). In a backend API, this distinction is critical because:

1. **HTTP status codes must be precise.** A `NullPointerException` and an "order not found" error are both technically exceptions, but the first should return 500 (server error) and the second should return 404 (not found). Built-in exceptions do not carry HTTP status information.

2. **Error responses must be structured.** Frontend applications and API consumers expect consistent error responses with error codes, messages, and sometimes field-level details. A raw `IllegalArgumentException` message like "null" is useless to a frontend developer trying to display a meaningful error to the user.

3. **Logging and monitoring need categorization.** When your production system processes millions of requests, you need to distinguish between "payment gateway timeout" (infrastructure issue, alert the ops team) and "invalid coupon code" (normal business condition, no alert needed). Custom exception types make this categorization automatic.

4. **Exception handling should be centralized.** With custom exceptions, your service layer throws domain-specific exceptions, and a single global handler translates them into HTTP responses. Without custom exceptions, every controller method would need its own `try-catch` logic to map generic exceptions to appropriate status codes.

### Designing a Custom Exception Hierarchy

A well-designed exception hierarchy follows these principles:

**1. Single base class for all application exceptions.**

Every custom exception in your application should extend a single base class. This allows the global exception handler to catch all application-specific errors with a single `@ExceptionHandler` method while still distinguishing them from framework errors and unexpected bugs.

```text
RuntimeException
    └── AppException (your base class)
            ├── NotFoundException
            ├── ValidationException
            ├── BusinessRuleException
            ├── AuthenticationException
            ├── AuthorizationException
            ├── RateLimitException
            └── IntegrationException
                    ├── PaymentException
                    ├── EmailException
                    └── SmsException
```

**2. Checked vs unchecked: prefer unchecked for application errors.**

In modern Spring Boot development, all custom exceptions should extend `RuntimeException` (unchecked). The reasons:

- Spring's transaction management rolls back on unchecked exceptions by default. If your `OrderService.createOrder()` throws a checked exception, the transaction might commit even though the order creation failed.
- Checked exceptions force every caller to write `try-catch` or `throws`, which clutters the code and provides little value when the caller cannot meaningfully handle the error.
- Spring wraps most checked exceptions from JDBC, JPA, and HTTP clients in unchecked wrappers anyway. Fighting this pattern creates inconsistency.

**3. Include error codes and HTTP status in the base class.**

Every exception should carry a machine-readable error code (for frontend logic) and an HTTP status code (for the response). These belong in the base class so that the global handler can access them uniformly.

**4. Support exception chaining.**

Always provide a constructor that accepts a `Throwable cause`. When a low-level error (database timeout, network failure) causes a high-level error (order creation failed), the cause chain preserves the full diagnostic trail.

**5. Include contextual data in specific exceptions.**

Domain-specific exceptions should carry the data that the error handler or the client needs. An `InsufficientStockException` should include the product name, requested quantity, and available quantity. An `OrderNotFoundException` should include the order ID. This context makes error messages precise and debugging easier.

### Exception Naming Conventions

| Pattern | Example | Use Case |
|---------|---------|----------|
| `[Entity]NotFoundException` | `OrderNotFoundException`, `UserNotFoundException` | Resource does not exist |
| `[Entity]AlreadyExistsException` | `EmailAlreadyExistsException` | Duplicate resource |
| `[Action]NotAllowedException` | `CancellationNotAllowedException` | Invalid state transition |
| `Insufficient[Resource]Exception` | `InsufficientStockException`, `InsufficientFundsException` | Not enough of something |
| `[Feature]Exception` | `PaymentException`, `ShippingException` | Domain-specific failures |
| `Invalid[Entity]Exception` | `InvalidOrderException`, `InvalidTokenException` | Malformed or invalid data |

### When NOT to Create Custom Exceptions

Not every error condition needs a custom exception class. Avoid creating custom exceptions when:

1. **A built-in exception is sufficient.** `IllegalArgumentException` is fine for simple input validation. `IllegalStateException` is fine for simple state checks. Do not create `NegativeQuantityException` when `IllegalArgumentException("Quantity must be positive")` conveys the same information.

2. **The exception is used in only one place.** If an error condition occurs in a single method and is caught in the same method, a custom exception class adds unnecessary complexity. Use a built-in exception or handle the condition with an `if-else`.

3. **The exception does not carry additional context.** If your custom exception has the same fields as `RuntimeException` (just a message), it adds no value. Custom exceptions should carry domain-specific data that the handler or client needs.

### Exception Factories

For complex exception creation logic, use static factory methods instead of constructors. Factory methods can have descriptive names, return cached instances for common errors, and encapsulate complex message formatting.

```java
import java.time.LocalDateTime;

public class OrderException extends AppException {

    private OrderException(String message, int httpStatus, String errorCode) {
        super(message, httpStatus, errorCode);
    }

    // Factory methods with descriptive names
    public static OrderException notFound(Long orderId) {
        return new OrderException(
            "Order not found: " + orderId, 404, "ORDER_NOT_FOUND"
        );
    }

    public static OrderException alreadyCancelled(Long orderId) {
        return new OrderException(
            "Order " + orderId + " is already cancelled", 409, "ORDER_ALREADY_CANCELLED"
        );
    }

    public static OrderException expired(Long orderId, LocalDateTime expiredAt) {
        return new OrderException(
            "Order " + orderId + " expired at " + expiredAt, 410, "ORDER_EXPIRED"
        );
    }
}
```

### How Custom Exceptions Work Internally

When you create a custom exception, you are creating a regular Java class that extends `RuntimeException` (or another exception class). The JVM treats it exactly like any other exception:

1. **Object creation**: `new OrderNotFoundException(42L)` allocates memory on the heap for the exception object. The `Throwable` constructor calls `fillInStackTrace()`, which walks the call stack and records every frame. This is the most expensive part of exception creation.

2. **Throwing**: `throw new OrderNotFoundException(42L)` interrupts the normal execution flow. The JVM begins stack unwinding, searching for a matching `catch` block.

3. **Matching**: The JVM uses `instanceof` semantics to match the thrown exception to `catch` blocks. If you throw `OrderNotFoundException` and the catch block is `catch (AppException e)`, it matches because `OrderNotFoundException instanceof AppException` is `true`.

4. **Polymorphic handling**: The global exception handler uses the exception's type to determine the HTTP status code and error response format. This is runtime polymorphism applied to error handling.

> [!tip] Key Insight
> The most important design decision for your exception hierarchy is the **granularity of the base class**. Too coarse (a single `AppException` for everything) and the global handler cannot distinguish between a 404 and a 403. Too fine (a separate exception class for every possible error) and you end up with hundreds of tiny classes that add no value. The sweet spot is one exception class per **error category** (not found, validation, business rule, authentication, authorization, rate limit, integration) with contextual data fields to distinguish specific cases within each category.

---

## Syntax and Basic Examples

### Example 1: Complete exception hierarchy for an e-commerce backend

```java
// ===== BASE EXCEPTION =====

public class AppException extends RuntimeException {
    private final int httpStatus;
    private final String errorCode;

    public AppException(String message, int httpStatus, String errorCode) {
        super(message);
        this.httpStatus = httpStatus;
        this.errorCode = errorCode;
    }

    public AppException(String message, int httpStatus, String errorCode, Throwable cause) {
        super(message, cause);
        this.httpStatus = httpStatus;
        this.errorCode = errorCode;
    }

    public int getHttpStatus() { return httpStatus; }
    public String getErrorCode() { return errorCode; }
}
```

```java
// ===== NOT FOUND EXCEPTIONS =====

public class ResourceNotFoundException extends AppException {
    private final String resourceName;
    private final Object identifier;

    public ResourceNotFoundException(String resourceName, Object identifier) {
        super(
            resourceName + " not found with identifier: " + identifier,
            404,
            resourceName.toUpperCase() + "_NOT_FOUND"
        );
        this.resourceName = resourceName;
        this.identifier = identifier;
    }

    public String getResourceName() { return resourceName; }
    public Object getIdentifier() { return identifier; }
}
```

```java
// ===== VALIDATION EXCEPTIONS =====

import java.util.Collections;
import java.util.Map;

public class ValidationException extends AppException {
    private final Map<String, String> fieldErrors;

    public ValidationException(String message, Map<String, String> fieldErrors) {
        super(message, 400, "VALIDATION_ERROR");
        this.fieldErrors = fieldErrors != null
            ? Collections.unmodifiableMap(fieldErrors)
            : Map.of();
    }

    public ValidationException(String field, String reason) {
        super("Validation failed: " + field + " - " + reason, 400, "VALIDATION_ERROR");
        this.fieldErrors = Map.of(field, reason);
    }

    public Map<String, String> getFieldErrors() { return fieldErrors; }
}
```

```java
// ===== BUSINESS RULE EXCEPTIONS =====

public class BusinessRuleException extends AppException {
    public BusinessRuleException(String message, String errorCode) {
        super(message, 422, errorCode);
    }
}

public class InsufficientStockException extends BusinessRuleException {
    private final String productName;
    private final int requested;
    private final int available;

    public InsufficientStockException(String productName, int requested, int available) {
        super(
            "Insufficient stock for '" + productName
                + "': requested " + requested + ", available " + available,
            "INSUFFICIENT_STOCK"
        );
        this.productName = productName;
        this.requested = requested;
        this.available = available;
    }

    public String getProductName() { return productName; }
    public int getRequested() { return requested; }
    public int getAvailable() { return available; }
}

public class OrderStateException extends BusinessRuleException {
    private final Long orderId;
    private final String currentStatus;
    private final String attemptedAction;

    public OrderStateException(Long orderId, String currentStatus, String attemptedAction) {
        super(
            "Cannot " + attemptedAction + " order " + orderId
                + " in status " + currentStatus,
            "INVALID_ORDER_STATE"
        );
        this.orderId = orderId;
        this.currentStatus = currentStatus;
        this.attemptedAction = attemptedAction;
    }

    public Long getOrderId() { return orderId; }
    public String getCurrentStatus() { return currentStatus; }
    public String getAttemptedAction() { return attemptedAction; }
}
```

```java
import java.time.LocalDateTime;

public class CouponExpiredException extends BusinessRuleException {
    private final String couponCode;
    private final LocalDateTime expiredAt;

    public CouponExpiredException(String couponCode, LocalDateTime expiredAt) {
        super(
            "Coupon '" + couponCode + "' expired at " + expiredAt,
            "COUPON_EXPIRED"
        );
        this.couponCode = couponCode;
        this.expiredAt = expiredAt;
    }

    public String getCouponCode() { return couponCode; }
    public LocalDateTime getExpiredAt() { return expiredAt; }
}
```

```java
// ===== AUTHENTICATION AND AUTHORIZATION EXCEPTIONS =====

public class AuthenticationException extends AppException {
    public AuthenticationException(String message) {
        super(message, 401, "AUTHENTICATION_FAILED");
    }
}

public class AuthorizationException extends AppException {
    private final String requiredRole;

    public AuthorizationException(String message, String requiredRole) {
        super(message, 403, "AUTHORIZATION_FAILED");
        this.requiredRole = requiredRole;
    }

    public String getRequiredRole() { return requiredRole; }
}
```

```java
// ===== INTEGRATION EXCEPTIONS =====

public class IntegrationException extends AppException {
    private final String serviceName;

    public IntegrationException(String message, String serviceName, Throwable cause) {
        super(message, 502, "INTEGRATION_ERROR_" + serviceName.toUpperCase(), cause);
        this.serviceName = serviceName;
    }

    public String getServiceName() { return serviceName; }
}

public class PaymentGatewayException extends IntegrationException {
    private final String transactionId;

    public PaymentGatewayException(String message, String transactionId, Throwable cause) {
        super(message, "PAYMENT_GATEWAY", cause);
        this.transactionId = transactionId;
    }

    public String getTransactionId() { return transactionId; }
}
```

### Example 2: Using the exception hierarchy in a service

```java
import java.time.LocalDateTime;

@Service
public class OrderService {

    private final OrderRepository orderRepository;
    private final ProductRepository productRepository;
    private final CouponRepository couponRepository;
    private final PaymentGatewayClient paymentClient;

    public OrderService(OrderRepository orderRepository,
                        ProductRepository productRepository,
                        CouponRepository couponRepository,
                        PaymentGatewayClient paymentClient) {
        this.orderRepository = orderRepository;
        this.productRepository = productRepository;
        this.couponRepository = couponRepository;
        this.paymentClient = paymentClient;
    }

    public Order createOrder(CreateOrderRequest request) {
        // 1. Validate stock (throws InsufficientStockException)
        for (OrderItemRequest item : request.getItems()) {
            Product product = productRepository.findById(item.getProductId())
                .orElseThrow(() -> new ResourceNotFoundException("Product", item.getProductId()));

            if (product.getStock() < item.getQuantity()) {
                throw new InsufficientStockException(
                    product.getName(), item.getQuantity(), product.getStock()
                );
            }
        }

        // 2. Validate coupon (throws CouponExpiredException)
        if (request.getCouponCode() != null) {
            Coupon coupon = couponRepository.findByCode(request.getCouponCode())
                .orElseThrow(() -> new ResourceNotFoundException("Coupon", request.getCouponCode()));

            if (coupon.getExpiresAt().isBefore(LocalDateTime.now())) {
                throw new CouponExpiredException(coupon.getCode(), coupon.getExpiresAt());
            }
        }

        // 3. Create and save the order
        Order order = buildOrder(request);
        Order savedOrder = orderRepository.save(order);

        // 4. Process payment (throws PaymentGatewayException)
        try {
            paymentClient.charge(savedOrder.getTotalAmount(), savedOrder.getUserId());
        } catch (PaymentGatewayException e) {
            // Rollback: cancel the order
            savedOrder.cancel();
            orderRepository.save(savedOrder);
            throw e;  // Rethrow with original cause chain intact
        }

        return savedOrder;
    }

    public Order cancelOrder(Long orderId) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new ResourceNotFoundException("Order", orderId));

        if (order.getStatus() == OrderStatus.SHIPPED) {
            throw new OrderStateException(orderId, "SHIPPED", "cancel");
        }
        if (order.getStatus() == OrderStatus.CANCELLED) {
            throw new OrderStateException(orderId, "CANCELLED", "cancel");
        }

        order.cancel();
        return orderRepository.save(order);
    }
}
```

### Example 3: Global exception handler that uses the hierarchy

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import java.util.Map;

@RestControllerAdvice
public class GlobalExceptionHandler {

    private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    // Most specific handlers first

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(404).body(new ErrorResponse(
            ex.getErrorCode(), ex.getMessage(), null
        ));
    }

    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(ValidationException ex) {
        return ResponseEntity.status(400).body(new ErrorResponse(
            ex.getErrorCode(), ex.getMessage(), ex.getFieldErrors()
        ));
    }

    @ExceptionHandler(InsufficientStockException.class)
    public ResponseEntity<ErrorResponse> handleInsufficientStock(InsufficientStockException ex) {
        // Include extra context in the error response
        Map<String, String> details = Map.of(
            "product", ex.getProductName(),
            "requested", String.valueOf(ex.getRequested()),
            "available", String.valueOf(ex.getAvailable())
        );
        return ResponseEntity.status(422).body(new ErrorResponse(
            ex.getErrorCode(), ex.getMessage(), details
        ));
    }

    @ExceptionHandler(BusinessRuleException.class)
    public ResponseEntity<ErrorResponse> handleBusinessRule(BusinessRuleException ex) {
        return ResponseEntity.status(422).body(new ErrorResponse(
            ex.getErrorCode(), ex.getMessage(), null
        ));
    }

    @ExceptionHandler(AuthenticationException.class)
    public ResponseEntity<ErrorResponse> handleAuth(AuthenticationException ex) {
        return ResponseEntity.status(401).body(new ErrorResponse(
            ex.getErrorCode(), ex.getMessage(), null
        ));
    }

    @ExceptionHandler(AuthorizationException.class)
    public ResponseEntity<ErrorResponse> handleAuthorization(AuthorizationException ex) {
        return ResponseEntity.status(403).body(new ErrorResponse(
            ex.getErrorCode(), ex.getMessage(),
            Map.of("requiredRole", ex.getRequiredRole())
        ));
    }

    @ExceptionHandler(IntegrationException.class)
    public ResponseEntity<ErrorResponse> handleIntegration(IntegrationException ex) {
        // Log the full exception chain for the ops team
        logger.error("Integration failure with {}: {}",
            ex.getServiceName(), ex.getMessage(), ex.getCause());

        // Do NOT expose internal service details to the client
        return ResponseEntity.status(502).body(new ErrorResponse(
            ex.getErrorCode(),
            "An external service is temporarily unavailable. Please try again later.",
            null
        ));
    }

    @ExceptionHandler(AppException.class)
    public ResponseEntity<ErrorResponse> handleAppException(AppException ex) {
        return ResponseEntity.status(ex.getHttpStatus()).body(new ErrorResponse(
            ex.getErrorCode(), ex.getMessage(), null
        ));
    }

    // Catch-all for unexpected errors
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleUnexpected(Exception ex) {
        logger.error("Unhandled exception", ex);
        return ResponseEntity.status(500).body(new ErrorResponse(
            "INTERNAL_ERROR",
            "An unexpected error occurred. Please contact support.",
            null
        ));
    }
}
```

```java
import java.util.Map;

// The structured error response DTO
public record ErrorResponse(
    String error,
    String message,
    Map<String, String> details
) {}
```

### Example 4: Exception factory pattern

```java
public final class OrderExceptions {
    // Private constructor: utility class
    private OrderExceptions() {}

    public static AppException notFound(Long orderId) {
        return new ResourceNotFoundException("Order", orderId);
    }

    public static AppException notFoundByNumber(String orderNumber) {
        return new ResourceNotFoundException("Order", orderNumber);
    }

    public static BusinessRuleException cannotCancel(Long orderId, OrderStatus status) {
        return new OrderStateException(orderId, status.name(), "cancel");
    }

    public static BusinessRuleException cannotModify(Long orderId, OrderStatus status) {
        return new OrderStateException(orderId, status.name(), "modify");
    }

    public static BusinessRuleException expired(Long orderId) {
        return new BusinessRuleException(
            "Order " + orderId + " has expired and can no longer be processed",
            "ORDER_EXPIRED"
        );
    }

    public static BusinessRuleException maxItemsExceeded(int maxItems) {
        return new BusinessRuleException(
            "Order cannot contain more than " + maxItems + " items",
            "MAX_ITEMS_EXCEEDED"
        );
    }
}
```

```java
// Usage in the service: clean and readable
public Order cancelOrder(Long orderId) {
    Order order = orderRepository.findById(orderId)
        .orElseThrow(() -> OrderExceptions.notFound(orderId));

    if (!order.canBeCancelled()) {
        throw OrderExceptions.cannotCancel(orderId, order.getStatus());
    }

    order.cancel();
    return orderRepository.save(order);
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Here are three realistic scenarios showing how custom exceptions are designed and used in production Spring Boot systems.

### Scenario 1: Domain-driven exception design for a payment module

In a well-architected backend, each bounded context (module) defines its own exception types that extend the application's base exception.

```java
package com.company.orderservice.payment.exception;

// Base exception for the payment module
public class PaymentException extends AppException {
    private final String transactionId;

    public PaymentException(String message, String errorCode, String transactionId) {
        super(message, 402, errorCode);
        this.transactionId = transactionId;
    }

    public PaymentException(String message, String errorCode, String transactionId, Throwable cause) {
        super(message, 402, errorCode, cause);
        this.transactionId = transactionId;
    }

    public String getTransactionId() { return transactionId; }
}
```

```java
package com.company.orderservice.payment.exception;

public class PaymentDeclinedException extends PaymentException {
    private final String declineReason;

    public PaymentDeclinedException(String transactionId, String declineReason) {
        super(
            "Payment declined: " + declineReason,
            "PAYMENT_DECLINED",
            transactionId
        );
        this.declineReason = declineReason;
    }

    public String getDeclineReason() { return declineReason; }
}
```

```java
package com.company.orderservice.payment.exception;

public class PaymentTimeoutException extends PaymentException {
    public PaymentTimeoutException(String transactionId, Throwable cause) {
        super(
            "Payment gateway did not respond within the timeout period",
            "PAYMENT_TIMEOUT",
            transactionId,
            cause  // Preserve the original timeout exception
        );
    }
}
```

```java
package com.company.orderservice.payment.exception;

public class DuplicatePaymentException extends PaymentException {
    public DuplicatePaymentException(String transactionId, String originalTransactionId) {
        super(
            "Duplicate payment detected. Original transaction: " + originalTransactionId,
            "DUPLICATE_PAYMENT",
            transactionId
        );
    }
}
```

```java
// Usage in the payment service:
@Service
public class PaymentService {

    public PaymentResult charge(Order order) {
        String txnId = generateTransactionId();

        try {
            StripeResponse response = stripeClient.charge(
                order.getTotalAmount(), order.getUserId()
            );

            if ("declined".equals(response.getStatus())) {
                throw new PaymentDeclinedException(txnId, response.getDeclineCode());
            }

            return new PaymentResult(txnId, response.getStatus());

        } catch (StripeTimeoutException e) {
            throw new PaymentTimeoutException(txnId, e);
        } catch (StripeDuplicateException e) {
            throw new DuplicatePaymentException(txnId, e.getOriginalTransactionId());
        } catch (StripeException e) {
            throw new PaymentException(
                "Unexpected payment error: " + e.getMessage(),
                "PAYMENT_ERROR",
                txnId,
                e
            );
        }
    }
}
```

**What to notice:**

- The payment module has its own exception hierarchy rooted at `PaymentException`, which extends the application's `AppException`. This allows the global handler to catch all payment errors with `@ExceptionHandler(PaymentException.class)` while still distinguishing specific cases.
- Each exception carries domain-specific context: `PaymentDeclinedException` has the decline reason, `PaymentTimeoutException` preserves the original timeout cause, `DuplicatePaymentException` references the original transaction.
- The `PaymentService` catches Stripe-specific exceptions and wraps them in application-specific exceptions. This decouples the rest of the application from the Stripe SDK. If you switch to a different payment provider, only this service changes.

### Scenario 2: Validation exception with field-level errors

Spring Boot's `@Valid` annotation triggers Jakarta Bean Validation, which throws `MethodArgumentNotValidException`. You can convert this into your custom `ValidationException` for consistency.

```java
import org.springframework.http.ResponseEntity;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import java.util.HashMap;
import java.util.Map;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(
            MethodArgumentNotValidException ex) {

        // Extract field-level errors from Spring's validation result
        Map<String, String> fieldErrors = new HashMap<>();
        for (FieldError error : ex.getBindingResult().getFieldErrors()) {
            fieldErrors.put(error.getField(), error.getDefaultMessage());
        }

        // Convert to our custom ValidationException format
        ValidationException validationException = new ValidationException(
            "Request validation failed", fieldErrors
        );

        return ResponseEntity.status(400).body(new ErrorResponse(
            validationException.getErrorCode(),
            validationException.getMessage(),
            validationException.getFieldErrors()
        ));
    }
}
```

```java
import jakarta.validation.Valid;
import jakarta.validation.constraints.NotEmpty;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Pattern;
import jakarta.validation.constraints.Positive;
import jakarta.validation.constraints.Size;
import java.util.List;

// The DTO with validation annotations:
public record CreateOrderRequest(
    @NotNull(message = "User ID is required")
    @Positive(message = "User ID must be positive")
    Long userId,

    @NotEmpty(message = "Order must contain at least one item")
    @Size(max = 100, message = "Maximum 100 items per order")
    List<@Valid OrderItemRequest> items,

    @Pattern(regexp = "^[A-Z0-9]{5,10}$", message = "Coupon code must be 5-10 uppercase alphanumeric characters")
    String couponCode
) {}

// When the client sends invalid data:
// {
//   "userId": -1,
//   "items": [],
//   "couponCode": "invalid!"
// }
//
// The API returns:
// {
//   "error": "VALIDATION_ERROR",
//   "message": "Request validation failed",
//   "details": {
//     "userId": "User ID must be positive",
//     "items": "Order must contain at least one item",
//     "couponCode": "Coupon code must be 5-10 uppercase alphanumeric characters"
//   }
// }
```

**What to notice:**

- The validation errors are returned as a structured map of field names to error messages. This allows the frontend to display specific error messages next to each form field.
- The `@Valid` annotation on the `List<@Valid OrderItemRequest>` parameter triggers nested validation. If an item has an invalid quantity, the error will include the path `items[0].quantity`.
- The custom `ValidationException` wraps Spring's validation result in the application's standard error format, ensuring consistency across all error responses.

### Scenario 3: Exception handling in async and scheduled tasks

In async methods and scheduled tasks, exceptions do not propagate to the controller. You need explicit exception handling to prevent silent failures.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Async;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class NotificationService {

    private static final Logger logger = LoggerFactory.getLogger(NotificationService.class);

    // Async methods run on a separate thread. Exceptions do not reach the controller.
    // You must handle them explicitly or configure an AsyncUncaughtExceptionHandler.
    @Async
    public void sendOrderConfirmation(Order order) {
        try {
            emailClient.send(
                order.getUserEmail(),
                "Order Confirmed: " + order.getOrderNumber(),
                buildConfirmationTemplate(order)
            );
        } catch (EmailException e) {
            // Log the error and schedule a retry.
            // Do NOT rethrow: there is no caller to catch it.
            logger.error("Failed to send confirmation for order {}: {}",
                order.getOrderNumber(), e.getMessage(), e);
            retryQueue.enqueue(new RetryTask("EMAIL_CONFIRMATION", order.getId()));
        } catch (Exception e) {
            // Catch-all for unexpected errors in async context.
            // Without this, the exception is silently swallowed by the thread pool.
            logger.error("Unexpected error sending notification for order {}",
                order.getOrderNumber(), e);
        }
    }

    // Scheduled tasks also need explicit exception handling.
    @Scheduled(cron = "0 0 * * * *")  // Every hour
    public void processExpiredOrders() {
        try {
            List<Order> expiredOrders = orderRepository.findExpiredPendingOrders();
            for (Order order : expiredOrders) {
                try {
                    order.cancel();
                    orderRepository.save(order);
                    logger.info("Cancelled expired order: {}", order.getOrderNumber());
                } catch (Exception e) {
                    // Log and continue: one failed cancellation should not stop the batch
                    logger.error("Failed to cancel expired order {}: {}",
                        order.getOrderNumber(), e.getMessage(), e);
                }
            }
        } catch (Exception e) {
            logger.error("Failed to process expired orders batch", e);
        }
    }
}
```

**What to notice:**

- In `@Async` methods, exceptions do not propagate to the HTTP response. The caller (the controller) has already returned a response by the time the async method runs. Unhandled exceptions in async methods are logged by the thread pool's uncaught exception handler and then silently discarded.
- In `@Scheduled` methods, an unhandled exception terminates the current execution but does not stop future scheduled runs. However, the error is only visible in the logs, not in any API response.
- The nested `try-catch` in `processExpiredOrders()` ensures that one failed order cancellation does not prevent the rest of the batch from being processed. This is the same pattern used in the batch processing example from the Loops note.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Creating too many exception classes

**Wrong:**

```java
public class OrderNotFoundException extends AppException { ... }
public class OrderNotFoundByIdException extends AppException { ... }
public class OrderNotFoundByNumberException extends AppException { ... }
public class OrderNotFoundByEmailException extends AppException { ... }
public class UserNotFoundException extends AppException { ... }
public class UserNotFoundByIdException extends AppException { ... }
public class UserNotFoundByEmailException extends AppException { ... }
// ... 50 more classes for every entity and every lookup field
```

**Right:**

```java
public class ResourceNotFoundException extends AppException {
    private final String resourceName;
    private final Object identifier;

    public ResourceNotFoundException(String resourceName, Object identifier) {
        super(resourceName + " not found: " + identifier, 404,
            resourceName.toUpperCase() + "_NOT_FOUND");
        this.resourceName = resourceName;
        this.identifier = identifier;
    }
}

// Usage:
throw new ResourceNotFoundException("Order", orderId);
throw new ResourceNotFoundException("Order", orderNumber);
throw new ResourceNotFoundException("User", email);
// One class handles all "not found" scenarios.
```

**Why it is wrong:** Each exception class adds a file to your codebase, a case to your global handler, and cognitive overhead for developers. If two exceptions are handled identically (same HTTP status, same response format), they should be the same class with different data. Use contextual fields (resource name, identifier) to distinguish cases, not separate classes.

### Mistake 2: Extending `Exception` instead of `RuntimeException`

**Wrong:**

```java
public class OrderNotFoundException extends Exception {  // Checked exception
    public OrderNotFoundException(Long id) {
        super("Order not found: " + id);
    }
}

// Now every method that might throw this must declare it:
public Order getOrder(Long id) throws OrderNotFoundException { ... }
public OrderResponse getOrderResponse(Long id) throws OrderNotFoundException { ... }
public ResponseEntity<OrderResponse> handleGetOrder(Long id) throws OrderNotFoundException { ... }
// The throws clause propagates through every layer of the application.
```

**Right:**

```java
public class OrderNotFoundException extends AppException {  // Unchecked
    public OrderNotFoundException(Long id) {
        super("Order not found: " + id, 404, "ORDER_NOT_FOUND");
    }
}

// No throws clause needed. The exception propagates to the global handler automatically.
public Order getOrder(Long id) { ... }
public OrderResponse getOrderResponse(Long id) { ... }
public ResponseEntity<OrderResponse> handleGetOrder(Long id) { ... }
```

**Why it is wrong:** Checked exceptions force every method in the call chain to declare `throws`, which clutters the code and couples every layer to the exception type. In Spring Boot, the global exception handler catches all exceptions regardless of whether they are checked or unchecked, so the compiler enforcement provides no benefit. Use unchecked exceptions for all application errors.

### Mistake 3: Exposing internal details in exception messages

**Wrong:**

```java
public Order getOrder(Long id) {
    try {
        return jdbcTemplate.queryForObject(
            "SELECT * FROM orders WHERE id = ?", new OrderRowMapper(), id
        );
    } catch (DataAccessException e) {
        // Exposing the SQL query and database error to the client!
        throw new AppException(
            "SQL error: " + e.getMessage() + " | Query: SELECT * FROM orders WHERE id = " + id,
            500, "DB_ERROR"
        );
    }
}
```

**Right:**

```java
public Order getOrder(Long id) {
    try {
        return jdbcTemplate.queryForObject(
            "SELECT * FROM orders WHERE id = ?", new OrderRowMapper(), id
        );
    } catch (EmptyResultDataAccessException e) {
        throw new ResourceNotFoundException("Order", id);  // Clean message for the client
    } catch (DataAccessException e) {
        // Log the full details for the development team
        logger.error("Database error fetching order {}", id, e);
        // Return a generic message to the client
        throw new AppException(
            "An error occurred while retrieving the order",
            500, "ORDER_RETRIEVAL_ERROR"
        );
    }
}
```

**Why it is wrong:** Exception messages that reach the client (through the API response) should never contain SQL queries, stack traces, internal class names, file paths, or server configuration details. This information can be exploited by attackers for SQL injection, reconnaissance, and other attacks. Log the full details internally, but return clean, generic messages to the client.

### Mistake 4: Not preserving the exception cause chain

**Wrong:**

```java
try {
    paymentGateway.charge(amount);
} catch (IOException e) {
    // The original IOException (with its stack trace showing the network error)
    // is discarded. The new exception has no connection to the root cause.
    throw new PaymentException("Payment failed");
}
```

**Right:**

```java
try {
    paymentGateway.charge(amount);
} catch (IOException e) {
    // The original IOException is preserved as the cause.
    // The stack trace will show both exceptions.
    throw new PaymentException("Payment failed", e);
}
```

**Why it is wrong:** When debugging a production issue, the root cause is often buried several layers deep in the exception chain. If any layer discards the cause, the diagnostic trail is broken, and you may spend hours trying to figure out why a payment failed when the real issue was a DNS resolution timeout. Always pass the original exception as the `cause` parameter.

---

## Key Takeaways

> [!tip] Remember these points
> 1. **Custom exceptions** represent domain-specific error conditions that built-in Java exceptions cannot express. They carry error codes, HTTP status codes, and contextual data that enable precise API responses and effective logging.
> 2. Design a **hierarchy** rooted at a single `AppException` base class that extends `RuntimeException`. Create one subclass per error **category** (not found, validation, business rule, authentication, integration), not per specific error case. Use contextual fields to distinguish cases within a category.
> 3. All custom exceptions should be **unchecked** (extend `RuntimeException`). This keeps method signatures clean, works with Spring's transaction management, and aligns with the framework's exception handling model.
> 4. Always provide a constructor that accepts a `Throwable cause` for **exception chaining**. When wrapping low-level exceptions (database, network, external API), pass the original exception as the cause to preserve the full diagnostic trail.
> 5. **Never expose internal details** (SQL queries, stack traces, class names) in exception messages that reach the client. Log full details internally and return clean, generic messages in API responses. Use the global exception handler to enforce this separation consistently.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Design an Exception Hierarchy (Easy)

Design and implement a complete exception hierarchy for a university course registration system. The hierarchy should include:

1. `UniversityException extends RuntimeException` (base class with `errorCode` and `httpStatus`).
2. `CourseNotFoundException` (404).
3. `CourseFullException` (422, with `courseCode` and `maxCapacity` fields).
4. `PrerequisiteNotMetException` (422, with `courseCode` and `missingPrerequisite` fields).
5. `RegistrationDeadlinePassedException` (422, with `deadline` field).
6. `DuplicateRegistrationException` (409, with `studentId` and `courseCode` fields).

Write a `RegistrationService` class with a `registerStudent(String studentId, String courseCode)` method that throws the appropriate exceptions based on simulated conditions. Write a `main()` method that tests each exception.

> **Hint:** Use `if` statements to simulate different error conditions based on the input values. For example, if `courseCode` is "CS999", throw `CourseNotFoundException`.

### Exercise 2: Exception Factory and Global Handler (Medium)

Create an exception factory class `UserExceptions` with static factory methods for common user-related errors:

- `notFound(Long id)`
- `notFoundByEmail(String email)`
- `emailAlreadyExists(String email)`
- `invalidPassword(String reason)`
- `accountLocked(String username, int remainingMinutes)`
- `accountSuspended(String username, String reason)`

Then create a `GlobalExceptionHandler` class (simulated, without Spring) with a `handle(Exception e)` method that uses `instanceof` pattern matching (Java 16+) to determine the exception type and returns a formatted error response string.

In `main()`, throw each exception type and pass it to the handler. Print the formatted response for each.

> **Hint:** The handler should check from most specific to least specific. Use `instanceof` with pattern matching: `if (e instanceof AccountLockedException locked) { ... }`.

### Exercise 3: Exception Chaining Demonstration (Medium)

Write a program that demonstrates exception chaining across three layers:

1. **Repository layer**: A method `findOrderInDatabase(Long id)` that simulates a database error by throwing a `RuntimeException("Connection refused to localhost:5432")`.
2. **Service layer**: A method `getOrder(Long id)` that calls the repository and catches the database exception, wrapping it in a custom `OrderRetrievalException` with the original as the cause.
3. **Controller layer**: A method `handleGetOrder(Long id)` that calls the service and catches the `OrderRetrievalException`, wrapping it in a custom `ApiException` with HTTP status 503.

In `main()`, call the controller method and print the full exception chain using `e.getCause()` recursively. Show that the original database error message is preserved at the bottom of the chain.

> **Hint:** Use a `while` loop to walk the cause chain: `Throwable cause = e; while (cause != null) { print cause; cause = cause.getCause(); }`.

### Exercise 4: Production-Grade Error Response System (Hard, Optional)

Build a complete error response system for a REST API:

1. Create the exception hierarchy from Exercise 1.
2. Create an `ErrorResponse` record with fields: `timestamp` (LocalDateTime), `status` (int), `error` (String), `message` (String), `path` (String), `details` (Map<String, Object>).
3. Create a `GlobalExceptionHandler` that converts each exception type into an `ErrorResponse`. For `CourseFullException`, include `courseCode` and `maxCapacity` in the `details` map. For `PrerequisiteNotMetException`, include the missing prerequisite.
4. Create a `RegistrationController` that calls the service and uses the handler to produce responses.
5. In `main()`, simulate several registration requests (valid and invalid) and print the JSON-like error responses.

> **Hint:** Use `String.format()` or `StringBuilder` to produce JSON-like output without a JSON library. The goal is to demonstrate the exception-to-response pipeline, not to produce valid JSON.

<details>
<summary>Solution for Exercise 1</summary>

```java
class UniversityException extends RuntimeException {
    private final int httpStatus;
    private final String errorCode;

    UniversityException(String message, int httpStatus, String errorCode) {
        super(message);
        this.httpStatus = httpStatus;
        this.errorCode = errorCode;
    }

    public int getHttpStatus() { return httpStatus; }
    public String getErrorCode() { return errorCode; }
}

class CourseNotFoundException extends UniversityException {
    CourseNotFoundException(String courseCode) {
        super("Course not found: " + courseCode, 404, "COURSE_NOT_FOUND");
    }
}

class CourseFullException extends UniversityException {
    private final String courseCode;
    private final int maxCapacity;

    CourseFullException(String courseCode, int maxCapacity) {
        super("Course " + courseCode + " is full (capacity: " + maxCapacity + ")", 422, "COURSE_FULL");
        this.courseCode = courseCode;
        this.maxCapacity = maxCapacity;
    }

    public String getCourseCode() { return courseCode; }
    public int getMaxCapacity() { return maxCapacity; }
}

class PrerequisiteNotMetException extends UniversityException {
    private final String courseCode;
    private final String missingPrerequisite;

    PrerequisiteNotMetException(String courseCode, String missingPrerequisite) {
        super("Prerequisite " + missingPrerequisite + " not met for " + courseCode, 422, "PREREQUISITE_NOT_MET");
        this.courseCode = courseCode;
        this.missingPrerequisite = missingPrerequisite;
    }
}

class RegistrationDeadlinePassedException extends UniversityException {
    private final String deadline;

    RegistrationDeadlinePassedException(String deadline) {
        super("Registration deadline passed: " + deadline, 422, "DEADLINE_PASSED");
        this.deadline = deadline;
    }
}

class DuplicateRegistrationException extends UniversityException {
    DuplicateRegistrationException(String studentId, String courseCode) {
        super("Student " + studentId + " is already registered for " + courseCode, 409, "DUPLICATE_REGISTRATION");
    }
}

class RegistrationService {
    void registerStudent(String studentId, String courseCode) {
        if ("CS999".equals(courseCode)) throw new CourseNotFoundException(courseCode);
        if ("CS101".equals(courseCode)) throw new CourseFullException(courseCode, 60);
        if ("CS301".equals(courseCode)) throw new PrerequisiteNotMetException(courseCode, "CS201");
        if ("CS201".equals(courseCode)) throw new RegistrationDeadlinePassedException("2025-06-30");
        if ("S001".equals(studentId) && "CS102".equals(courseCode))
            throw new DuplicateRegistrationException(studentId, courseCode);
        System.out.println("Registered " + studentId + " for " + courseCode);
    }
}

public class Main {
    public static void main(String[] args) {
        RegistrationService service = new RegistrationService();
        String[][] tests = {
            {"S001", "CS100"}, {"S001", "CS999"}, {"S002", "CS101"},
            {"S003", "CS301"}, {"S004", "CS201"}, {"S001", "CS102"}
        };
        for (String[] t : tests) {
            try {
                service.registerStudent(t[0], t[1]);
            } catch (UniversityException e) {
                System.out.printf("[%d %s] %s%n", e.getHttpStatus(), e.getErrorCode(), e.getMessage());
            }
        }
    }
}
```

</details>

<details>
<summary>Solution for Exercise 3</summary>

```java
class OrderRetrievalException extends RuntimeException {
    OrderRetrievalException(String message, Throwable cause) {
        super(message, cause);
    }
}

class ApiException extends RuntimeException {
    private final int httpStatus;

    ApiException(String message, int httpStatus, Throwable cause) {
        super(message, cause);
        this.httpStatus = httpStatus;
    }

    public int getHttpStatus() { return httpStatus; }
}

class Repository {
    void findOrderInDatabase(Long id) {
        throw new RuntimeException("Connection refused to localhost:5432");
    }
}

class Service {
    private final Repository repo = new Repository();

    void getOrder(Long id) {
        try {
            repo.findOrderInDatabase(id);
        } catch (RuntimeException e) {
            throw new OrderRetrievalException("Failed to retrieve order " + id, e);
        }
    }
}

class Controller {
    private final Service service = new Service();

    void handleGetOrder(Long id) {
        try {
            service.getOrder(id);
        } catch (OrderRetrievalException e) {
            throw new ApiException("Service temporarily unavailable", 503, e);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        try {
            new Controller().handleGetOrder(42L);
        } catch (ApiException e) {
            System.out.println("=== Exception Chain ===");
            int depth = 0;
            Throwable cause = e;
            while (cause != null) {
                System.out.printf("  %s[%d] %s: %s%n",
                    "  ".repeat(depth), depth,
                    cause.getClass().getSimpleName(), cause.getMessage());
                cause = cause.getCause();
                depth++;
            }
        }
    }
}
```

**Output:**

```text
=== Exception Chain ===
  [0] ApiException: Service temporarily unavailable
    [1] OrderRetrievalException: Failed to retrieve order 42
      [2] RuntimeException: Connection refused to localhost:5432
```

</details>

---

## Related Notes

- [[Java - Exception Handling - Try Catch Finally Throw Throws]]
- [[Java - Collections Framework Overview]] (next note)
- [[Java - Enum and Enum Methods]]

---

## Resources

- [Oracle Java Tutorials: Creating Exception Classes](https://docs.oracle.com/javase/tutorial/essential/exceptions/creating.html) - Official documentation on defining custom exceptions.
- [Baeldung: Custom Exceptions in Java](https://www.baeldung.com/java-new-custom-exception) - Comprehensive guide with best practices and anti-patterns.
- [Baeldung: Exception Handling in Spring MVC](https://www.baeldung.com/exception-handling-for-rest-with-spring) - Detailed guide to `@ControllerAdvice`, `@ExceptionHandler`, and `ProblemDetail`. Essential reading for Phase 4.
- [Effective Java by Joshua Bloch - Item 70: Use Checked Exceptions for Recoverable Conditions and Runtime Exceptions for Programming Errors](https://www.oreilly.com/library/view/effective-java/9780134686097/) - The definitive guide on when to use checked vs unchecked exceptions.
- [Effective Java by Joshua Bloch - Item 75: Include Failure-Capture Information in Detail Messages](https://www.oreilly.com/library/view/effective-java/9780134686097/) - How to write exception messages that actually help with debugging.
- [RFC 7807: Problem Details for HTTP APIs](https://datatracker.ietf.org/doc/html/rfc7807) - The industry standard for structured error responses in REST APIs. Spring Boot 3 supports this natively via `ProblemDetail`.
