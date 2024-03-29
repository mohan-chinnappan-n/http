## HTTP

HTTP is the application protocol that powers the Web. It began life as the so-called HTTP/0.9 protocol in 1991, and by 1999 had evolved to HTTP/1.1, which was standardized within the IETF (Internet Engineering Task Force). HTTP/1.1 was good enough for a long time but the ever changing needs of the Web called for a better suited protocol, and HTTP/2 emerged in 2015. More recently it was announced that the IETF is intending to deliver a new version - HTTP/3.  


----

Here's a markdown table comparing HTTP/1.1, HTTP/2.0, and HTTP/3.0 along with their advantages and issues:

| Feature     | HTTP/1.1                                  | HTTP/2.0                                 | HTTP/3.0                                 |
|-------------|-------------------------------------------|------------------------------------------|------------------------------------------|
| Protocol    | Text-based                                | Binary-based                             | Binary-based                             |
| Multiplexing| No                                        | Yes                                      | Yes                                      |
| Header Compression| No                                  | Yes (HPACK)                              | Yes (QPACK)                              |
| Server Push | No                                        | Yes                                      | Yes                                      |
| Encryption  | Optional (HTTPS)                          | Mandatory (HTTPS)                        | Mandatory (HTTPS)                        |
| Advantages  | - Simplicity                              | - Multiplexing                           | - Reduced latency                        |
|             | - Widely supported                        | - Header Compression                     | - Improved performance over UDP         |
|             |                                           | - Server Push                            | - Better handling of packet loss        |
|             |                                           | - Reduced latency                        | - Improved security                      |
| Issues      | - Head-of-line blocking                   | - Complexity                             | - Concerns over implementation maturity |
|             | - Performance issues with multiple requests| - Potential for "priority inversion"     | - Dependency on UDP                    |
|             | - Limited parallelism                     | - Dependency on HTTPS                    | - Limited adoption                       |

Please note that the issues mentioned are not exhaustive and may vary depending on specific implementations and use cases. Additionally, HTTP/3.0 is relatively new at the time of writing, so its issues and adoption may evolve over time.

## Priority inversion

"Priority inversion" is a potential issue in the context of HTTP/2.0, particularly concerning how streams are handled.

In HTTP/2.0, streams can have different priorities. This feature allows clients to specify the importance of certain resources, indicating which streams should be given higher priority for transmission by the server. However, it also opens up the possibility of priority inversion.

Priority inversion occurs when a lower-priority stream holds a dependency on a higher-priority stream. In this scenario, the server might prioritize the lower-priority stream to satisfy its dependency, even though higher-priority data is available. This situation can lead to suboptimal resource allocation and performance degradation.

For example, consider a webpage where a lower-priority resource, such as a background image, depends on a higher-priority resource, like critical CSS or JavaScript. If the server prioritizes the delivery of the background image over the critical resources, it can delay the rendering of the webpage, resulting in a poor user experience.

To mitigate priority inversion, it's essential for servers to effectively manage stream prioritization based on the dependencies and importance of resources. Additionally, clients should carefully set priorities to ensure that critical resources are transmitted promptly. However, despite efforts to manage priorities effectively, the potential for priority inversion remains a concern in HTTP/2.0 implementations.

In the context of HTTP/2.0, clients have the ability to set priorities for different resources to indicate their relative importance. By assigning priorities, clients can influence the order in which resources are transmitted by the server, thereby ensuring that critical resources are delivered promptly.

---

## Clients can carefully set priorities

Here's a breakdown of how clients can carefully set priorities:

1. **Understanding Resource Importance**: Clients should assess the importance of various resources within a webpage or application. Critical resources, such as essential CSS, JavaScript, or HTML files necessary for rendering the page correctly and quickly, should be identified.

2. **Setting Priority Levels**: HTTP/2.0 introduces the concept of stream prioritization, where each stream (representing a resource) is assigned a priority level. Priorities are indicated using integer values ranging from 0 to 2^31 - 1, where lower values indicate higher priority.

3. **Assigning Priority Dependencies**: Clients can establish dependencies between streams to reflect resource dependencies. For example, a CSS file may depend on the base HTML document for proper styling and rendering. By setting the CSS stream's dependency on the HTML stream, the client ensures that the CSS file is transmitted only after the HTML document is received and processed.

4. **Dynamic Adjustment**: Clients can dynamically adjust priorities based on user interactions or changes in resource importance. For instance, if a user scrolls down a webpage and triggers the loading of additional resources, the client may prioritize loading images or additional content that becomes visible.

5. **Testing and Optimization**: It's crucial for clients to test and optimize priority settings to ensure optimal performance. This may involve monitoring network traffic, analyzing resource loading patterns, and fine-tuning priority assignments to minimize latency and improve overall user experience.

By carefully setting priorities, clients can effectively guide the server in delivering critical resources first, thereby minimizing latency and enhancing the perceived speed and responsiveness of webpages and applications. This proactive approach to resource prioritization is fundamental to maximizing the benefits of HTTP/2.0's multiplexing and stream prioritization features.

### Example
Here's an example markup demonstrating how clients can set priorities in HTTP/2.0 using the `priority` field in the HTTP/2.0 headers:

```http
GET /example_page HTTP/2.0
Host: www.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.99 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Priority: 10   // Higher priority for the main HTML document

...

GET /styles.css HTTP/2.0
Host: www.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.99 Safari/537.36
Accept: text/css,*/*;q=0.1
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Priority: 20   // Lower priority for the CSS file

...

GET /logo.png HTTP/2.0
Host: www.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.99 Safari/537.36
Accept: image/avif,image/webp,image/apng,image/*,*/*;q=0.8
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Priority: 30   // Even lower priority for non-essential resources like images
```

