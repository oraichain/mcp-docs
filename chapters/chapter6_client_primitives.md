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

[← Back to Table of Contents](../)

# Chapter 6: Client-side Primitives

## 6.1 Overview of Client Features

While server-side primitives form the foundation of MCP's data provision capabilities, client-side primitives enable servers to leverage the capabilities of the host application and language model. These client features create a bidirectional relationship, allowing servers to not only provide context but also request services from the client.

MCP defines two primary client-side primitives:

1.  **Sampling**: Allowing servers to request language model completions
2.  **Roots**: Defining filesystem locations that servers can access

These primitives complement the server-side features, creating a comprehensive framework for AI integration. Each primitive addresses a specific need in the MCP ecosystem, with clear protocols for negotiation and use.

By implementing these primitives, clients enable more sophisticated server behaviors while maintaining appropriate security boundaries and user control. Let's explore each of these primitives in detail, examining their purpose, implementation, and best practices.

## 6.2 Sampling: Leveraging Language Models

### 6.2.1 Sampling Definition and Purpose

Sampling in MCP refers to the ability of servers to request completions from the language model hosted by the client. This capability allows servers to leverage the AI's reasoning and generation capabilities for their own operations, creating more intelligent and adaptive server behaviors.

The key characteristics of MCP sampling include:

- **Server-Initiated**: Sampling requests originate from servers based on their operational needs.
- **Client Control**: The client maintains control over whether and how sampling requests are fulfilled.
- **Structured Format**: Sampling follows a consistent message format similar to standard LLM APIs.
- **Security Boundaries**: Sampling respects security boundaries, with clients mediating access to the language model.
- **User Consent**: Sampling typically requires user consent, especially for sensitive operations.

Sampling serves several important purposes in the MCP ecosystem:

1.  **Enhanced Server Intelligence**: It enables servers to make more intelligent decisions based on language model reasoning.
2.  **Dynamic Content Generation**: It allows servers to generate content tailored to specific contexts and needs.
3.  **Complex Processing**: It supports sophisticated text processing that would be difficult to implement without AI capabilities.
4.  **Adaptive Behavior**: It enables servers to adapt their behavior based on natural language understanding of user needs.

For example, a code analysis server might use sampling to generate explanations of complex code patterns, leveraging the language model's understanding of programming concepts and natural language.

### 6.2.2 Protocol Messages for Sampling

MCP defines several protocol messages for working with sampling:

#### Sampling Request

To request a language model completion, servers send a `sampling/sample` request:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "sampling/sample",
  "params": {
    "messages": [
      {
        "role": "system",
        "content": {
          "type": "text",
          "text": "You are a helpful assistant that explains code."
        }
      },
      {
        "role": "user",
        "content": {
          "type": "text",
          "text": "Explain this function: def fibonacci(n): return n if n <= 1 else fibonacci(n-1) + fibonacci(n-2)"
        }
      }
    ]
  }
}
```

The client processes this request, potentially obtaining user consent, and responds with the language model's completion:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "message": {
      "role": "assistant",
      "content": {
        "type": "text",
        "text": "This function implements a recursive calculation of the Fibonacci sequence. The Fibonacci sequence is a series of numbers where each number is the sum of the two preceding ones, usually starting with 0 and 1.\n\nHere's how the function works:\n\n1. It takes a parameter `n` which represents the position in the Fibonacci sequence.\n2. If `n` is 0 or 1, it returns `n` directly (base cases).\n3. For any other value of `n`, it recursively calls itself for `n-1` and `n-2`, and returns their sum.\n\nThis implementation is elegant but inefficient for larger values of `n` due to the repeated calculations in the recursive calls."
      }
    }
  }
}
```

### 6.2.3 Security Considerations

Sampling introduces important security considerations that must be carefully managed:

1.  **User Consent**: Users should explicitly approve sampling requests, understanding what information is being processed by the language model.
2.  **Data Privacy**: Sampling requests may contain sensitive information that should be protected according to appropriate privacy policies.
3.  **Rate Limiting**: Clients should implement rate limiting to prevent abuse of sampling capabilities.
4.  **Content Filtering**: Clients may need to filter or moderate sampling requests and responses to prevent misuse.
5.  **Transparency**: Users should be aware when servers are leveraging language model capabilities through sampling.

