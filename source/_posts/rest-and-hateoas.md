---
title: rest and hateoas
toc: true
tags:
    - rest
    - design
date: 2019-06-04 15:47:32
---

# why REST?

> The World Wide Web is arguably the world's largest distributed application. Understanding the key architectural principles underlying the Web can help explain its technical success and may lead to improvements in other distributed applications, particularly those that are amenable to the same or similar methods of interaction. REST contributes both the rationale behind the modern Web's software architecture and a significant lesson in how software engineering principles can be systematically applied in the design and evaluation of a real software system. ---- Fielding's REST dissertation

In summary, to meet the needs of an Internet-scale distributed hypermedia system, some properties are promoted: **scalability, interoperability, simplicity, and flexibility**. To do this, the REST(Representational State Transfer) architecture is developed for designing good networked applications.

REST is a coordinated set of architectural constraints that attempts to minimize latency and network communication while at the same time maximizing the independence and scalability of component implementations. This is achieved by placing constraints on connector semantics where other styles have focused on component semantics. REST enables the caching and reuse of interactions, dynamic substitutability of components, and processing of actions by intermediaries, thereby meeting the needs of an Internet-scale distributed hypermedia system.

# What's REST

REST is an architectural style that defines a set of constraints and elements for designing networked applications.

## Constraints(Deriving REST)

These constraints are developed step by step along the development of WWW.

### Client-Server

**Constraint:**

Separating the user interface concerns from the data storage concerns. **Separation of concerns** is the principle behind the client-server constraints.

**Why client-server constraint:**

By separating the user interface concerns from the data storage concerns, we improve the **<u>portability</u>** of the user interface across multiple platforms and improve **<u>scalability</u>** by simplifying the server components. Perhaps most significant to the Web, however, is that the separation allows the components to **<u>evolve independently</u>**, thus supporting the Internet-scale requirement of multiple organizational domains.

### Stateless

**Constraint:**

Communication must be stateless in nature. The server does not maintain any client state between requests. Each request from the client must contain all the necessary information for the server to understand and process it. Session state is therefore kept entirely on the client.

**Why stateless constraint**:

It's a design trade-off.