In this example:

- The client requests the main HTML document (`/example_page`) with a high priority level of `10`.
- The CSS file (`/styles.css`) is requested with a lower priority level of `20`.
- The image file (`/logo.png`) is requested with an even lower priority level of `30`.

By setting different priority levels, the client instructs the server to prioritize the delivery of critical resources, such as the HTML document, over less important resources like CSS and images.


## http/2.0 HOL (head of the line) blocking

In HTTP/2.0, HOL (Head-of-Line) blocking refers to a potential performance bottleneck that can occur due to the way HTTP/2.0 handles multiplexed streams.

Multiplexing is a key feature of HTTP/2.0, allowing multiple requests and responses to be sent and received asynchronously over a single TCP connection. However, because all streams share the same connection, if one stream encounters a delay in transmission, it can hold up subsequent streams, causing them to be blocked.

Here's how HOL blocking can occur in HTTP/2.0:

1. **Shared Connection**: HTTP/2.0 uses a single TCP connection for multiple streams. While this improves efficiency by reducing connection overhead, it also means that all streams share the same network resources.

2. **Ordered Stream Delivery**: Streams in HTTP/2.0 are delivered in order. If a higher-priority stream experiences delay or congestion, it can block subsequent streams from being processed until it is completed.

3. **Dependency Tree**: Streams can have dependencies, and lower-priority streams may have to wait for higher-priority streams to be processed first. If a higher-priority stream is delayed, it can hold up dependent streams, even if they could be processed independently.

HOL blocking can lead to suboptimal resource utilization and increased latency, particularly in scenarios where there's a mix of high and low-priority requests. It can negate some of the performance benefits of multiplexing in HTTP/2.0.

To mitigate HOL blocking, HTTP/2.0 clients and servers can implement strategies such as:

- **Prioritization**: Setting appropriate priorities for streams can help ensure that critical resources are delivered promptly, reducing the risk of HOL blocking.
- **Stream Dependency Management**: Careful management of stream dependencies can help minimize the impact of delays on dependent streams.
- **Flow Control**: Implementing flow control mechanisms can prevent individual streams from monopolizing network resources, reducing the likelihood of HOL blocking.
- **Optimized Server Configuration**: Servers can be configured to efficiently handle concurrent streams and manage resources to minimize the risk of HOL blocking.

Overall, while HTTP/2.0's multiplexing offers significant performance benefits, HOL blocking is a potential issue that needs to be addressed through careful implementation and optimization strategies.



## What is HTTP/2?
HTTP/2 is a replacement for how HTTP is expressed “on the wire.” It is not a ground-up rewrite of the protocol; HTTP methods, status codes and semantics are the same, and it should be possible to use the same APIs as HTTP/1.x (possibly with some small additions) to represent the protocol.

The focus of the protocol is on **performance; specifically, end-user perceived latency, network and server resource usage**. One major goal is to allow the **use of a single connection from browsers to a Web site**.
HTTP/2 provides an optimized transport for HTTP semantics.  HTTP/2 supports all of the core features of HTTP/1.1 but aims to be more efficient in several ways.


## What is new in HTTP/3
An important difference in HTTP/3 is that it runs on **QUIC**, a new transport protocol. 
QUIC (Quick UDP Internet Connections) is designed for mobile-heavy Internet usage in which people carry smartphones that constantly **switch from one network to another** as they move about their day. This was not the case when the first Internet protocols were developed: devices were less portable and did not switch networks very often.
The use of QUIC means that HTTP/3 relies on the **User Datagram Protocol (UDP)**, not the Transmission Control Protocol (TCP). Switching to UDP will enable faster connections and faster user experience when browsing online.

![Layer Cake](https://blog.cloudflare.com/content/images/2019/01/http3-stack.png)

## TCP and UDP
- TCP (Transmission Control Protocol)
    - Has guaranteed delivery and the correct order of the packets. Thus it created reliable connections, and later on reliable streams of information

- UDP (User Datagram Protocol)
    - Quick, fire and forget standard: you threw a packet across the network and it was caught, or sometimes it wasn’t.


## QUIC

QUIC tries to make an HTTPS connection between a computer (phone) and server work reliably despite the poor conditions, it does this with a collection of technologies.

- QUIC  solves the HTTP/2 HoL problem.
    - Since HTTP/2 sits on top of TCP and TCP guarantees delivery order if a packet gets lost the entire TCP connection has to wait while the missing packet is retransmitted.
    - That’s OK if only one stream of data is passing over the TCP connection, but for efficiency it’s better to have **multiple streams per connection**.
    - This means **all streams wait** when a packet gets lost. 
    - QUIC solves that because it doesn’t rely on TCP for delivery and ordering and can make **an intelligent decision** about which streams need to wait and which can continue when a packet goes astray.
    - QUIC controls all aspects of the connect it **merges together connection and encryption into a single handshake**.





## References

- [http/2](https://http2.github.io/)
- [http/2 spec](https://datatracker.ietf.org/doc/html/rfc7540)

- [The Full Picture on HTTP/2 and HOL Blocking](https://engineering.salesforce.com/the-full-picture-on-http-2-and-hol-blocking-7f964b34d205/)

- [What is HTTP/3?](https://www.cloudflare.com/learning/performance/what-is-http3/)
- [HTTP/3: From root to tip](https://blog.cloudflare.com/http-3-from-root-to-tip)
- [The QUICening](https://blog.cloudflare.com/the-quicening/)