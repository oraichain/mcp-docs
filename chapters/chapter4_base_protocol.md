# Model Context Protocol

### Chapters

- [Chapter 1: Introduction](./chapter1_introduction.md)
- [Chapter 2: Fundamentals](./chapter2_fundamentals.md)
- [Chapter 3: Architecture](./chapter3_architecture.md)
- [Chapter 4: Base Protocol](./chapter4_base_protocol.md)
- [Chapter 5: Server Primitives](./chapter5_server_primitives.md)
- [Chapter 6: Client Primitives](./chapter6_client_primitives.md)
- [Chapter 7: Security](./chapter7_security.md)
- [Chapter 8: Implementation](./chapter8_implementation.md)
- [Chapter 9: Case Studies](./chapter9_case_studies.md)
- [Chapter 10: Future Directions](./chapter10_future_directions.md)

[← Back to Table of Contents](/)

# Chapter 4: The Base Protocol

## 4.1 JSON-RPC Foundation

The Model Context Protocol (MCP) is built on a solid foundation: the JSON-RPC 2.0 specification. This choice provides a well-established, language-agnostic framework for remote procedure calls that balances simplicity with power.

JSON-RPC is a lightweight remote procedure call (RPC) protocol that uses JSON (JavaScript Object Notation) for data encoding. It's designed to be simple, yet flexible enough to support complex interactions. For MCP, this foundation offers several advantages:

1.  **Widespread Support**: JSON is supported in virtually every programming language, making MCP implementations accessible across different technology stacks.
2.  **Human-Readable Format**: The text-based nature of JSON makes debugging and development more straightforward compared to binary protocols.
3.  **Stateless Design**: The base protocol is stateless, with each message containing all the information needed to understand it, simplifying implementation.
4.  **Extensibility**: JSON's flexible structure allows for protocol extensions without breaking backward compatibility.

In MCP, all communication between clients and servers follows the JSON-RPC 2.0 specification. Each message is a JSON object with specific fields that identify its purpose and content. This standardized approach ensures that different implementations can reliably communicate with each other.

## 4.2 Message Types: Requests, Responses, and Notifications

MCP defines three fundamental types of messages that form the backbone of all client-server communication:

### Requests

Requests are messages sent to initiate an operation. They always include:

- A unique identifier (`id`) that will be matched in the response
- A method name that specifies the operation to perform
- Optional parameters for the operation

For example, a request to list available resources might look like:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "resources/list",
  "params": {
    "cursor": "optional-cursor-value"
  }
}
```

Requests are bidirectional—both clients and servers can send requests, depending on the operation being performed.

### Responses

Responses are messages sent in reply to requests. They always include:

- The same identifier (`id`) as the corresponding request
- Either a result object or an error object

Responses are further categorized as either successful results or errors:

#### Successful Results

A successful response includes a `result` field containing the operation's output:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "resources": [
      {
        "uri": "file:///project/src/main.rs",
        "name": "main.rs",
        "description": "Primary application entry point",
        "mimeType": "text/x-rust"
      }
    ],
    "nextCursor": "next-page-cursor"
  }
}
```

#### Errors

