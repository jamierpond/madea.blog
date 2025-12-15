# What is yapi?
## Yapi is the API client that runs in your terminal.

> Yapi is the hacker's Postman, Insomnia or Bruno.

Yapi is an OSS command line tool that makes it easy to test APIs from your
terminal. Yapi speaks HTTP, gRPC, TCP, GraphQL (and more coming soon).

### Yapi speaks HTTP
I wanted a fun way to make HTTP requests from the terminal (without massive,
ad-hoc `curl` incantations).

### Elephant in the room, *why another API client?*
I built yapi for me to quickly test APIs from the without `curl` incantations,
and without%20having%20to%20manually%20encode%20query%20parameters%21.

I initially did this with a simple bash script that used `curl` & `jq`.

Then I was working on other projects using gRPC and TCP, and I saw that I could
support them with a little `grpcurl` and `netcat` magic.

It kinda kept growing, and I thought *"hey, maybe other people would find this useful?"*

#### POST
This request:

```yaml
# create-issue.yapi.yml
yapi: v1 # specify yapi version
method: POST # or GET, PUT, DELETE, PATCH, etc.
url: https://api.github.com/repos/jamierpond/yapi/issues
headers:
  Accept: application/vnd.github+json
  Authorization: Bearer ${GITHUB_PAT} # supports environment variables
body: # defaults to JSON body, converted automatically
  title: Help yapi made me too productive.
  body: Now I can't stop YAPPIN' about yapi!
expect: # supports expected response tests
  status: 201
  assert: # assert using jq syntax
    - .body == "Now I can't stop YAPPIN' about yapi!"
```

Gives you this response:

```json
yapi run create-issue.yapi.yml
{
  "active_lock_reason": null,
  "assignee": null,
  "assignees": [],
  "author_association": "OWNER",
  "body": "Now I can't stop YAPPIN' about yapi!\n",
  // ...blah blah blah
}

URL: https://api.github.com/repos/jamierpond/yapi/issues
Time: 579.408625ms
Size: 2.3 kB (1 lines, 2288 chars)

[PASS] status check
[PASS] .body == "Now I can't stop YAPPIN' about yapi!"
```
*(Only the JSON goes to stdout, the rest goes to stderr, so is pipeable!)*

You can also do PUT, PATCH, DELETE and any other HTTP method.

### Yapi supports chaining requests between protocols
#### Multi-protocol chaining
Yapi makes it easy to chain requests and share data between them, even if they are different protocols.
```yaml
# multi-protocol-chain.yapi.yml
yapi: v1
chain:
  - name: get_todo
    url: https://jsonplaceholder.typicode.com/todos/1
    method: GET

  - name: tcp_echo
    url: tcp://tcpbin.com:4242
    data: "Todo: ${get_todo.title}\n"
    encoding: text
    read_timeout: 5
    close_after_send: true

  - name: grpc_hello
    url: grpc://grpcb.in:9000
    service: hello.HelloService
    rpc: SayHello
    plaintext: true
    body:
      greeting: $get_todo.title

  - name: create_post
    url: https://jsonplaceholder.typicode.com/posts
    method: POST
    headers:
      Content-Type: application/json
    body:
      original_todo: $get_todo.title
      grpc_reply: $grpc_hello.reply
      userId: $get_todo.userId
    expect:
      status: 200
      assert:
        # run tests using jq assertions
        - .userId == $get_todo.userId
```

And gives you this response:
```json
yapi run multi-protocol-chain.yapi.yml
{
  "completed": false,
  "id": 1,
  "title": "delectus aut autem",
  "userId": 1
}
{
  "reply": "hello delectus aut autem"
}
{
  "grpc_reply": "hello delectus aut autem",
  "id": 101,
  "original_todo": "delectus aut autem",
  "userId": 1
}
```



