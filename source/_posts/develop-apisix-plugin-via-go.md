---
title: Develop APISIX plugin via Golang
date: 2022-03-08 8:18:15
tags: System Architecture
---

# Develop APISIX plugin via Golang

APISIX allows users to extend functions through plugins, Although the Apache APISIX kernel code is written in Lua by the plugin, Apache APISIX supports the development of plugins in multiple languages, such as Go and Java. This article will explain in detail how to use plugins to develop Apache APISIX.
We will write an HTTP base auth plugin in Golang

## write the plugin
### Prerequisites
* Go (>= 1.15)
* apisix-go-plugin-runner

Let's look at the Plugin interface, To write a custom plugin, we need to implement the plugin's interface  
```
type Plugin interface {
 // Name returns the plguin name
 Name() string

 // ParseConf is the method to parse the given plugin configuration. When the
 // configuration can't be parsed, it will be skipped.
 ParseConf(in []byte) (conf interface{}, err error)

 // Filter is the method to handle request.
 // It is like the `http.ServeHTTP`, plus the ctx and the configuration created by
 // ParseConf.
 //
 // When the `w` is written, the execution of the plugin chain will be stopped.
 // We don't use onion model like Gin/Caddy because we don't serve the whole request lifecycle
 // inside the runner. The plugin is only a filter running at one stage.
 Filter(conf interface{}, w http.ResponseWriter, r pkgHTTP.Request)
}
```
<!--more-->

- Name returns the plugin's name
- ParseConfig will accept a bytes array and we can define our plugin settings
- Filter that's the place we handle HTTP request and response


Let's start writing a plugin by Golang, in this example, we'll write a basic-auth plugin, Basic Authentication is a method for an HTTP user agent (e.g., a web browser) to provide a username and password when making a request. When employing Basic Authentication, users include an encoded string in the Authorization header of each request they make. 

The Authorization header follows this format:
`Authorization: Basic <credentials>`
We then construct the credentials like this:
- The user’s username and password are combined with a colon.
- The resulting string is base64 encoded.

 we'll implement this plugin in the following ways.

-  The plugins' name will be `go-plugin-basic-auth`
-  Define the plugin's config by username/password
-  Handler HTTP request from Filter function, try to decode the Authorization and compare with the config, if the username/password matches, let the request go through (do nothing) otherwise return 401


Here's the code

```
type BasicAuthPlugin struct {
}

type BasicAuthConfig struct {
        Username string `json:"username"`
        Password string `json:"password"`
}

func (b *BasicAuthPlugin) Name() string {
        return "go-plugin-basic-auth"
}

func (b *BasicAuthPlugin) ParseConf(in []byte) (interface{}, error) {
        conf := BasicAuthConfig{}
        err := json.Unmarshal(in, &conf)
        return conf, err
}

func (b *BasicAuthPlugin) Filter(conf interface{}, w http.ResponseWriter, r pkgHTTP.Request) {
        baseAuthConfig := conf.(BasicAuthConfig)

        username, password, ok := basicAuth(r)
        if !ok {
                w.Header().Set("WWW-Authenticate", `Basic realm="restricted", charset="UTF-8"`)
                w.WriteHeader(http.StatusUnauthorized)
                w.Write([]byte(`No basic auth present`))
                return
        }

        if username != baseAuthConfig.Username || password != baseAuthConfig.Password {
                w.Header().Set("WWW-Authenticate", `Basic realm="restricted", charset="UTF-8"`)
                w.WriteHeader(http.StatusUnauthorized)
                w.Write([]byte(`Username/Password is not correct`))
                return
        }
}
```

And then register the plugin in the main function
```
func main() {
        cfg := runner.RunnerConfig{}
        cfg.LogLevel = zapcore.DebugLevel
        err := plugin.RegisterPlugin(&plugins.BasicAuthPlugin{})
        if err != nil {
                log.Fatalf("failed to register plugin BasicAuthPlugin: %s", err)
        }
        runner.Run(cfg)
}
```

Run it 
```
APISIX_LISTEN_ADDRESS=unix:/tmp/runner.sock go run main.go 
```
After it starts, we'll see logs like this 
```
2022-03-07T19:43:41.952+0800    INFO    plugin/plugin.go:66     register plugin go-plugin-basic-auth
2022-03-07T19:43:41.952+0800    WARN    server/server.go:177    conf cache ttl is 1h0m0s
2022-03-07T19:43:41.952+0800    WARN    server/server.go:185    listening to /tmp/runner.sock
```

Now we can get into the next part



## Integrations with APISIX

