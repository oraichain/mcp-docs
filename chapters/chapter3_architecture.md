# Model Context Protocol

### Chapters

- [Chapter 1: Introduction](/chapters/chapter1_introduction.md)
- [Chapter 2: Fundamentals](/chapters/chapter2_fundamentals.md)
- [Chapter 3: Architecture](/chapters/chapter3_architecture.md)
- [Chapter 4: Base Protocol](/chapters/chapter4_base_protocol.md)
- [Chapter 5: Server Primitives](/chapters/chapter5_server_primitives.md)
- [Chapter 6: Client Primitives](/chapters/chapter6_client_primitives.md)
- [Chapter 7: Security](/chapters/chapter7_security.md)
- [Chapter 8: Implementation](/chapters/chapter8_implementation.md)
- [Chapter 9: Case Studies](/chapters/chapter9_case_studies.md)
- [Chapter 10: Future Directions](/chapters/chapter10_future_directions.md)

[← Back to Table of Contents](/)

# Chapter 3: MCP Architecture in Depth

## 3.1 Client-Host-Server Architecture

The Model Context Protocol (MCP) is built on a carefully designed client-host-server architecture that balances flexibility, security, and ease of implementation. This architecture enables AI systems to access diverse data sources while maintaining clear boundaries and responsibilities.

At its core, the MCP architecture follows a hub-and-spoke model:

- The **host** application sits at the center, coordinating all interactions
- Multiple **clients** act as spokes, each connecting to a specific server
- **Servers** provide specialized capabilities through a standardized interface

This design creates a clear separation of concerns while enabling rich, integrated experiences. Let's examine how these components interact in a typical MCP workflow:

1.  The host application creates client instances as needed
2.  Each client establishes a connection with a specific server
3.  The client and server negotiate capabilities during initialization
4.  The host coordinates data flow between clients and the AI model
5.  Each server provides context and capabilities within its domain
6.  The host aggregates context from multiple servers when appropriate

This architecture enables several key benefits:

- **Isolation**: Each server operates independently, with clear security boundaries
- **Composability**: Multiple servers can be combined to create rich experiences
- **Scalability**: New data sources can be added by implementing additional servers
- **Simplicity**: Each component has focused responsibilities, simplifying implementation

Consider a practical example: Claude Desktop (host) might connect to a file system server to access local documents, a GitHub server to access code repositories, and a database server to access structured data. Each server provides specialized capabilities within its domain, while Claude Desktop coordinates these connections and presents a unified interface to the user.

## 3.2 Design Principles of MCP

The Model Context Protocol is built on several key design principles that inform its architecture and implementation. Understanding these principles helps developers create effective MCP implementations that align with the protocol's goals.

### 1\. Servers Should Be Extremely Easy to Build

MCP is designed to minimize the implementation burden for server developers:

- Host applications handle complex orchestration responsibilities
- Servers focus on specific, well-defined capabilities
- Simple interfaces minimize implementation overhead
- Clear separation enables maintainable code

This principle encourages a thriving ecosystem of specialized servers. By making server implementation straightforward, MCP enables developers to quickly expose their data sources and tools to AI systems without needing to understand the complexities of AI integration.

### 2\. Servers Should Be Highly Composable

The power of MCP comes from combining multiple servers to create rich, integrated experiences:

- Each server provides focused functionality in isolation
- Multiple servers can be combined seamlessly
- Shared protocol enables interoperability
- Modular design supports extensibility

This composability allows users to build custom workflows that leverage multiple data sources and tools. For example, a developer might combine a file system server, a code analysis server, and a documentation server to create a comprehensive coding assistant.

### 3\. Servers Should Have Limited Visibility

MCP implements the principle of least privilege by limiting what servers can see:

- Servers receive only necessary contextual information
- Full conversation history stays with the host
- Each server connection maintains isolation
- Cross-server interactions are controlled by the host
- Host process enforces security boundaries

This principle protects user privacy and security by ensuring that servers only have access to the information they need to perform their specific functions. It also prevents potential security issues from spreading between servers.

### 4\. Features Can Be Added Progressively

MCP is designed to evolve over time while maintaining compatibility:

- Core protocol provides minimal required functionality
- Additional capabilities can be negotiated as needed
- Servers and clients evolve independently
- Protocol designed for future extensibility
- Backwards compatibility is maintained

This progressive approach allows the MCP ecosystem to grow and adapt to new requirements without breaking existing implementations. It also enables developers to start with simple implementations and add more sophisticated features as needed.

## 3.3 Security Boundaries and Isolation

Security is a fundamental consideration in the MCP architecture. The protocol implements several mechanisms to ensure that data remains protected and that servers operate within appropriate boundaries.

### Host-Enforced Security

The host application serves as the security gatekeeper for MCP interactions:

- Controls which servers can be connected
- Manages user consent and authorization
- Enforces data access policies
- Monitors server behavior
- Can terminate connections if necessary

This centralized security model ensures that users maintain control over their data and that servers cannot exceed their authorized access.

### Client Isolation

