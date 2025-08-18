---
title: copilot-extension
toc: true
tags:
  - ai
  - copilot
date: 2025-08-18 19:27:57
---


# Abstract

We can implement copilot extensions so that it can use copilot llm abilities and do special things for us. There are several ways to implement the extensions:

1. **client side extensions**: Now only vscode support to add copilot extensions. It's running locally in your machine, and thus use your local resource and can access all your local workspace and utilize your local installed tools. **Ideal for tasks deeply embedded in developer workflows, like editing, real-time code generation, or using local context**

2. **server side extensions**: it's executed on your backend server. Since it's a backend server, it can be run on any clients (e.g. vscode, vs, github mobile app...). Therefore you can have single implementation runs on any client where copilot is supported. However it can't acccess your local workspace (e.g. you stashed changes, your local working files, etc.). 

3. **MCP (model context protocol) server**: As a protocol that enables automatic tool invocation based on intent, MCP is more flexible and powerful. It can work both client side and server side. A key distinction between Copilot Extensions and MCP is their activation model: Extensions require explicit invocation, while MCP servers are automatically called by Copilot based on the user’s intent. This “invisible” integration makes MCP particularly promising for the future of AI-assisted development.

# Abstract (copilot)



### Overview

GitHub introduced **Copilot Extensions** in public preview on September 17, 2024. These allow developers to enhance Copilot with customizable extensions right within their preferred environments—such as IDEs (like VS Code), GitHub.com, or the GitHub mobile app—letting them work with databases, testing tools, deployment systems, and internal knowledge without context switching ([Tiago Pascoal](https://pascoal.net/2024/10/22/gh-copilot-extensions/?utm_source=chatgpt.com "GitHub Copilot Extensions - Tiago Pascoal")).

---

### Extension Types: Client-Side vs Server-Side

#### **Client-Side Extensions**

- Run locally within IDEs (currently only supported in VS Code) ([Tiago Pascoal](https://pascoal.net/2024/10/22/gh-copilot-extensions/?utm_source=chatgpt.com "GitHub Copilot Extensions - Tiago Pascoal")).

- Strengths:
  
  - Full access to the local workspace (including uncommitted changes).
  
  - Integrates with other installed tools, terminal outputs, machine state, or other extensions.
  
  - Leverages Copilot’s Gen AI seamlessly—no need for manual authentication or API handling.
  
  - Accessible via Copilot Chat, command palette, or editor context menu ([Tiago Pascoal](https://pascoal.net/2024/10/22/gh-copilot-extensions/?utm_source=chatgpt.com "GitHub Copilot Extensions - Tiago Pascoal")).

- Ideal for tasks deeply embedded in developer workflows, like editing, real-time code generation, or using local context ([Tiago Pascoal](https://pascoal.net/2024/10/22/gh-copilot-extensions/?utm_source=chatgpt.com "GitHub Copilot Extensions - Tiago Pascoal")).

#### **Server-Side Extensions**

- Run on your own backend; accessible across all Copilot-supported clients including GitHub.com and mobile apps ([Tiago Pascoal](https://pascoal.net/2024/10/22/gh-copilot-extensions/?utm_source=chatgpt.com "GitHub Copilot Extensions - Tiago Pascoal")).

- Strengths:
  
  - Unified implementation works across IDEs, web, and mobile.
  
  - Can access internal resources securely (e.g., databases, GPUs).
  
  - Ideal when local workspace access isn't needed or for centralized resource control ([Tiago Pascoal](https://pascoal.net/2024/10/22/gh-copilot-extensions/?utm_source=chatgpt.com "GitHub Copilot Extensions - Tiago Pascoal")).

- Limitations:
  
  - Cannot access local files or uncommitted state.
  
  - Does not support client-side references.
  
  - Requires infrastructure to host and run the extension ([Tiago Pascoal](https://pascoal.net/2024/10/22/gh-copilot-extensions/?utm_source=chatgpt.com "GitHub Copilot Extensions - Tiago Pascoal")).

A comparison table in the post summarizes these pros and cons in detail ([Tiago Pascoal](https://pascoal.net/2024/10/22/gh-copilot-extensions/?utm_source=chatgpt.com "GitHub Copilot Extensions - Tiago Pascoal")).

---

### Model Context Protocol (MCP)

- **What is MCP?**  
  A specification originally developed by Anthropic for Claude AI, enabling AI agents to access external data sources (e.g., file systems, databases, third-party tools) via standard interfaces ([Tiago Pascoal](https://pascoal.net/2024/10/22/gh-copilot-extensions/?utm_source=chatgpt.com "GitHub Copilot Extensions - Tiago Pascoal")).

- **Transport Types Supported:**
  
  - Local execution via standard input/output.
  
  - Remote hosting via HTTP + SSE (Server-Sent Events), with recent support for streamable HTTP that upgrades to SSE ([Tiago Pascoal](https://pascoal.net/2024/10/22/gh-copilot-extensions/?utm_source=chatgpt.com "GitHub Copilot Extensions - Tiago Pascoal")).

- **Capabilities Exposed by MCP Servers:**
  
  - Resources (data access)
  
  - Prompts (pre-configured queries)
  
  - Tools (executable actions)
  
  - Sampling (data generation) ([Tiago Pascoal](https://pascoal.net/2024/10/22/gh-copilot-extensions/?utm_source=chatgpt.com "GitHub Copilot Extensions - Tiago Pascoal"))

- **Recent Developments:**  
  On April 4, 2025, VS Code added MCP server support (preview), and GitHub open-sourced its own MCP server ([Tiago Pascoal](https://pascoal.net/2024/10/22/gh-copilot-extensions/?utm_source=chatgpt.com "GitHub Copilot Extensions - Tiago Pascoal")).

- **Key Difference from Extensions:**  
  While Copilot Extensions require explicit user invocation, MCP servers can be **automatically called by Copilot** when the user’s intent matches—making the toolchain more seamless and "invisible" ([Tiago Pascoal](https://pascoal.net/2024/10/22/gh-copilot-extensions/?utm_source=chatgpt.com "GitHub Copilot Extensions - Tiago Pascoal")).

---

### Examples & Ecosystem

#### **Client-Side Examples:**

- **copilot-pg**: Queries PostgreSQL databases via Copilot.

- **vscode-mssql-chat**: Uses Copilot Chat to query SQL Server within VS Code.

- **Data Analysis for Copilot**: Runs Python locally for computations and data analysis.

- **MongoDB for VS Code**: Enables natural-language interactions with MongoDB clusters via Copilot Chat ([Tiago Pascoal](https://pascoal.net/2024/10/22/gh-copilot-extensions/?utm_source=chatgpt.com "GitHub Copilot Extensions - Tiago Pascoal")).

#### **Server-Side Examples:**

- Launching a Docker deployment from GitHub.com Chat.

- Extensions that interact with backend systems or GitHub APIs for unified cross-client experiences ([Tiago Pascoal](https://pascoal.net/2024/10/22/gh-copilot-extensions/?utm_source=chatgpt.com "GitHub Copilot Extensions - Tiago Pascoal")).

- You can discover extensions on GitHub Marketplace or VS Code Marketplace ([Tiago Pascoal](https://pascoal.net/2024/10/22/gh-copilot-extensions/?utm_source=chatgpt.com "GitHub Copilot Extensions - Tiago Pascoal")).

---

### Key Takeaways

1. **Copilot as a Platform**  
   Copilot is now extendable—developers can create custom tools that integrate AI capabilities, their own data, and internal policies—all within the environments they already use ([Tiago Pascoal](https://pascoal.net/2024/10/22/gh-copilot-extensions/?utm_source=chatgpt.com "GitHub Copilot Extensions - Tiago Pascoal")).

2. **Two Flexible Paths**
   
   - *Client-side* for local, context-rich workflows.
   
   - *Server-side* for unified, centrally managed experiences.

3. **MCP Enhances Transparency**  
   As a protocol that enables automatic tool invocation based on intent, MCP represents a powerful, emerging layer for intelligent, context-aware AI tooling ([Tiago Pascoal](https://pascoal.net/2024/10/22/gh-copilot-extensions/?utm_source=chatgpt.com "GitHub Copilot Extensions - Tiago Pascoal")).

4. **Rising Ecosystem**  
   A growing library of example extensions and increasing Marketplace visibility shows mounting momentum behind Copilot’s extensibility ([Tiago Pascoal](https://pascoal.net/2024/10/22/gh-copilot-extensions/?utm_source=chatgpt.com "GitHub Copilot Extensions - Tiago Pascoal")).

---

Let me know if you'd like me to deep-dive into how to **build a client-side VS Code extension** or work with **MCP integration**—happy to help craft implementation guidelines or examples!





# References

[github copilot extension mechnism overview | Tiago Pascoal](https://pascoal.net/2024/10/22/gh-copilot-extensions/)