The MCP specification emphasizes these security considerations, recommending that applications implement appropriate controls and transparency mechanisms.

### 6.2.4 Implementation Best Practices

When implementing sampling support in MCP clients, several best practices can enhance security, usability, and effectiveness:

1.  **Clear User Interfaces**: Provide clear interfaces for user consent, helping users understand what sampling requests are being made and why.
2.  **Appropriate Rate Limits**: Implement rate limits that balance server needs with resource constraints and security considerations.
3.  **Caching Strategies**: Consider caching strategies for common sampling requests to improve performance and reduce redundant processing.
4.  **Error Handling**: Implement robust error handling for sampling failures, providing informative error messages to servers.
5.  **Logging and Auditing**: Maintain appropriate logs of sampling requests for security monitoring and troubleshooting.
6.  **Performance Optimization**: Optimize sampling request handling to minimize latency and resource usage.
7.  **Content Policies**: Establish clear policies for acceptable sampling content and implement appropriate enforcement mechanisms.

By following these practices, developers can create sampling implementations that enhance server capabilities while maintaining security, performance, and user trust.

## 6.3 Roots: Filesystem Access Control

### 6.3.1 Roots Definition and Purpose

Roots in MCP define filesystem locations that servers can access, providing a structured approach to filesystem integration while maintaining security boundaries. This capability allows servers to work with local files and directories within controlled, user-approved locations.

The key characteristics of MCP roots include:

- **Client Declaration**: Roots are declared by clients during capability negotiation.
- **URI-Based Identification**: Each root is identified by a URI scheme and path.
- **Access Control**: Roots define the boundaries of filesystem access for servers.
- **User Approval**: Root access typically requires explicit user approval.
- **Scheme Support**: Roots support various URI schemes, including `file://` for local filesystem access.

Roots serve several important purposes in the MCP ecosystem:

1.  **Controlled Access**: They provide controlled access to filesystem resources without exposing the entire filesystem.
2.  **Security Boundaries**: They establish clear boundaries for server filesystem operations.
3.  **User Control**: They enable users to explicitly approve which filesystem locations servers can access.
4.  **Integration Flexibility**: They support various integration scenarios, from local development to cloud-based workflows.

For example, a development environment might define roots for project directories, allowing code analysis servers to access source files without exposing unrelated system files.

### 6.3.2 Protocol Messages for Roots

MCP defines protocol messages for working with roots as part of the initialization process:

#### Client Initialization