Each client maintains a 1:1 relationship with a specific server, creating clear isolation:

- Separate connection for each server
- Independent capability negotiation
- Isolated message routing
- Server-specific security boundaries

This isolation prevents servers from directly interacting with each other, reducing the risk of unauthorized data access or security breaches.

### Explicit Consent Requirements

MCP emphasizes user consent and control:

- Users must explicitly consent to server connections
- Data access requires appropriate authorization
- Tool invocations should be visible and controllable
- Sampling requests must be approved

These consent requirements ensure that users understand and control how their data is being used, building trust in the MCP ecosystem.

### Data Minimization

The protocol implements data minimization principles:

- Servers receive only the information they need
- Full conversation history remains with the host
- Servers declare their data requirements explicitly
- Hosts can filter sensitive information

By limiting the data shared with servers, MCP reduces the potential impact of security breaches and protects user privacy.

## 3.4 Message Flow and Communication Patterns

MCP uses a structured message flow based on JSON-RPC 2.0, enabling clear, bidirectional communication between clients and servers. Understanding these communication patterns is essential for implementing effective MCP components.

### Session Initialization

When a client connects to a server, they follow a structured initialization process:

1.  The client sends an initialization request with its capabilities
2.  The server responds with its supported capabilities
3.  Both parties store these capabilities for the session duration
4.  The session enters an active state with negotiated features

This capability-based negotiation ensures that clients and servers have a shared understanding of available features and can gracefully handle differences in implementation.

### Client-Initiated Requests

The most common communication pattern involves clients making requests to servers:

1.  The client sends a request with a unique ID and method name
2.  The server processes the request and performs any necessary operations
3.  The server sends a response with the same ID, containing either a result or an error
4.  The client processes the response and updates the host application

This pattern is used for operations like listing resources, reading resource contents, listing available tools, and retrieving prompts.

### Server Notifications

Servers can send one-way notifications to clients for events that don't require a response:

1.  The server sends a notification message with a method name but no ID
2.  The client processes the notification and updates the host application as needed

Notifications are used for events like resource changes, subscription updates, and capability changes.

### Sampling Requests

MCP supports a reverse flow where servers request language model completions from clients:

1.  The server sends a sampling request to the client
2.  The client obtains user consent if required
3.  The client forwards the request to the language model
4.  The language model generates a completion
5.  The client returns the completion to the server

This pattern enables servers to leverage the host's language model capabilities while maintaining appropriate security controls.

### Error Handling

MCP defines structured error handling for various failure scenarios:

1.  When an error occurs, the responding party includes an error object
2.  Error objects contain a code, message, and optional data
3.  Clients and servers must handle errors gracefully
4.  The protocol defines standard error codes for common scenarios

This structured approach to error handling improves reliability and helps developers diagnose and resolve issues.

## 3.5 Capability Negotiation

Capability negotiation is a cornerstone of the MCP architecture, enabling flexible and extensible interactions between clients and servers. This mechanism allows components to explicitly declare their supported features and adapt their behavior accordingly.

### Capability Declaration

During session initialization, both clients and servers declare their capabilities:

- Servers declare capabilities like resource subscriptions, tool support, and prompt templates
- Clients declare capabilities like sampling support and notification handling
- Capabilities can include optional features and parameters
- Both parties must respect declared capabilities throughout the session

This explicit declaration ensures that components only attempt to use features that are mutually supported, preventing compatibility issues.

### Progressive Enhancement

Capability negotiation enables progressive enhancement of MCP implementations:

- Basic functionality works with minimal capabilities
- Advanced features are enabled when mutually supported
- New capabilities can be added in future protocol versions
- Older clients and servers can still interoperate with newer ones

This approach allows the MCP ecosystem to evolve while maintaining backward compatibility, encouraging innovation without breaking existing implementations.

### Capability Examples

Here are some examples of capabilities that might be negotiated:

- **Resources**: Whether the server supports resource subscriptions and list change notifications
- **Tools**: Whether the server exposes executable tools and supports list change notifications
- **Prompts**: Whether the server provides prompt templates and supports list change notifications
- **Sampling**: Whether the client supports server-initiated language model sampling
- **Roots**: Whether the client provides information about filesystem roots

Each capability can include parameters that further define its behavior, allowing for fine-grained negotiation.

### Security Implications

Capability negotiation has important security implications:

- Capabilities define the boundaries of permitted operations
- Servers cannot use capabilities that clients haven't declared
- Clients cannot use capabilities that servers haven't declared
- Attempting to use unsupported capabilities results in errors

This mechanism helps enforce the principle of least privilege, ensuring that components only perform operations that have been explicitly authorized.

By implementing a robust capability negotiation system, MCP creates a flexible, extensible protocol that can adapt to diverse requirements while maintaining security and compatibility. This approach has proven successful in other protocols like the Language Server Protocol and contributes significantly to MCP's effectiveness as a universal connector for AI systems.

[Previous Chapter](/chapters/chapter2_fundamentals.md)[Next Chapter](/chapters/chapter4_base_protocol.md)