An error response includes an `error` object with a code, message, and optional data:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32601,
    "message": "Method not found",
    "data": {
      "details": "The method 'unknown/method' is not supported"
    }
  }
}
```

Error codes follow the JSON-RPC 2.0 specification, with additional MCP-specific codes for specialized error conditions.

### Notifications

Notifications are one-way messages that don't expect a reply. They include:

- A method name that specifies the notification type
- Optional parameters with additional information
- No identifier field

For example, a notification about a resource change might look like:

```json
{
  "jsonrpc": "2.0",
  "method": "notifications/resources/changed",
  "params": {
    "uri": "file:///project/src/main.rs"
  }
}
```

Notifications are used for events like resource changes, capability updates, and other situations where no response is needed.

## 4.3 Protocol Layers

The Model Context Protocol consists of several key components that work together to create a comprehensive integration framework:

### Base Protocol

The foundation of MCP is the base protocol layer, which defines:

- Core JSON-RPC message types (requests, responses, notifications)
- Basic message structure and delivery semantics
- Error handling and reporting mechanisms

This layer provides the fundamental communication patterns that all MCP implementations must support.

### Lifecycle Management

Built on top of the base protocol is the lifecycle management layer, which handles:

- Connection initialization and capability negotiation
- Session establishment and maintenance
- Graceful connection termination
- Protocol version compatibility

This layer ensures that clients and servers can establish and maintain productive connections.

### Server Features

MCP defines several server-side features that provide the core functionality of the protocol:

- **Resources**: Data and content exposed by servers
- **Prompts**: Templates and instructions for language model interactions
- **Tools**: Executable functions that models can invoke

These features represent the primary ways that servers provide context and capabilities to clients.

### Client Features

Complementing the server features are client-side capabilities:

- **Sampling**: Allowing servers to request language model completions
- **Roots**: Defining filesystem locations that servers can access

These features enable servers to leverage client capabilities while maintaining appropriate security boundaries.

### Utilities

Cross-cutting concerns are addressed by utility components:

- Logging and diagnostics
- Argument completion and validation
- Progress reporting
- Operation cancellation

These utilities enhance the usability and robustness of MCP implementations.

All MCP implementations must support the base protocol and lifecycle management components. Other components can be implemented based on specific needs, with capability negotiation ensuring that clients and servers only attempt to use mutually supported features.

## 4.4 Authentication and Authorization

Authentication and authorization are critical considerations for any protocol that provides access to potentially sensitive data. MCP takes a flexible approach to these concerns, acknowledging that different implementations will have different security requirements.

### Current State

In the current MCP specification, authentication and authorization are not explicitly part of the core protocol. Instead, the specification notes:

> Authentication and authorization are not currently part of the core MCP specification, but we are considering ways to introduce them in future.

This approach allows implementations to adopt authentication mechanisms appropriate to their specific contexts while the community works toward standardized approaches.

### Implementation Options

While not standardized in the protocol, MCP implementations typically address authentication and authorization through several approaches:

1.  **Transport-Level Security**: Using secure transport mechanisms like TLS to ensure that connections are encrypted and authenticated.
2.  **Host-Managed Authorization**: The host application controls which servers can be connected and what permissions they have, often with explicit user consent.
3.  **Custom Authentication Headers**: Implementations can extend the protocol with custom authentication headers or parameters.
4.  **Environment-Based Authentication**: For local servers, authentication may leverage the operating system's security model.

### Best Practices

Although MCP doesn't mandate specific authentication mechanisms, several best practices have emerged:

1.  **Explicit User Consent**: Users should explicitly approve connections to MCP servers, understanding what data will be shared.
2.  **Principle of Least Privilege**: Servers should request and receive only the minimum permissions needed for their functionality.
3.  **Transparent Data Access**: Users should be able to see what data is being accessed by MCP servers.
4.  **Revocable Permissions**: Users should be able to revoke access permissions at any time.

These practices help ensure that MCP implementations maintain appropriate security boundaries while providing valuable functionality.

### Future Directions

The MCP community is actively discussing standardized approaches to authentication and authorization. Potential future additions to the protocol include:

1.  **Standardized Authentication Flow**: A defined process for servers to authenticate with clients.
2.  **Permission Scopes**: Standardized permission categories that servers can request.
3.  **Token-Based Authorization**: A token system for managing and validating access permissions.
4.  **Federated Identity Support**: Integration with existing identity providers and authentication systems.

As MCP continues to evolve, these security considerations will likely become more formalized within the protocol specification.

## 4.5 Error Handling and Logging

Robust error handling and logging are essential for creating reliable MCP implementations. The protocol defines structured approaches to both, helping developers create systems that gracefully handle failures and provide useful diagnostic information.

### Error Handling

MCP follows the JSON-RPC 2.0 error model, with standardized error codes and structures. Errors in MCP are represented as JSON objects with three required fields:

1.  **code**: A numeric error code that identifies the error type
2.  **message**: A short, human-readable description of the error
3.  **data** (optional): Additional information about the error

Error codes in MCP include:

- **Standard JSON-RPC errors** (e.g., -32700 for parse errors, -32601 for method not found)
- **MCP-specific errors** (e.g., capability negotiation failures, resource access errors)
- **Implementation-specific errors** (custom error codes for specific scenarios)

When an error occurs, the responding party returns an error object instead of a result:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32601,
    "message": "Method not found",
    "data": {
      "details": "The method 'unknown/method' is not supported"
    }
  }
}
```

Clients and servers must handle errors gracefully, providing appropriate feedback to users and attempting recovery when possible.

### Logging

MCP includes a logging facility that allows components to share diagnostic information. The logging system is designed to be:

1.  **Flexible**: Supporting different verbosity levels and filtering options
2.  **Structured**: Providing consistent log formats for easier analysis
3.  **Non-intrusive**: Avoiding performance impacts during normal operation

Logging in MCP is implemented through the `$/logTrace` notification, which components can use to share diagnostic information:

```json
{
  "jsonrpc": "2.0",
  "method": "$/logTrace",
  "params": {
    "message": "Resource not found: file:///missing.txt",
    "verbose": false
  }
}
```

The `verbose` parameter allows components to distinguish between essential messages and more detailed diagnostic information. Recipients can filter logs based on this parameter and other criteria.

### Best Practices

Several best practices have emerged for error handling and logging in MCP implementations:

1.  **Graceful Degradation**: When a feature isn't supported or an error occurs, implementations should continue functioning with reduced capabilities rather than failing completely.
2.  **Informative Error Messages**: Error messages should provide clear, actionable information to help users and developers understand and resolve issues.
3.  **Appropriate Log Levels**: Implementations should use appropriate verbosity levels for different types of log messages, avoiding unnecessary noise.
4.  **Error Recovery**: Where possible, implementations should attempt to recover from errors automatically, falling back to alternative approaches when primary methods fail.
5.  **User Feedback**: Critical errors should be surfaced to users in a clear, understandable way, with suggestions for resolution when applicable.

By following these practices, MCP implementations can provide robust, reliable experiences even when unexpected conditions arise.

[Previous Chapter](./chapter3_architecture.md)[Next Chapter](./chapter5_server_primitives.md)
