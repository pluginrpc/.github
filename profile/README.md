PluginRPC is an Protobuf RPC framework for plugins. It enables writing plugins defined by Protobuf
services while only relying on CLI primitives such as args, stdin, and stdout. PluginRPC has no
reliance on network calls. Individual RPCs are invoked via arguments passed to the CLI plugin,
requests are sent on stdin, and responses are sent on stdout. It's everything you need to use
Protobuf services to represent your plugin API without any of the cruft.

PluginRPC has the following goals:

- Make it possible to evolve both the PluginRPC protocol and the APIs for individual plugins in
  backward-compatible ways over time.
- Make it possible to invoke multiple methods as part of a plugin. Many plugin frameworks only
  provide for a single entrypoint. In contrast, PluginRPC provides the full power of Protobuf
  services: multiple services, and multiple RPCs per service, can be exposed.
- Expose the plugin interface in CLI-idiomatic ways. PluginRPC exposes individual RPCs as
  sub-commands with flags for control, optionally allowing the specific sub-commands to be
  customizable. PluginRPC attempts to consume and produce requests and responses in a manner that
  will be readable by humans invoking the sub-commands.
- Make it easy to call plugins even in the absence of an official language-specific library.

PluginRPC currently has one official language library:

- [pluginrpc-go](https://github.com/pluginrpc/pluginrpc-go).

Go is a natural language to write plugins in, and we have a direct use case for an RPC library
in Go for our custom lint and breaking change plugins in [buf](https://github.com/bufbuild/buf).
However, PluginRPC is purposefully designed to be simple to implement in any language. If there is
sufficient demand, we may provide official implementations for other languages in the future.

If using Protobuf to write plugins, there's traditionally been two mechanisms:

1. Have a single request message type sent over stdin, and return a single response message type
   over stdout. This is how
   [`protoc`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/compiler/plugin.proto)
   operates, for example, taking in a `CodeGeneratorRequest` and sending back a
   `CodeGeneratorRequest` over stdout. This is very simple, however it makes doing anything else
   extremely difficult. All plugin API evolution needs to remain within these single messages for
   all time, and providing functionality for multiple methods is awkward at best (for example, via
   use of `oneofs`). Multiple content types cannot be supported.
2. Bring a full-powered network RPC framework into the mix. This is how
   [go-plugin](https://github.com/hashicorp/go-plugin) operates, for example, making a plugin expose
   a gRPC service, and then doing an exec/kill dance to bring the plugin up temporarily and expose
   it on a port, call the required method, and then kill the plugin entirely. While effective in
   allowing lots of flexibility, it's bringing in a very complicated framework to solve a simple
   engineering problem, adding brittleness, requiring network calls, and not allowing plugins to
   remain idiomatic CLI tools

PluginRPC provides the best of both worlds: simple CLI constructs with all the power you require. If
you want to use Protobuf services across the network, use [ConnectRPC](https://connectrpc.com). If
you want to use Protobuf services to write plugins, use PluginRPC.
