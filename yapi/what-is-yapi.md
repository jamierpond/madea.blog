### What is yapi?

**yapi** is a unified, terminal-centric API client designed for developers who prefer "configuration-as-code." Instead of clicking through a GUI like Postman or Insomnia, yapi allows you to define API requests and testing workflows in simple **YAML** files.

It acts as a single executable CLI tool written in Go that can execute these files, handle complex workflows, and provide rich terminal output. It bridges the gap between simple `curl` scripts and full UI-based clients by supporting multiple protocols and stateful request chaining.

-----

### Core Features

#### 1\. Multi-Protocol Support

Yapi is not limited to standard REST APIs. It uses a unified configuration structure to support diverse transport protocols:

  * **HTTP/HTTPS:** Full support for methods, headers, query params, and JSON bodies.
  * **gRPC:** First-class support using `grpc://` URLs. You can specify the service, RPC method, and proto files (or use server reflection).
  * **GraphQL:** Native support for queries and variables.
  * **TCP:** Send raw data (text, hex, or base64) directly to TCP sockets.

#### 2\. Request Chaining (Workflows)

One of yapi's most powerful features is **Chaining**. You can define a sequence of dependent requests where steps can reference data from previous steps.

  * **Example:** A "Login" step gets a token, and the next step uses that token in the Authorization header.
  * **Variable Extraction:** You can reference values like `${login.body.token}` or `${login.headers.Set-Cookie}`.

#### 3\. Built-in Testing & Assertions

Yapi includes a lightweight testing framework via the `expect` block. You can validate responses without writing external scripts:

  * **Status Checks:** Assert specific HTTP or gRPC status codes.
  * **JQ Assertions:** Use `jq` boolean expressions to validate the response body (e.g., `body.data.id != null`).

#### 4\. Developer Experience (DX) & TUI

Yapi is designed for a modern terminal workflow:

  * **Watch Mode (`yapi watch`):** A TUI (Terminal User Interface) that watches your config file and re-runs the request instantly when you save changes.
  * **Interactive Runner:** Running `yapi run` without arguments opens a fuzzy-search file picker to select a config file.
  * **Syntax Highlighting:** Response bodies (JSON, HTML) are automatically syntax-highlighted.

#### 5\. Utilities & Integration

  * **JQ Filtering:** You can filter response output directly in the config using the `jq_filter` key.
  * **LSP Support:** Yapi includes a built-in Language Server (`yapi lsp`) that provides autocompletion and validation for `.yapi.yml` files in your IDE.
  * **Sharing:** The `yapi share` command compresses your configuration and generates a link to share it via the web.

-----

### Example Configuration

Here is what a typical **yapi** configuration looks like (supporting both a simple request and assertions):

```yaml
# my-request.yapi.yml
yapi: v1
url: https://api.example.com/users
method: POST
headers:
  Authorization: Bearer ${MY_ENV_VAR}
json: |
  {
    "name": "Alice",
    "role": "admin"
  }
expect:
  status: 201
  assert:
    - .id != null
    - .role == "admin"
```
