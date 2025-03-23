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

# Chapter 5: Server-side Primitives

## 5.1 Overview of Server Features

Server-side primitives form the foundation of MCP's functionality, providing the essential building blocks that enable AI models to interact with external data and systems. These primitives represent different ways that servers can expose capabilities to clients, each with its own control model and use cases.

MCP defines three primary server-side primitives:

1.  **Prompts**: User-controlled templates that guide language model interactions
2.  **Resources**: Application-controlled data that provides context to models
3.  **Tools**: Model-controlled functions that enable actions and data retrieval

These primitives work together to create a comprehensive framework for AI integration. Each primitive addresses a different aspect of context provision and interaction, with a distinct control model that determines how it's used:

Primitive

Control

Description

Example

Prompts

User-controlled

Interactive templates invoked by user choice

Slash commands, menu options

Resources

Application-controlled

Contextual data attached and managed by the client

File contents, git history

Tools

Model-controlled

Functions exposed to the LLM to take actions

API POST requests, file writing

By implementing these primitives, servers can provide rich, interactive experiences that leverage the full capabilities of language models while maintaining appropriate security boundaries and user control.

Let's explore each of these primitives in detail, examining their purpose, implementation, and best practices.

## 5.2 Prompts: User-Controlled Templates

### 5.2.1 Prompt Definition and Purpose

Prompts in MCP are pre-defined templates or instructions that guide language model interactions. They provide structured messages and workflows that help users accomplish specific tasks with language models.

The key characteristics of MCP prompts include:

- **User Control**: Prompts are designed to be explicitly selected by users, typically through UI elements like slash commands or menu options.
- **Parameterization**: Prompts can accept arguments that customize their behavior for specific use cases.
- **Structured Format**: Prompts follow a consistent format that includes metadata and message content.
- **Discoverability**: Clients can list available prompts, helping users discover relevant capabilities.

Prompts serve several important purposes in the MCP ecosystem:

1.  **Standardization**: They provide consistent ways to accomplish common tasks.
2.  **Expertise Encoding**: They capture expert knowledge about effective ways to use language models.
3.  **Workflow Simplification**: They reduce complex multi-step processes to simple, parameterized commands.
4.  **Domain Specialization**: They enable domain-specific interactions tailored to particular use cases.

For example, a code review prompt might provide a structured template for analyzing code quality, accepting a code snippet as an argument and generating a comprehensive review with suggestions for improvement.

### 5.2.2 Protocol Messages for Prompts

MCP defines several protocol messages for working with prompts:

#### Listing Prompts

To discover available prompts, clients send a `prompts/list` request:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "prompts/list",
  "params": {
    "cursor": "optional-cursor-value"
  }
}
```

The server responds with a list of available prompts, including their names, descriptions, and required arguments:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "prompts": [
      {
        "name": "code_review",
        "description": "Asks the LLM to analyze code quality and suggest improvements",
        "arguments": [
          {
            "name": "code",
            "description": "The code to review",
            "required": true
          }
        ]
      }
    ],
    "nextCursor": "next-page-cursor"
  }
}
```

This operation supports pagination through the optional `cursor` parameter, allowing clients to retrieve large prompt lists in manageable chunks.

#### Getting a Prompt

To retrieve a specific prompt with arguments, clients send a `prompts/get` request:

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "prompts/get",
  "params": {
    "name": "code_review",
    "arguments": {
      "code": "def hello():\n    print('world')"
    }
  }
}
```

The server responds with the prompt content, including messages that can be sent to the language model:

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "description": "Code review prompt",
    "messages": [
      {
        "role": "user",
        "content": {
          "type": "text",
          "text": "Please review this Python code:\ndef hello():\n    print('world')"
        }
      }
    ]
  }
}
```

#### List Changed Notification

When the list of available prompts changes, servers that declared the `listChanged` capability send a notification:

```json
{
  "jsonrpc": "2.0",
  "method": "notifications/prompts/list_changed"
}
```

This notification allows clients to refresh their prompt lists when new prompts become available or existing ones are modified.

### 5.2.3 Implementation Best Practices

When implementing prompt support in MCP servers, several best practices can enhance the user experience and ensure effective integration:

1.  **Clear Descriptions**: Provide concise, informative descriptions for each prompt, helping users understand its purpose and appropriate use cases.
2.  **Meaningful Argument Names**: Use descriptive names for prompt arguments, with clear descriptions of their purpose and expected format.
3.  **Reasonable Defaults**: Where appropriate, provide default values for optional arguments to simplify common use cases.
4.  **Consistent Naming**: Adopt a consistent naming convention for prompts, making them easier to discover and understand.
5.  **Appropriate Granularity**: Design prompts with appropriate scope—neither too narrow (requiring many similar prompts) nor too broad (requiring complex argument structures).
6.  **User-Friendly Presentation**: Consider how prompts will be presented in user interfaces, optimizing for discoverability and ease of use.
7.  **Versioning Strategy**: Plan for prompt evolution, considering how to handle updates without disrupting existing workflows.
8.  **Documentation**: Provide comprehensive documentation for complex prompts, including examples and usage guidelines.