### Run APISIX
The easy way to start apisix is to use docker, Here's the example docker-compose file
```
version: '3'
services:

  apisix:
    image: apache/apisix:2.12.1-alpine
    volumes:
        - /tmp/runner.sock:/tmp/runner.sock
        - ./apisix_log:/usr/local/apisix/logs
        - ./apisix_conf/config.yaml:/usr/local/apisix/conf/config.yaml:ro
    depends_on:
        - etcd
    ports:
        - "9080:9080/tcp"
        - "9091:9091/tcp"
        - "9443:9443/tcp"
        - "9092:9092/tcp"
    networks:
      apisix:

  etcd:
    image: bitnami/etcd:3.4.15
    environment:
        ETCD_ENABLE_V2: "true"
        ALLOW_NONE_AUTHENTICATION: "yes"
        ETCD_ADVERTISE_CLIENT_URLS: "http://0.0.0.0:2379"
        ETCD_LISTEN_CLIENT_URLS: "http://0.0.0.0:2379"
    volumes:
      - etcd_data:/bitnami/etcd
    networks:
      apisix:

networks:
  apisix:
    driver: bridge

volumes:
  etcd_data:
    driver: local
```

and here's the config file `config.yaml`
```
apisix:
  node_listen: 9080              # APISIX listening port
  enable_ipv6: false

  allow_admin:                  # http://nginx.org/en/docs/http/ngx_http_access_module.html#allow
    - 0.0.0.0/0              # We need to restrict ip access rules for security. 0.0.0.0/0 is for test.

  admin_key:
    - name: "admin"
      key: edd1c9f034335f136f87ad84b625c8f1
      role: admin                 # admin: manage all configuration data
                                  # viewer: only can view configuration data
    - name: "viewer"
      key: 4054f7cf07e344346cd3f287985e76a2
      role: viewer

  enable_control: true
  control:
    ip: "0.0.0.0"
    port: 9092

etcd:
  host:                           # it's possible to define multiple etcd hosts addresses of the same etcd cluster.
    - "http://etcd:2379"     # multiple etcd address
  prefix: "/apisix"               # apisix configurations prefix
  timeout: 30                     # 30 seconds

ext-plugin:
  path_for_test: /tmp/runner.sock  #plugin's sock address
```

we define the plugin's listening sock in `path_for_test`, That
means after hitting a routing rule, APISIX will make an RPC request to /`tmp/runner.sock`.

run `docker-compose up` to start apisix, use commands below to see if it works 
`
curl "http://127.0.0.1:9080/apisix/admin/services/" -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1'
`

The following data is returned to indicate that Apache APISIX was successfully started:
`
{
  "count":1,
  "action":"get",
  "node":{
    "key":"/apisix/services",
    "nodes":{},
    "dir":true
  }
}
`


## Test the plugin

![runner-overview](/images/runner-overview.png)
The diagram above shows the workflow of APISIX on the left, while the plugin runner on the right is responsible for running external plugins written in different languages. apisix-go-plugin-runner is one such runner that supports Go.
When you configure a plugin runner in APISIX, APISIX will treat the plugin runner as a child process of its own. This sub-process belongs to the same user as the APISIX process. When we restart or reload APISIX, the plugin runner will also be restarted.

### create test routes with upstream and enable the plugin

```
curl --location --request PUT 'http://127.0.0.1:9080/apisix/admin/routes/1' \
--header 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' \
--header 'Content-Type: application/json' \
--data-raw '{
    "uri": "/get",
    "plugins": {
        "ext-plugin-pre-req": {
            "conf": [
                {
                    "name": "go-plugin-basic-auth",
                    "value": "{\"username\":\"foo\",\"password\":\"bar\"}"
                }
            ]
        }
    },
    "upstream": {
        "type": "roundrobin",
        "nodes": {
            "httpbin.org": 1
        }
    }
}'
```

In this route, we set the following config
- define router in `/get`, APISIX will pass the request to its upstream  httpbin.org
- enable ext-plugin-pre-req plugin, add `go-plugin-basic-auth` and its config with the `value`.  username foo and password as bar
There are two kinds of plugins 
ext-plugin-pre-req: before executing most of the APISIX built-in plugins (Lua language plugins)
ext-plugin-post-req: after the execution of the APISIX built-in plugins (Lua language plugins)

we use ext-plugin-pre-req for the base auth plugin because we need to break the request when the Authorization fails, and in the plugin config, we pass username and password AS JSON string which matches The Golang code `BasicAuthConf` 's structure. 

Now let's go to http://127.0.0.1:9080/get, the web browser will ask us to input username and password,  fill the form with foo and bar then we'll see the response from httpbin.org

Let's it, develop Golang plugin in APISIX is easy 

## summary
At present, go plugin runner is still in the early development stage, and we will gradually improve its functions. A successful open source project is inseparable from your contributions. Welcome to participate in the development of the APISIX Go plugin runner. Let’s build a bridge between Apache APISIX and Go!

### Full code repo
github.com:ezioruan/apisix_go_plugin_example

### Related reading
https://apisix.apache.org/docs/go-plugin-runner/getting-started/

