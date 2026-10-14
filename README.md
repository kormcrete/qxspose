# qxspose

**Expose your JPA model as a typed query API: filter, sort, page and project, over HTTP or in-process.**

Add one starter to a Spring Boot app and every `@Entity` becomes a resource you can query, with no controller,
repository method or mapping code to write.

## How it works

- **Model:** qxspose reads the JPA metamodel into a GraphQL-shaped schema of resources, attributes and
  relationships, cycles included.
- **Query:** one URL grammar for filters, sorts, paging and projections, with a JSON body for queries too long for a
  URL. Counts, facets and analytics use the same grammar.
- **Type:** a Maven plugin generates a typed Java client from the schema. The same generated code runs over HTTP and
  in-process. TypeScript and Kotlin clients are planned.
- **Project:** results come back shaped as your DTO. Its fields decide what gets fetched.
- **Guard:** once you declare an access policy, it denies everything it does not grant.

## Quick start

It is not on Maven Central yet, so build and install it first (JDK 8, Maven 3):

```sh
git clone https://github.com/kormcrete/qxspose.git && cd qxspose && mvn -B install
```

Add the server starter to a Spring Boot 2.7 app:

```xml
<dependency>
    <groupId>io.github.kormcrete</groupId>
    <artifactId>qxspose-server-spring-boot-starter</artifactId>
    <version>0.2-SNAPSHOT</version>
</dependency>
```

Put an entity in your `@SpringBootApplication` package:

```java
@Entity @Table(name = "orders")
public class Order {
    @Id Long id;
    String reference;
    BigDecimal total;
    @Enumerated(EnumType.STRING) OrderStatus status;
    LocalDate placedOn;
}
```

Start the app and query it. `Order` is served as `orders`:

```sh
curl -g 'http://localhost:8080/qxspose/orders/list?fields=id,reference,total&filter[status]=NEW&filter[total][gt]=500&sort=-placedOn&page=0,20'
curl -g 'http://localhost:8080/qxspose/orders/count?filter[status]=NEW'
```

For the typed client, add `qxspose-maven-plugin` to a module that depends on your entities. It generates `OrdersQ`
and `OrdersQRepository`:

```java
@Autowired OrdersQRepository orders;   // OrderRow: any DTO with matching fields

List<OrderRow> rows = orders.query()
        .filter(OrdersQ.status().eq(OrderStatus.NEW))
        .filter(OrdersQ.total().gt(new BigDecimal("500")))
        .sort(OrdersQ.placedOn().desc())
        .as(OrderRow.class);
```

## Modules

- **Core:** `qxspose-domain-annotations`, `qxspose-domain`, `qxspose-domain-serialization`: the schema model.
- **Client:** `qxspose-client-api` (the query model and client interfaces) and `qxspose-client-http` (an HTTP client
  without Spring).
- **Server:** `qxspose-request-handler`, `qxspose-executor-api`, `qxspose-jpa-executor`, `qxspose-bean-executor`
  (resources computed in Java), `qxspose-caching-executor` and `qxspose-access-policy`. None of them needs Spring.
- **Codegen:** `qxspose-codegen-engine` and `qxspose-maven-plugin`.
- **Spring Boot:** `qxspose-server-spring-boot-starter` and `qxspose-client-spring-boot-starter`, with their
  `-autoconfigure` modules.

## License

Apache License 2.0. See [LICENSE](LICENSE).