During initialization, clients declare supported roots in their capabilities:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "capabilities": {
      "roots": {
        "schemes": ["file"],
        "paths": [
          {
            "scheme": "file",
            "path": "/home/user/projects"
          }
        ]
      }
    }
  }
}
```

This declaration informs servers about the filesystem locations they can access, allowing them to operate within these boundaries.

### 6.3.3 Security Considerations

Roots introduce important security considerations that must be carefully managed:

1.  **Scope Limitation**: Roots should be defined with the minimum necessary scope, limiting server access to only required locations.
2.  **User Consent**: Users should explicitly approve root access, understanding which filesystem locations servers can access.
3.  **Path Traversal Prevention**: Implementations must prevent path traversal attacks that attempt to access files outside defined roots.
4.  **Permission Enforcement**: Access controls within root directories should be enforced according to user permissions.
5.  **Sensitive File Protection**: Special consideration should be given to protecting sensitive files even within approved root directories.

The MCP specification emphasizes these security considerations, recommending that applications implement appropriate controls and validation mechanisms.

### 6.3.4 Implementation Best Practices

When implementing roots support in MCP clients, several best practices can enhance security, usability, and effectiveness:

1.  **Clear User Interfaces**: Provide clear interfaces for root approval, helping users understand what filesystem access is being granted and why.
2.  **Path Normalization**: Implement robust path normalization to prevent path traversal attacks and ensure consistent access control.
3.  **Granular Permissions**: Consider implementing granular permissions within roots, such as read-only vs. read-write access.
4.  **Dynamic Root Management**: Support dynamic addition and removal of roots based on user needs and workflow changes.
5.  **Root Visualization**: Provide visual indicators of active roots, helping users understand current filesystem access boundaries.
6.  **Validation Mechanisms**: Implement thorough validation of all filesystem operations against defined roots.
7.  **Audit Logging**: Maintain logs of filesystem access for security monitoring and troubleshooting.

By following these practices, developers can create roots implementations that enable powerful filesystem integration while maintaining security and user control.

## 6.4 Client-Server Interaction Patterns

The client-side primitives in MCP enable sophisticated interaction patterns between clients and servers, creating bidirectional relationships that enhance the capabilities of both components. Understanding these patterns helps developers create more effective MCP implementations.

### 6.4.1 Complementary Capabilities

Client and server primitives are designed to complement each other, creating a balanced ecosystem where each component contributes its strengths:

- **Servers provide context** through resources, tools, and prompts
- **Clients provide intelligence** through sampling and controlled access

This complementary relationship enables powerful workflows that leverage both data access and AI capabilities, creating experiences that neither component could provide alone.

### 6.4.2 Request Flows

MCP supports several request flow patterns that enable different types of interactions:

#### Client-Initiated Flows

In client-initiated flows, the client requests information or services from the server:

1.  Client sends a request (e.g., `resources/list`, `tools/list`)
2.  Server processes the request and performs necessary operations
3.  Server sends a response with results or error information
4.  Client incorporates the response into its operations

This pattern is used for operations like discovering available resources, reading resource contents, and listing available tools.

#### Server-Initiated Flows

In server-initiated flows, the server requests services from the client:

1.  Server sends a request (e.g., `sampling/sample`)
2.  Client processes the request, potentially obtaining user consent
3.  Client performs the requested operation (e.g., generating a language model completion)
4.  Client sends a response with results or error information
5.  Server incorporates the response into its operations

This pattern is used for operations like sampling, where servers leverage client capabilities to enhance their functionality.

#### Notification Flows

In notification flows, components send one-way messages without expecting responses:

1.  Sender generates a notification (e.g., `notifications/resources/list_changed`)
2.  Receiver processes the notification and updates its state accordingly

This pattern is used for events like resource changes, capability updates, and other situations where no response is needed.

### 6.4.3 Orchestration Patterns

MCP enables several orchestration patterns that combine client and server capabilities:

#### Context Enhancement

In this pattern, servers provide context that enhances language model capabilities:

1.  Client requests relevant resources from servers
2.  Servers provide resource contents
3.  Client incorporates these resources into language model context
4.  Language model generates more informed, contextually relevant responses

This pattern is commonly used for knowledge-intensive tasks where external information improves model performance.

#### Tool-Augmented Reasoning

In this pattern, language models use server-provided tools to extend their capabilities:

1.  Client discovers available tools from servers
2.  Client exposes these tools to the language model
3.  Language model decides when to use tools based on user queries
4.  Client facilitates tool execution through server requests
5.  Language model incorporates tool results into its reasoning

This pattern enables models to perform actions and access information beyond their built-in capabilities.

#### Collaborative Processing

In this pattern, clients and servers collaborate on complex tasks:

1.  Client receives user query
2.  Client requests relevant context from servers
3.  Servers process the query using sampling to leverage language model capabilities
4.  Servers return enhanced results incorporating both data access and AI reasoning
5.  Client presents unified results to the user

This pattern combines the strengths of both components to deliver more sophisticated functionality than either could provide alone.

### 6.4.4 Implementation Considerations

When implementing client-server interactions in MCP, several considerations can improve effectiveness:

1.  **Appropriate Pattern Selection**: Choose interaction patterns that match the specific requirements of your use case.
2.  **Performance Optimization**: Consider the performance implications of different patterns, especially for latency-sensitive operations.
3.  **Error Handling**: Implement robust error handling for all interaction patterns, ensuring graceful degradation when components fail.
4.  **Security Boundaries**: Maintain appropriate security boundaries in all interaction patterns, especially when handling sensitive data.
5.  **User Experience**: Design interactions with user experience in mind, minimizing latency and providing appropriate feedback.
6.  **Resource Efficiency**: Optimize resource usage in interaction patterns, avoiding unnecessary requests and processing.
7.  **Scalability**: Consider the scalability implications of different patterns, especially for high-volume or distributed deployments.

By carefully designing client-server interactions, developers can create MCP implementations that leverage the full power of both components while maintaining security, performance, and usability.

[Previous Chapter](./chapter5_server_primitives.md)[Next Chapter](./chapter7_security.md)
