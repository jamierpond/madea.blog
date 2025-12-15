# Introducing Yapi: The API Client Of Your Dreams

> Yapi is the hacker's Postman, Insomnia or Bruno.

Yapi is an OSS command line tool that makes it easy to test APIs from your
terminal. Yapi speaks HTTP, gRPC, TCP, GraphQL (and more coming soon).

![yapi in action](https://github.com/jamierpond/madea.blog/blob/main/yapi/yapi-example.gif?raw=true)

### Elephant in the room, *why another API client?*
I built yapi for me to quickly test APIs from the without `curl` incantations,
and without%20having%20to%20manually%20encode%20query%20parameters%21.

I initially did this with a simple bash script that used `curl` & `jq`.

Then I was working on other projects using gRPC and TCP, and I saw that I could
support them with a little `grpcurl` and `netcat` magic.

Eventually I realised that I was building a new tool, and that other people
might find it useful too, so I rewrote it in Go, added more features and
released it as yapi.

### Show me some examples!
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
  title: Help! yapi made me too productive.
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
  - name: get_todo # HTTP request
    url: https://jsonplaceholder.typicode.com/todos/1
    method: GET
  - name: grpc_hello # gRPC request
    url: grpc://grpcb.in:9000
    service: hello.HelloService
    rpc: SayHello # Supports reflection if server has it enabled
    plaintext: true
    body:
      greeting: $get_todo.title # use data from previous request
    expect:
      assert:
        - .reply == "hello delectus aut autem" # assert on gRPC response
```
[Run this example in the yapi playground](https://yapi.run/c/Df.8OgAzqUG-ZpArHJoiQovBOB1XLQM8BNSbz655jxnoC_d1kKWwWp3qeqyLUO_M73lKIwrUWJ6prOLMl5R2.qS8r1QzMvktH~6BGCJjH3E6CO6.iI_fuN4z.kAT3W70t~of01maCruwdBfYgS_nxmtesmpiqKbjalxSWIKMGIRbdL_u8WwRc89gKXPB0kW5wy_CGDmIF0Kef.vuRpV7Hm3BjZie2~yI6DHtvb-RgOqEzldl0W~Y.m4RTNw_Oy.OlksIrAkhwiIy6gdyVqPk0tgTzOePDzuo.trOigUj1AtfBNDpowrEdIU3MmT53pjaobcZV73FYwwpjWOfdNCEJqKTTUVrCc8ZgVB5riViWnJxqo5yz759FWyiBFbi~_2nLHnu67V7RLQPUYnFknAovF2SegQYEmmI~vrbJjjmdNShJNaVW)

### Yapi supports wriring intgration tests with expectations and assertion.
Yapi has built-in support for writing integration tests with expectations and assertions.
Which makes it easy to write regresssion tests and ensure your APIs are working as expected.

```yaml
yapi: v1
method: GET
url: https://api.github.com/repos/jamierpond/yapi/issues/5
headers:
    Accept: application/vnd.github+json
expect:
    status: 200
    assert:
      - .state == "closed"
      - .closed_by.login == "jamierpond"
```
[Run this example in the yapi playground](https://yapi.run/c/TccVrmpVtt_daI~NZovhqJO6LNVxr-qTEAL9aQp.nAvs9GoUK8PdGgJ6o34wC-THPf64WS.-iAWCfXohQ8~eo~7iOhp6Wop-a_lwPT.Y624K-1S97CVJgXwMRxtwUzZz6on7xftwVegIkX~6hO81OIvTsA6ASUgWdnf8va0QlaGHLI5w1d3LZ5WQ4xzYDg8~5lqPB5s3FE~~EL8aBeCKk-FtOPJ1AIPDMNH4X55A8QLYUF3NWY)