It induces the properties of **<u>visibility</u>**, **<u>reliability</u>**, and **<u>scalability</u>**. Visibility is improved because a monitoring system does not have to look beyond a single request datum in order to determine the full nature of the request. Reliability is improved because it eases the task of recovering from partial failures [[133](https://www.ics.uci.edu/~fielding/pubs/dissertation/references.htm#ref_133)]. Scalability is improved because not having to store state between requests allows the server component to quickly free resources, and further simplifies implementation because the server doesn't have to manage resource usage across requests.

The disadvantage is that it may <u>decrease network performance</u> by increasing the repetitive data (per-interaction overhead) sent in a series of requests, since that data cannot be left on the server in a shared context. In addition, placing the application state on the client-side <u>reduces the server's control over consistent application behavior</u>, since the application becomes dependent on the correct implementation of semantics across multiple client versions.

### Cache

**Constraint:**

Require that the data within a response to a request be implicitly or explicitly labeled as cacheable or non-cacheable. If a response is cacheable, then a client cache is given the right to reuse that response data for later, equivalent requests.

REST supports caching at various levels (client, server, and intermediary) to improve performance and reduce the load on the server.

**Why cache constraint**:

To improve network **<u>efficiency</u>**.

The trade-off, however, is that a cache can **<u>decrease reliability</u>** if stale data within the cache differs significantly from the data that would have been obtained had the request been sent directly to the server.

### Uniform Interface

From here, the constraints are added to the Web's architectural style in order to guide the extensions that form the modern Web architecture.

Emphasis on a uniform interface between components is the **central feature** that distinguishes the REST architectural style from other network-based styles.

**Constraint:**

Use a uniform set of interfaces to communicate among components. For example, standard methods (GET, POST, PUT, PATCH, DELETE) for manipulating resources, resource identification through URLs, and hypermedia as the engine of application state (HATEOAS), which allows clients to discover and navigate resources dynamically.

**Why uniform interface constraint:**

By applying the software engineering principle of generality to the component interface, the overall system architecture is **<u>simplified</u>** and the **<u>visibility</u>** of interactions is improved. Implementations are decoupled from the services they provide, which encourages **<u>independent evolvability</u>**.

The trade-off, though, is that a uniform interface **<u>degrades efficiency</u>**, since information is transferred in a standardized form rather than one which is specific to an application's needs. The REST interface is designed to be efficient for **large-grain hypermedia data transfer**, optimizing for the common case of the Web, but resulting in an interface that is not optimal for other forms of architectural interaction.

#### Interface Constraints

In order to obtain a uniform interface, multiple architectural constraints are needed to guide the behavior of components. REST is defined by four interface constraints: **<u>identification of resources</u>; <u>manipulation of resources through representations</u>; <u>self-descriptive messages</u>; and, <u>hypermedia as the engine of application state</u>.**

##### Resource & Resource Identifiers

RESTful systems are built around the concept of resources, which are identified by unique URLs (Uniform Resource Locators).

Resource is a concept. The real entities or data it denotes may vary by time.

REST defines a resource to be the semantics of what the author intends to identify, rather than the value corresponding to those semantics at the time the reference is created.

> A resource is **a <u><em>conceptual</em></u> mapping** to a set of entities, not the entity that corresponds to the mapping at any particular point in time.
> 
> More precisely, a resource *R* is **a temporally varying membership function** *M*R*(t)*, which for time *t* maps to a set of entities, or values, which are equivalent. The values in the set may be *resource representations* and/or *resource identifiers*. Some resources are static in the sense that, when examined at any time after their creation, they always correspond to the same value set.
> 
> For example, the "authors' preferred version" of an academic paper is a mapping whose value changes over time, whereas a mapping to "the paper published in the proceedings of conference X" is static. These are two distinct resources, even if they both map to the same value at some point in time. The distinction is necessary so that both resources can be identified and referenced independently. A similar example from software engineering is the separate identification of a version-controlled source code file when referring to the "latest revision", "revision number 1.2.7", or "revision included with the Orange release."

While there is some conceptual alignment between storage systems and REST APIs, a service with a resource-oriented API is not necessarily a database, and has enormous flexibility in how it interprets resources and methods. For example, creating a calendar event (resource) may create additional events for attendees, send email invitations to attendees, reserve conference rooms, and update video conference schedules.

##### Representaion

When transferring resource among components, it's transferring a representation (e.g. HTTP Message entity) of that resource in a format (e.g. JSON/XML) . TA representation is a snapshot of a resource's state at a particular point in time. Clients interact with resources by sending representations in requests and receiving representations in responses. The representation contains:

- data

- metadata: may include both representation metadata and resource metadata: information about the resource that is not specific to the supplied representation.

- metadata to describe the metadata

- control data:  a kind of metadata. It defines the purpose of a message between components, such as the action being requested or the meaning of a response. e.g. cacheable info, reponse status, http method, etc.

- media type: The data format of a representation is a media type. It can directly impact the user-perceived performance. See [Media Type](#Media-Type)

> Other commonly used but less precise names for a representation include: document, file, and HTTP message entity, instance, or variant.
> 
> Depending on the message control data, a given representation may indicate the current state of the requested resource, the desired state for the requested resource, or the value of some other resource, such as a representation of the input data within a client's query form, or a representation of some error condition for a response.

Whether the representation is in the same format as the raw source (e.g. data in a relational database), or is derived from the source, remains hidden behind the interface. This brings the separation of concerns of the client-server style without the server scalability problem, allows information hiding through a generic interface to enable encapsulation and evolution of services.

##### Self-descriptive Messages

REST enables intermediate processing by constraining messages to be self-descriptive: interaction is stateless between requests, standard methods and media types are used to indicate semantics and exchange information, and responses explicitly indicate cacheability.

##### HATEOAS

HATEOAS (Hypermedia as the Engine of Application State) emphasizes the use of hypermedia links in API responses to enable clients to discover and navigate resources dynamically. With HATEOAS, the server includes relevant links alongside the resource representation, providing clients with information on what actions they can take next. See [HATEOAS](#HATEOAS-1).

### Layered System

**Constraint:**

Allow an architecture to be composed of hierarchical layers by constraining component behavior such that each component cannot "see" beyond the immediate layer with which they are interacting.

**Why layered system constraint:**

By restricting knowledge of the system to a single layer, we place a **<u>bound on the overall system complexity</u>** and promote substrate **<u>independence</u>**. Layers can be used to **<u>encapsulate</u>** legacy services and to protect new services from legacy clients, simplifying components by moving infrequently used functionality to a shared intermediary. Intermediaries can also be used to improve system **<u>scalability</u>** by enabling load balancing of services across multiple networks and processors.

The primary disadvantage is that they add overhead and latency to the processing of data, reducing user-perceived performance.

Within REST, intermediary components can actively transform the content of messages because the messages are self-descriptive and their semantics are visible to intermediaries, and the server is stateless so that the message contains all the necessary information.

### Code-On-Demand (Optional)

This is an optional constraint.

**Constraint:**

Allow client functionality to be extended by downloading from server and executing code in the form of applets or scripts.

**Why code-on-demand constraint:**

This **<u>simplifies</u>** clients by reducing the number of features required to be pre-implemented. Allowing features to be downloaded after deployment improves system **<u>extensibility</u>**.

However, it also **<u>reduces visibility</u>**, and thus is only an optional constraint within REST.

Here's a simplified code example to demonstrate the concept of code-on-demand in REST using JavaScript:

Suppose you have a RESTful API that provides a resource, such as "books". The server includes a link to executable JavaScript code in the response, allowing the client to dynamically fetch and execute that code.

Server response:

```json
{
  "title": "The Hitchhiker's Guide to the Galaxy",
  "author": "Douglas Adams",
  "link": "http://example.com/scripts/book-script.js"
}
```

Client-side code:

```javascript
// Fetch the resource from the server
fetch('http://example.com/api/books/42')
  .then(response => response.json())
  .then(data => {
    // Check if a script link is provided in the response
    if (data.link) {
      // Fetch the script file from the provided link
      return fetch(data.link);
    } else {
      // No script link provided, continue processing the data
      return Promise.resolve(null);
    }
  })
  .then(scriptResponse => {
    // Execute the fetched script (if available)
    if (scriptResponse) {
      return scriptResponse.text();
    } else {
      return Promise.resolve(null);
    }
  })
  .then(scriptCode => {
    // Execute the fetched script code (if available)
    if (scriptCode) {
      eval(scriptCode); // Execute the script code
    }

    // Continue processing the resource data or perform other actions
    // ...
  });
```

The JavaScript code fetched from the server can extend the functionality of the client, modify the UI, or perform other custom operations based on the specific logic defined in the script. This dynamic execution of code-on-demand allows the server to provide additional behavior or features to the client without requiring the client to have prior knowledge of them.

Note that the usage of `eval()` in this example is for demonstration purposes only. In real-world scenarios, you should carefully evaluate the security implications and consider using safer alternatives like `Function()`, strict sandboxing, or other approaches depending on your specific requirements.

# RPC

RPC (Remote Procedure Call) is a communication protocol that allows a program on one computer to invoke a procedure or method in another computer or distributed system. It enables programs to communicate and exchange data across different machines or network boundaries.

In RPC, the calling program (client) sends a request to a remote program (server), specifying the desired procedure to be executed along with any necessary parameters. The server processes the request, executes the procedure, and returns the result to the client.

## RPC example

Here's an example of RPC implementation using Java and Python:

Java Example:

1. Define an interface for the remote procedure:

```java
public interface CalculatorService {
    int add(int a, int b);
}
```

2. Implement the server that provides the remote procedure:

```java
public class CalculatorServiceImpl implements CalculatorService {
    public int add(int a, int b) {
        return a + b;
    }
}
```

3. Create a server that exposes the remote procedure:

```java
import java.rmi.registry.Registry;
import java.rmi.registry.LocateRegistry;
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;

public class Server {
    public static void main(String[] args) {
        try {
            CalculatorService calculator = new CalculatorServiceImpl();
            CalculatorService stub = (CalculatorService) UnicastRemoteObject.exportObject(calculator, 0);

            Registry registry = LocateRegistry.createRegistry(1099);
            registry.bind("CalculatorService", stub);

            System.out.println("Server started");
        } catch (Exception e) {
            System.err.println("Server exception: " + e.toString());
            e.printStackTrace();
        }
    }
}
```

4. Create a client that invokes the remote procedure:

```java
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class Client {
    public static void main(String[] args) {
        try {
            Registry registry = LocateRegistry.getRegistry("localhost", 1099);
            CalculatorService calculator = (CalculatorService) registry.lookup("CalculatorService");

            int result = calculator.add(5, 3);
            System.out.println("Result: " + result);
        } catch (Exception e) {
            System.err.println("Client exception: " + e.toString());
            e.printStackTrace();
        }
    }
}
```

Python Example:

1. Install the `xmlrpc` library if needed: `pip install xmlrpc.client`

2. Implement the server that provides the remote procedure:

```python
from xmlrpc.server import SimpleXMLRPCServer

def add(a, b):
    return a + b

server = SimpleXMLRPCServer(("localhost", 8000))
server.register_function(add, "add")
server.serve_forever()
```

3. Create a client that invokes the remote procedure:

```python
import xmlrpc.client

proxy = xmlrpc.client.ServerProxy("http://localhost:8000/")
result = proxy.add(5, 3)
print("Result:", result)
```

## API vs RPC

RPC (Remote Procedure Call) and HTTP API (Application Programming Interface) calls are two different approaches for communication between distributed systems. Let's compare them based on a few key aspects:

1. Communication Protocol:
   
   - RPC: Typically uses a binary protocol with a defined interface and specific methods or procedures.
   - HTTP API: Uses the HTTP protocol with a request-response model, where requests and responses are typically in a textual format (e.g., JSON, XML).

2. Interface Definition:
   
   - RPC: In RPC, the interface is usually defined using a specific interface definition language (IDL), which allows for a strongly typed contract between the client and server.
   - HTTP API: The interface is defined through HTTP methods (GET, POST, PUT, DELETE) and resource endpoints, and the data format is often described using open standards like OpenAPI (formerly known as Swagger).

3. Data Representation:
   
   - RPC: Typically focuses on remote method invocations and parameter passing. The data exchanged is often binary or serialized objects.
   - HTTP API: Supports a broader range of data representations, including JSON, XML, or other formats. It allows for more flexible data exchange and handling.

4. Transport:
   
   - RPC: Can use various transport protocols, including TCP, UDP, or custom protocols optimized for performance and low latency.
   - HTTP API: Primarily uses HTTP as the transport protocol, allowing for easy interoperability and leveraging existing infrastructure like web servers and proxies.

5. Semantic Meaning:
   
   - RPC: Typically emphasizes the invocation of specific procedures or methods and is closely tied to the implementation details of the server.
   - HTTP API: Focuses on resource-oriented design and follows REST principles. It provides a more abstract and standardized approach for building web services, emphasizing resource identification, manipulation, and statelessness.

### Higher Performance

RPC is often perceived to provide higher performance compared to HTTP APIs.

RPC protocols are typically designed to be more efficient in terms of serialization and deserialization of data. They can be designed to use lower-level network protocols, such as TCP or UDP, or custom protocols tailored for specific performance requirements. They often use binary formats and optimize data transmission for specific programming languages, leading to reduced overhead compared to textual data formats used in HTTP APIs like JSON or XML.

However, Performance can vary depending on the specific implementation, use case, and optimization efforts. While RPC can provide advantages in certain scenarios, the performance difference may not always be significant compared to well-designed and optimized HTTP APIs.

### RPC Drawbacks

There are some drawbacks to consider when comparing it with HTTP API:

1. Complexity: RPC can introduce additional complexity compared to HTTP APIs. It often requires generating and maintaining code bindings for different programming languages. This can add overhead in terms of development time, debugging, and maintenance.

2. Tight Coupling: RPC tends to create tighter coupling between the client and server. Both the client and server need to understand the RPC protocol and share the same data types and contracts. This can limit the flexibility to evolve and modify the system independently.

3. Firewall and Proxy Limitations: RPC may face challenges when crossing firewalls or proxies. Some RPC frameworks use custom protocols or ports that can be blocked or restricted by network policies. HTTP, on the other hand, is widely supported and passes through most network infrastructures.

4. Limited Interoperability: RPC frameworks often have language-specific implementations, which can limit interoperability between different programming languages. It may require additional effort to support cross-language communication compared to the more standardized nature of HTTP APIs.

5. Scalability and Load Balancing: Scaling RPC-based systems can be more complex compared to HTTP APIs. Load balancing, service discovery, and fault tolerance mechanisms need to be implemented at the RPC framework level or handled manually.

6. Tooling and Ecosystem: The ecosystem around RPC frameworks may not be as mature or extensive as that of HTTP APIs. HTTP has a wide range of tools, libraries, and frameworks that support various aspects of API development, testing, documentation, and monitoring.

## API or RPC

On the Internet, HTTP REST APIs have been hugely successful. In 2010, about 74% of public network APIs were HTTP REST (or REST-like) APIs, most using JSON as the wire format.

While HTTP/JSON APIs are very popular on the Internet, the amount of traffic they carry is smaller than traditional RPC APIs. For example, about half of Internet traffic in America at peak time is **video content**, and few people would consider using HTTP/JSON APIs to deliver such content for obvious **performance reasons**. Inside data centers, many companies use socket-based RPC APIs to carry most network traffic, which can involve orders of magnitude more data (measured in bytes) than public HTTP/JSON APIs.

RPC is often favored in scenarios where performance and low-level control are critical, such as high-performance computing or distributed systems. On the other hand, HTTP APIs are widely used for building web services, enabling interoperability, and leveraging the extensive tooling and infrastructure available for the HTTP protocol. API provides a more flexible, standardized, and widely supported approach for integrating applications and services in the cloud environment.

## RPC Application

Sure! Here are a few examples of business products or systems that may use RPC internally to implement certain functions:

1. Distributed File Systems: Distributed file systems like Hadoop Distributed File System (HDFS) or Google File System (GFS) use RPC for communication between different components. RPC enables data transfer, coordination, and metadata operations across multiple nodes in the distributed file system.

2. Microservices Architecture: In a microservices-based architecture, where applications are divided into small, independent services, RPC can be used for inter-service communication. Services can use RPC calls to invoke specific functions or exchange data with other services, allowing them to work together to provide a cohesive application.

3. Service-Oriented Architecture (SOA): In a service-oriented architecture, RPC can be used for communication between services. RPC calls allow services to invoke methods or procedures on remote services to access their functionality or exchange data. This enables the composition and integration of different services within the overall system.

4. Cloud Computing Platforms: Cloud computing platforms like Amazon Web Services (AWS), Microsoft Azure, and Google Cloud Platform (GCP) often use RPC internally for various purposes. RPC enables communication between different components of the platform, such as virtual machines, storage systems, and network infrastructure, allowing them to work together to provide cloud services.

5. Enterprise Resource Planning (ERP) Systems: ERP systems typically involve multiple modules and components that need to communicate with each other. RPC can be used internally to enable communication and data exchange between different modules within the ERP system, facilitating seamless integration and functionality across various business processes.

It's worth noting that while these examples highlight the use of RPC in certain business products or systems, the specific implementation details may vary. RPC is just one of several communication mechanisms available, and its adoption depends on the specific requirements and architecture of the system being developed.

### HDFS application

Hadoop Distributed File System (HDFS) and Google File System (GFS) use RPC internally for various communication tasks. Here's how RPC is typically used in these distributed file systems:

1. Metadata Operations: Both HDFS and GFS employ RPC for metadata operations, such as retrieving file system metadata, creating directories, renaming files, and managing permissions. Clients make RPC calls to the metadata server or master node, which handles these operations and maintains the file system's metadata.

2. Data Transfer: When a client wants to read or write data to the distributed file system, it needs to communicate with the appropriate data nodes that store the data blocks. RPC is used to coordinate and facilitate data transfer between the client and data nodes. The client sends an RPC request to the data node, specifying the block it wants to read or write, and the data node responds accordingly.

3. Heartbeat and Health Monitoring: Both HDFS and GFS use RPC for heartbeat messages between different nodes in the cluster. Nodes periodically send heartbeat RPC requests to indicate their availability and health status to the central server or master node. This helps in detecting failed or slow nodes and enables the system to take appropriate actions.

4. Coordination and Control: RPC is used for coordination and control tasks within the distributed file system. For example, in HDFS, the NameNode communicates with the DataNodes using RPC to maintain a consistent view of the file system and to control replication and block allocation policies. Similarly, in GFS, the master node coordinates activities among chunk servers through RPC calls.

These are just a few examples of how HDFS and GFS leverage RPC internally to facilitate communication, coordination, and data transfer in their distributed file systems. RPC allows different components of the file system to interact and work together seamlessly, enabling efficient storage and retrieval of data across multiple nodes in the cluster.

# <a name="hateoas" />HATEOAS

HATEOAS (Hypermedia as the Engine of Application State) emphasizes the use of hypermedia links in API responses to enable clients to discover and navigate resources dynamically.

HATEOAS promotes loose coupling between the client and server, as clients rely on the hypermedia links to discover and interact with resources. It promotes scalability and enables the server to evolve the API without breaking client functionality, as long as the hypermedia links and their semantics remain consistent.

## Application State

In the context of HATEOAS (Hypermedia as the Engine of Application State), the term "application state" refers to the current condition or status of the application from the client's perspective. It represents the combination of data, resources, and possible actions that are available for the client to interact with at a given point in time.

Application state encapsulates the information needed by the client to understand what it can do next within the application. It includes details about **the current resource being accessed, the available actions that can be performed on that resource, and any related resources that can be explored.**

In a HATEOAS-compliant API, the server includes hypermedia links within API responses, which provide the necessary information to drive the application state. These links act as navigational cues for the client, guiding it to the next set of available actions or transitions.

Application state in REST is represented and transferred through representations exchanged between client and server. The model application is therefore an engine that moves from one state to the next by examining and choosing from among the alternative state transitions in the current set of representations.

## Pros & Cons

While HATEOAS promotes discoverability and decoupling between clients and servers, its practical adoption may vary based on the specific requirements and constraints of an API.

**Advantages**:

1. Discoverability: HATEOAS provides self-descriptive APIs by including links in API responses. This allows clients to discover available actions dynamically without relying heavily on prior knowledge or documentation.

2. Flexibility and Evolution: With HATEOAS, servers can evolve the API by adding new resources, endpoints, or actions without breaking existing clients. Clients that understand and navigate the provided links can adapt to changes more easily.

3. Reduced Coupling: Clients that follow hypermedia links are less tightly coupled to the API structure. They can navigate resources and interact with the API based on the available links, which allows for more flexibility in server-side changes.

**Considerations**:

1. Learning Curve: While HATEOAS promotes dynamic exploration of API capabilities, it requires clients to understand and interpret the provided links. Clients may need to invest additional effort in parsing and processing the hypermedia links to effectively interact with the API.

2. Tooling and Documentation: Many existing client libraries and tools are built around static API documentation rather than dynamic exploration of hypermedia links. Clients may rely on API documentation to understand available endpoints, request structures, and response formats.

3. Standardization and Consistency: HATEOAS lacks standardization in terms of link formats, semantics, and conventions. API providers need to ensure consistency and clarity in the representation of links to avoid ambiguity and confusion for clients.

In essence, the concept of application state in HATEOAS emphasizes the dynamic nature of APIs, where clients can determine their next steps based on the information presented to them through hypermedia links, rather than relying solely on predefined knowledge or fixed API structures.

In practice, the adoption of HATEOAS varies depending on the goals, complexity, and constraints of the API. The level of its implementation may vary based on the trade-offs between discoverability and the practicality of client development and tooling. API providers need to carefully consider the needs of their client developers and strike a balance between HATEOAS and traditional documentation approaches.

## Example

Here's an example to illustrate HATEOAS in a simple API response:

```json
{
  "id": 123,
  "name": "John Doe",
  "age": 30,
  "links": [
    {
      "rel": "self",
      "href": "/users/123"
    },
    {
      "rel": "update",
      "href": "/users/123",
      "method": "PUT"
    },
    {
      "rel": "delete",
      "href": "/users/123",
      "method": "DELETE"
    }
  ]
}
```

In the above example, the API response provides the representation of a user resource with the fields `id`, `name`, and `age`. Additionally, it includes a `links` array that contains hypermedia links related to the resource.

By including these hypermedia links, the server guides the client on the available actions it can take on the resource without requiring prior knowledge of the API's structure or endpoints. Clients can dynamically navigate the API by following the provided links and perform actions based on the resources' available state transitions.

## Auto Generation

Here are some patterns that can be used to generate HATEOAS links:

1. Resource-based Links: In this pattern, the HATEOAS links are generated based on the resources and their relationships. For example, if you have a "users" resource, you can generate links like "/users/{userId}" to retrieve a specific user or "/users" to retrieve a list of users.

2. CRUD Operations: HATEOAS links can be generated based on the CRUD operations (Create, Read, Update, Delete) supported by the API. For example, you can generate links like "/users/{userId}" for retrieving a user, "/users/{userId}" for updating a user, and "/users/{userId}" for deleting a user.

3. Pagination Links: If your API supports pagination, you can generate links for navigating through the paginated results. For example, you can include links like "prev", "next", "first", and "last" to allow users to navigate to the previous, next, first, and last pages of the results.

4. Related Resources: HATEOAS links can be generated to represent relationships between resources. For example, if a user has associated orders, you can include a link to retrieve the orders for that user.

# <a name="media-type" />Media Type

Impact data transfer efficiency, parsing speed, network overhead, and ease of interaction.

1. JSON vs. XML:
   
   - JSON (JavaScript Object Notation) is a lightweight data interchange format known for its simplicity and efficiency. Compared to XML (eXtensible Markup Language), JSON typically requires less bandwidth and is easier to parse, resulting in faster data transfer and improved performance.

2. Protocol Buffers:
   
   - Protocol Buffers is a binary serialization format developed by Google. It offers a compact representation of structured data, allowing for efficient encoding and decoding. The smaller message size and faster parsing of Protocol Buffers can enhance the performance of a hypermedia system, especially in scenarios with limited network bandwidth.

3. HAL (Hypertext Application Language):
   
   - HAL is a media type that provides a simple and consistent way to express hypermedia links and resources. It defines conventions for embedding links within JSON or XML representations. By promoting standardized link formats, HAL enables clients to easily navigate and interact with hypermedia resources, leading to improved usability and perceived performance.

4. GraphQL:
   
   - GraphQL is a query language and runtime for APIs that allows clients to request specific data structures and reduce over-fetching. The design of GraphQL enables clients to retrieve precisely the data they need, reducing network round trips and improving performance. This flexibility and granularity in data retrieval can significantly impact the perceived performance of a hypermedia system.

5. MsgPack:
   
   - MsgPack is a binary serialization format that aims to be more efficient than JSON or XML. It offers compact representation, fast encoding/decoding, and supports a wide range of data types. The reduced message size and efficient serialization provided by MsgPack can enhance the performance of data transfer in a distributed hypermedia system.

# Best Practices

Some design ideas can reference [Google API Design Guide](https://cloud.google.com/apis/design). For example:

- how to use the standard methods

- keep a limited set of custom methods

- some standard fields naming and semantics

- common error codes & status

## PUT vs PATCH

PUT:

- PUT is typically used when you want to completely replace the existing resource with the new representation provided in the request payload.
- If the resource exists, PUT will overwrite it with the new representation. If it doesn't exist, PUT may create a new resource with the provided representation.
- An API with an `Update` method that supports resource creation **should** also provide a `Create` method. Rationale is that it is not clear how to create resources if the `Update` method is the only way to do it.
- PUT is idempotent, meaning that sending the same request multiple times will have the same effect as sending it once.

PATCH:

- PATCH is used when you want to apply a partial update to the resource, modifying only specific fields or properties.
- PATCH is not necessarily idempotent since multiple patch requests can have different effects based on the state of the resource.

While both PUT and PATCH have their uses, PUT is often seen as less commonly used compared to PATCH in practice. Because it means a full resource update, which's not recommended:

1. Overhead: It can result in unnecessary data transfer, especially if the resource is large or contains many fields. This can increase network bandwidth usage and add overhead to the update process.

2. Concurrency Issues: In multi-user or distributed systems, if multiple clients attempt to update the same resource simultaneously, conflicts can occur when the complete representation is used. Granular updates that only modify specific fields or properties of the resource can help mitigate these conflicts.

3. Data Integrity and Validation: In some cases, a full resource update may bypass data validation and integrity checks that would be applied when updating specific fields.

4. Caching and Performance Optimization: Full resource updates may invalidate the cached representations, leading to increased load on the server and reduced caching effectiveness.

# References

1. "Representational State Transfer (REST)" - This is the official chapter from the Fielding Dissertation, where REST was introduced by Roy Fielding, one of its creators:
   
   - Link: [Fielding Dissertation](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)

2. "HTTP/1.1 Semantics and Content" - This document describes the HTTP protocol and its request methods, headers, and response codes, which are foundational for RESTful APIs:
   
   - Link: [RFC 7231](https://tools.ietf.org/html/rfc7231)

3. "RESTful Web Services" - This is the official documentation for Java's JAX-RS API, which provides a set of annotations and tools for building RESTful web services:
   
   - Link: [JAX-RS Specification](https://jax-rs-spec.java.net/)

4. "Building Web APIs with ASP.NET Core" - Microsoft's official documentation on creating RESTful APIs using ASP.NET Core framework:
   
   - Link: [ASP.NET Core Web API Documentation](https://docs.microsoft.com/en-us/aspnet/core/web-api/?view=aspnetcore-5.0)

5. "Building and Documenting Web APIs with Django Rest Framework" - Django Rest Framework (DRF) is a powerful toolkit for building RESTful APIs with Django. The official documentation provides comprehensive guides and examples:
   
   - Link: [Django Rest Framework Documentation](https://www.django-rest-framework.org/)

6. "Building RESTful Web Services with Node.js" - This is the official documentation for Express.js, a popular Node.js framework for building web applications and RESTful APIs:
   
   - Link: [Express.js Documentation](https://expressjs.com/)

7. "API Design Guide" - This is Google's API design guide, which provides best practices and guidelines for designing RESTful APIs:
   
   - Link: [Google API Design Guide](https://cloud.google.com/apis/design)

8. "REST API Tutorial" - This tutorial by RestfulAPI.net covers the basics of RESTful APIs, HTTP methods, status codes, and more:
   
   - Link: [REST API Tutorial](https://restfulapi.net/)

9. [why hateoas is useless and what that means for REST](https://medium.com/@andreasreiser94/why-hateoas-is-useless-and-what-that-means-for-rest-a65194471bc8)

10. [what does hateoas have to do with Application State](https://softwareengineering.stackexchange.com/questions/384704/what-does-hateoas-have-to-do-with-application-state)