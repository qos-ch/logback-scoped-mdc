# logback-scoped-mdc

ScopedValue-based MDC for [Logback Classic](https://logback.qos.ch/), designed for virtual threads and structured concurrency on Java 25+.

## Why

Logback's built-in `MDC` is `ThreadLocal`-based, so values placed in it are not inherited by virtual threads forked via `java.util.concurrent.StructuredTaskScope`. `ScopedMDC` is backed by `java.lang.ScopedValue`, which **is** inherited by forked tasks — making it a natural fit for virtual-thread and structured-concurrency code. A pattern converter (`%scopedContext` / `%Y`) lets existing logback pattern layouts read its values.

## Prerequisites

- JDK 25+
- Maven 3.9+

## Build & test

```sh
mvn clean verify
```

Tests run with `--enable-preview` (configured via the surefire `argLine` in `pom.xml`) because `StructuredTaskScope` is still not stable. Main sources compile as plain Java 25 — no preview flag needed at runtime since `ScopedValue` is stable.

## Usage

### Maven dependency

```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-scoped-mdc</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```

### Setting scoped context

```java
ScopedMDC.put("requestId", "abc-123")
         .put("userId", "user-42")
         .run(() -> {
             logger.info("Processing request");
         });
```

Also available:

- `ScopedMDC.putAll(Map<String, String>)` for bulk entries.
- `Binding.call(CallableOp)` for value-returning operations that may throw.

Nested scopes inherit (and may override) parent entries; parent entries are restored automatically when the nested scope exits. Values bound via `ScopedMDC` are automatically visible inside virtual threads forked from a `StructuredTaskScope`.

### Wiring the pattern converter

Register the converter in `logback.xml`:

```xml
<conversionRule conversionWord="scopedContext"
    converterClass="ch.qos.logback.classic.scoped.ScopedMDCConverter" />
<conversionRule conversionWord="Y"
    converterClass="ch.qos.logback.classic.scoped.ScopedMDCConverter" />
```

Then use it in a pattern:

- `%scopedContext` — all entries, formatted as `key1=value1, key2=value2`.
- `%scopedContext{requestId}` — value for a specific key, or empty string if absent.
- `%scopedContext{requestId:-unknown}` — value for a specific key with default.

## License

Dual-licensed under EPL-2.0 or LGPL-2.1, matching Logback. See [`LICENSE.txt`](LICENSE.txt).