By following these practices, developers can create prompt implementations that enhance the user experience and leverage the full power of language models in a structured, accessible way.

## 5.3 Resources: Application-Controlled Data

### 5.3.1 Resource Definition and Purpose

Resources in MCP represent structured data or content that provides additional context to language models. They allow servers to share information from various sources, such as files, databases, or application-specific data stores.

The key characteristics of MCP resources include:

- **Application Control**: Resources are managed by the client application, which decides how and when to use them based on application logic and user needs.
- **Unique Identification**: Each resource is uniquely identified by a URI (Uniform Resource Identifier).
- **Structured Content**: Resources have defined content types and structures, often represented using MIME types.
- **Discoverability**: Clients can list available resources and explore their organization.
- **Change Notification**: Servers can notify clients when resources change, enabling real-time updates.

Resources serve several important purposes in the MCP ecosystem:

1.  **Context Provision**: They provide relevant information that helps language models generate more accurate and helpful responses.
2.  **Data Integration**: They connect language models to various data sources, expanding their knowledge beyond training data.
3.  **Real-time Updates**: They enable access to current information that may not be available in the model's training data.
4.  **Specialized Knowledge**: They provide domain-specific information that enhances the model's capabilities in particular areas.

For example, a file system resource might provide access to local code files, enabling a language model to reference and understand the user's codebase when answering programming questions.

### 5.3.2 Protocol Messages for Resources

MCP defines several protocol messages for working with resources:

#### Listing Resources

To discover available resources, clients send a `resources/list` request:

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

The server responds with a list of available resources, including their URIs, names, descriptions, and MIME types:

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

This operation supports pagination through the optional `cursor` parameter, allowing clients to retrieve large resource lists in manageable chunks.

#### Reading Resources

To retrieve resource contents, clients send a `resources/read` request:

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "resources/read",
  "params": {
    "uri": "file:///project/src/main.rs"
  }
}
```

The server responds with the resource content:

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "contents": [
      {
        "uri": "file:///project/src/main.rs",
        "mimeType": "text/x-rust",
        "text": "fn main() {\n    println!(\"Hello world!\");\n}"
      }
    ]
  }
}
```

#### List Changed Notification

When the list of available resources changes, servers that declared the `listChanged` capability send a notification:

```json
{
  "jsonrpc": "2.0",
  "method": "notifications/resources/list_changed"
}
```

This notification allows clients to refresh their resource lists when new resources become available or existing ones are modified.

### 5.3.3 Resource Templates

Resource templates provide a powerful way to expose parameterized resources using URI templates. They allow servers to define patterns for accessing resources dynamically, based on client-provided parameters.

To list available resource templates, clients send a `resources/templates/list` request:

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "resources/templates/list"
}
```

The server responds with a list of available templates:

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "result": {
    "resourceTemplates": [
      {
        "uriTemplate": "file:///{path}",
        "name": "Project Files",
        "description": "Access files in the project directory",
        "mimeType": "application/octet-stream"
      }
    ]
  }
}
```

Clients can then use these templates to construct specific resource URIs by replacing the template parameters with actual values.

### 5.3.4 Implementation Best Practices

When implementing resource support in MCP servers, several best practices can enhance the user experience and ensure effective integration:

1.  **Efficient Resource Listing**: Implement pagination for resource listings to handle large resource collections efficiently.
2.  **Appropriate Granularity**: Design resource hierarchies with appropriate granularity, balancing specificity with manageability.
3.  **Meaningful URIs**: Use clear, consistent URI schemes that reflect the resource organization and make resources easily identifiable.
4.  **Content Type Accuracy**: Provide accurate MIME types for resources to enable appropriate handling and presentation.
5.  **Change Notifications**: Implement change notifications to keep clients updated about resource modifications.
6.  **Resource Caching**: Consider caching strategies to improve performance for frequently accessed resources.
7.  **Error Handling**: Provide clear error messages when resources cannot be accessed, helping users understand and resolve issues.
8.  **Security Boundaries**: Respect security boundaries and access controls when exposing resources.

By following these practices, developers can create resource implementations that provide rich context to language models while maintaining performance, security, and usability.

## 5.4 Tools: Model-Controlled Functions

### 5.4.1 Tool Definition and Purpose

Tools in MCP represent executable functions that language models can invoke to perform actions or retrieve information. They enable models to interact with external systems, extending their capabilities beyond text generation.

The key characteristics of MCP tools include:

