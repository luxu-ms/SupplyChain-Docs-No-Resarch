# Supply Chain Architecture and Workflow

The frontend serves the procurement UI, the backend owns business rules and stores records in MySQL, and the worker consumes RabbitMQ messages for background notifications. All three use Java 8 and Spring Boot 2.7.

```text
Browser --> Frontend :8081 --REST--> Backend :8080 --> MySQL
                                        |
                                        +--> RabbitMQ --> Worker
```

## Component responsibilities

| Component | Responsibility |
| --- | --- |
| **SupplyChain-Frontend** | Spring MVC renders JSP pages for the dashboard, purchase orders, inventory, and vendors. It calls the backend over HTTP using `RestTemplate`; it does not access MySQL or RabbitMQ directly. See the [frontend API client](https://github.com/luxu-ms/SupplyChain-Frontend/blob/main/src/main/java/com/acme/scm/frontend/service/BackendApiService.java). |
| **SupplyChain-Backend** | Exposes `/api/orders`, `/api/inventory`, and `/api/vendors`. Controllers delegate to services, which apply business rules and use Spring Data JPA to access MySQL. The backend also publishes events to RabbitMQ. See the [backend architecture](https://github.com/luxu-ms/SupplyChain-Backend/blob/main/README.md). |
| **SupplyChain-Worker** | Listens for `order.created`, `inventory.alert`, and `approval.pending`. Its handlers simulate notifications with delays and log messages; they do not send real emails or change order state. See the [message listener](https://github.com/luxu-ms/SupplyChain-Worker/blob/main/src/main/java/com/acme/scm/worker/listener/OrderMessageListener.java). |

## Typical purchase-order workflow

1. A procurement user opens `/orders/new` and submits the form. The frontend sends `POST /api/orders` to the backend.
2. The backend checks that the amount is positive and the vendor is active, then saves the order in MySQL. New orders default to `PENDING` when no status is supplied. The backend attempts to publish `order.created`; if publishing throws an exception, it logs the failure without failing the order request. The frontend redirects to `/orders` on success.
3. The worker consumes `order.created` asynchronously and logs a simulated notification. It does not approve the order. Approval is a separate backend operation, `PUT /api/orders/{orderNumber}/approve`, which changes a `PENDING` order to `APPROVED` and records who approved it and when.

## What isn't wired end to end

The [pending-orders page](https://github.com/luxu-ms/SupplyChain-Frontend/blob/main/src/main/webapp/WEB-INF/jsp/orders/pending.jsp#L82) shows an **Approve** link, but the [frontend controller](https://github.com/luxu-ms/SupplyChain-Frontend/blob/main/src/main/java/com/acme/scm/frontend/controller/OrderController.java) has no matching route.

The order-creation path does not publish `approval.pending`, and viewing low stock only reads data; it does not publish `inventory.alert`.
