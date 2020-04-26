---
title: "Using a custom java.time date with Jackson"
date: 2018-11-26 16:06
classes: wide
categories: general
---

It look me quite a bit more research to figure out how to use a custom (or lightly custom) time format with Jackson. I
required a date format to be deserialised that did not have a
[colon](https://stackoverflow.com/questions/46487403/java-8-date-and-time-parse-iso-8601-string-without-colon-in-offset?noredirect=1&lq=1) in the offset. So a date string like
`2018-11-05T05:58:00+0000` i.e. `+0000` not `+00:00`.

In the end it is as simple as:

```java
JavaTimeModule javaTimeModule = new JavaTimeModule();
// Make a new formatter that's based on the ISO_LOCAL_DATE_TIME one
DateTimeFormatter formatter = new DateTimeFormatterBuilder()
    .append(DateTimeFormatter.ISO_LOCAL_DATE_TIME)
    .optionalStart()
    // And just add what we need
    .appendOffset("+HHMM", "+0000")
    .optionalEnd()
    .toFormatter();

LocalDateTimeDeserializer deserializer = new LocalDateTimeDeserializer(formatter);
javaTimeModule.addDeserializer(LocalDateTime.class, deserializer);

// Mapper can come from anywhere, manually created or as part of a framwork
ObjectMapper mapper =  ObjectMapper MAPPER = new ObjectMapper();
// Finally use the new time module
mapper.registerModule(javaTimeModule);
// Use mapper to read JSON with our lightly tweaked time format
```
