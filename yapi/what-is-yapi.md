# (draft) what is yapi?
## yapi is the api client that runs in your terminal.

> yapi is the hacker's postman, insomnia or bruno.

yapi is an oss command line tool that makes it easy to test apis from your terminal. yapi speaks http, grpc, tcp, graphql (and more coming soon).


## Firstly, why is yapi?

### yapi speaks http
i wanted a fun way to make http requests from the terminal (without massive, ad-hoc `curl` incantations).
#### get
this request:
```yaml
# search.yapi.yml
yapi: v1
method: get
url: https://api.github.com/search/repositories
headers:
  authorization: bearer ${github_pat} # reads from your environment
query:
  q: yapi in:name, jamierpond in:owner
jq_filter: |
    .items[] | {
      name: .name,
      stars: .stargazers_count,
      url: .html_url
    }
```

gives you this response:
```json
yapi run search.yapi.yml
{
  "name": "yapi",
  "stars": 5, // at time of writing!
  "url": "https://github.com/jamierpond/yapi"
}
{
  "name": "yapi-blog",
  "stars": 0,
  "url": "https://github.com/jamierpond/yapi-blog"
}
{
  "name": "homebrew-yapi",
  "stars": 0,
  "url": "https://github.com/jamierpond/homebrew-yapi"
}
```

#### post
this request:
```yaml
# create-issue.yapi.yml
yapi: v1
method: post
url: https://api.github.com/repos/jamierpond/yapi/issues
headers:
  accept: application/vnd.github+json
  authorization: bearer ${github_pat}
body:
  title: help yapi made me too productive.
  body: |
    now i can't stop yappin' about yapi!
```
gives you this response:
```json
yapi run create-issue.yapi.yml
{
  "active_lock_reason": null,
  "assignee": null,
  "assignees": [],
  "author_association": "owner",
  "body": "now i can't stop yappin' about yapi!\n",
  "closed_at": null,
  "closed_by": null,
  "comments": 0,
  // ...blah blah blah
}
```
you can also do put, patch, delete and any other http method.

### yapi supports chaining requests between protocols
#### multi-protocol chaining
yapi makes it easy to chain requests and share data between them, even if they are different protocols.
```yaml
# multi-protocol-chain.yapi.yml
yapi: v1
chain:
  - name: get_todo
    url: https://jsonplaceholder.typicode.com/todos/1
    method: get

  - name: tcp_echo
    url: tcp://tcpbin.com:4242
    data: "todo: ${get_todo.title}\n"
    encoding: text
    read_timeout: 5
    close_after_send: true

  - name: grpc_hello
    url: grpc://grpcb.in:9000
    service: hello.helloservice
    rpc: sayhello
    plaintext: true
    body:
      greeting: $get_todo.title

  - name: create_post
    url: https://jsonplaceholder.typicode.com/posts
    method: post
    headers:
      content-type: application/json
    body:
      original_todo: $get_todo.title
      grpc_reply: $grpc_hello.reply
      userid: $get_todo.userid
    expect:
      status: 200
      assert:
        # run tests using jq assertions
        - .userid == $get_todo.userid
```

and gives you this response:
```json
yapi run multi-protocol-chain.yapi.yml
{
  "completed": false,
  "id": 1,
  "title": "delectus aut autem",
  "userid": 1
}
{
  "reply": "hello delectus aut autem"
}
{
  "grpc_reply": "hello delectus aut autem",
  "id": 101,
  "original_todo": "delectus aut autem",
  "userid": 1
}
```