- **Model Control**: Tools are designed to be discovered and invoked by language models based on their understanding of user needs and the current context.
- **Structured Inputs**: Tools have defined input schemas that specify the parameters they accept.
- **Structured Outputs**: Tools return results in a consistent format that models can interpret and incorporate into their responses.
- **Discoverability**: Models can discover available tools through tool listings and descriptions.
- **Security Controls**: Tool execution typically requires user consent, especially for operations with side effects.

Tools serve several important purposes in the MCP ecosystem:

1.  **Action Execution**: They allow models to perform actions in external systems, such as creating resources or updating data.
2.  **Information Retrieval**: They enable models to access real-time information that may not be available in their training data.
3.  **Computation**: They support specialized calculations or processing that would be difficult for models to perform through text generation alone.
4.  **System Integration**: They connect models to existing systems and services, leveraging established functionality.

For example, a weather tool might allow a model to retrieve current weather information for a specific location, providing up-to-date data that wouldn't be available in its training corpus.

### 5.4.2 Protocol Messages for Tools

MCP defines several protocol messages for working with tools:

#### Listing Tools

To discover available tools, clients send a `tools/list` request:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/list",
  "params": {
    "cursor": "optional-cursor-value"
  }
}
```

The server responds with a list of available tools, including their names, descriptions, and input schemas:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "tools": [
      {
        "name": "get_weather",
        "description": "Get current weather information for a location",
        "inputSchema": {
          "type": "object",
          "properties": {
            "location": {
              "type": "string",
              "description": "City name or zip code"
            }
          },
          "required": ["location"]
        }
      }
    ],
    "nextCursor": "next-page-cursor"
  }
}
```

This operation supports pagination through the optional `cursor` parameter, allowing clients to retrieve large tool lists in manageable chunks.

#### Calling Tools

To invoke a tool, clients send a `tools/call` request:

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/call",
  "params": {
    "name": "get_weather",
    "arguments": {
      "location": "New York"
    }
  }
}
```

The server executes the tool and responds with the result:

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Current weather in New York:\nTemperature: 72°F\nConditions: Partly cloudy"
      }
    ],
    "isError": false
  }
}
```

If the tool execution fails, the server indicates this through the `isError` flag and provides error information in the content.

#### List Changed Notification

When the list of available tools changes, servers that declared the `listChanged` capability send a notification:

```json
{
  "jsonrpc": "2.0",
  "method": "notifications/tools/list_changed"
}
```

This notification allows clients to refresh their tool lists when new tools become available or existing ones are modified.

### 5.4.3 Security Considerations

Tools represent a powerful capability that requires careful security consideration. Since tools can execute code and modify systems, they introduce potential risks that must be managed:

1.  **User Consent**: Users should explicitly approve tool invocations, especially for tools with side effects or access to sensitive data.
2.  **Clear Visibility**: Tool invocations should be clearly visible to users, with transparent indication of what actions are being performed.
3.  **Input Validation**: Tool implementations should validate inputs to prevent injection attacks or other security issues.
4.  **Least Privilege**: Tools should operate with the minimum permissions necessary for their functionality.
5.  **Rate Limiting**: Implementations should consider rate limiting to prevent abuse or resource exhaustion.
6.  **Audit Logging**: Tool invocations should be logged for security monitoring and troubleshooting.

The MCP specification emphasizes these security considerations, recommending that applications:

- Provide UI that makes clear which tools are being exposed to the AI model
- Insert clear visual indicators when tools are invoked
- Present confirmation prompts to the user for operations, to ensure a human is in the loop

### 5.4.4 Implementation Best Practices

When implementing tool support in MCP servers, several best practices can enhance security, usability, and effectiveness:

1.  **Clear Descriptions**: Provide concise, informative descriptions for each tool, helping models understand when and how to use them.
2.  **Precise Input Schemas**: Define input schemas that clearly specify required parameters and their formats, enabling accurate tool invocation.
3.  **Informative Outputs**: Structure tool outputs to be easily interpretable by models, with clear indications of success or failure.
4.  **Appropriate Granularity**: Design tools with appropriate scope—neither too narrow (requiring many similar tools) nor too broad (requiring complex argument structures).
5.  **Error Handling**: Implement robust error handling with informative error messages that help diagnose and resolve issues.
6.  **Performance Considerations**: Optimize tool execution for performance, especially for tools that may be frequently invoked.
7.  **Documentation**: Provide comprehensive documentation for complex tools, including examples and usage guidelines.
8.  **Testing**: Thoroughly test tools with various inputs, including edge cases and potential error conditions.

By following these practices, developers can create tool implementations that extend the capabilities of language models while maintaining security, usability, and reliability.

[Previous Chapter](/chapters/chapter4_base_protocol.md)[Next Chapter](/chapters/chapter6_client_primitives.md)
