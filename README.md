GELF Client
===========

[![Maven Central](https://img.shields.io/maven-central/v/org.graylog2/gelfclient.svg)](https://mvnrepository.com/artifact/org.graylog2/gelfclient)
[![Build Status](https://travis-ci.org/Graylog2/gelfclient.svg)](https://travis-ci.org/Graylog2/gelfclient)
[![Coverage Status](https://img.shields.io/coveralls/Graylog2/gelfclient.svg)](https://coveralls.io/r/Graylog2/gelfclient)

A Java GELF client library with support for different transports.

Available transports:

* TCP
* UDP

All default transport implementations use a queue to send messages in a
background thread to avoid blocking the calling thread until a message has
been sent. That means that the `send()` and `trySend()` methods do not
actually send the messages but add them to a queue where the background
thread will pick them up. This is important to keep in mind when it comes to
message delivery guarantees.

The library uses [Netty v4](http://netty.io/) to handle all network related
tasks and [Jackson](https://github.com/FasterXML/jackson) for JSON encoding.

## Usage

### Maven Dependency

```xml
<dependency>
  <groupId>org.graylog2</groupId>
  <artifactId>gelfclient</artifactId>
  <version>1.5.0</version>
</dependency>
```

### Example

```java
public class Application {
    public static void main(String[] args) {
        final GelfConfiguration config = new GelfConfiguration(new InetSocketAddress("example.com", 12201))
              .transport(GelfTransports.UDP)
              .queueSize(512)
              .connectTimeout(5000)
              .reconnectDelay(1000)
              .tcpNoDelay(true)
              .sendBufferSize(32768);

        final GelfTransport transport = GelfTransports.create(config);
        final GelfMessageBuilder builder = new GelfMessageBuilder("", "example.com")
                .level(GelfMessageLevel.INFO)
                .additionalField("_foo", "bar");

        boolean blocking = false;
        for (int i = 0; i < 100; i++) {
            final GelfMessage message = builder.message("This is message #" + i)
                    .additionalField("_count", i)
                    .build();

            if (blocking) {
                // Blocks until there is capacity in the queue
                transport.send(message);
            } else {
                // Returns false if there isn't enough room in the queue
                boolean enqueued = transport.trySend(message);
            }
        }
    }
}
```

## 更新到 graylog.jar

> @see [Java Archive (JAR) Files](https://mp.weixin.qq.com/s/ammU9NW2zPB6cWdbFfeloA)
> - [graylog2-server/pom.xml at 4.3.0 · Graylog2/graylog2-server · GitHub](https://github.com/Graylog2/graylog2-server/blob/4.3.0/distribution/pom.xml)
> - [graylog2-server/GelfOutput.java at 4.3.0 · Graylog2/graylog2-server · GitHub](https://github.com/Graylog2/graylog2-server/blob/4.3.0/graylog2-server/src/main/java/org/graylog2/outputs/GelfOutput.java)
> - [UDP 分包与组包_董哥的黑板报的博客](https://blog.csdn.net/qq_41453285/article/details/107236053)

```bash
# 编译项目
mvn clean compile

# 替换 graylog.jar 中的文件
jar uvf graylog.jar -C <绝对路径>/gelfclient/target/classes/ .
```


## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## License

Apache License, Version 2.0 -- http://www.apache.org/licenses/LICENSE-2.0
