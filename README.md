# Titan: Lessons learned

### Titan is dead. You want [JanusGraph](https://github.com/sunsided/janusgraph-docker) instead.

Sources for the `Dockerfile` and surroundings:

* [efaurie/docker-titan-cassandra](https://github.com/efaurie/docker-titan-cassandra)
* [elubow/titan-gremlin](https://github.com/elubow/titan-gremlin)

For multiple graphs in Titan, follow these links:

* [How many graphs can i create in one Titan DB?](https://stackoverflow.com/a/40545537/195651)
* [Serving multiple Titan graphs over Gremlin Server (TinkerPop 3)](https://medium.com/@jbmusso/serving-multiple-titan-graphs-over-gremlin-server-tinkerpop-3-d3c971d07964)
* [One graph in one Titan instance](https://jaceklaskowski.gitbooks.io/titan-scala/content/one_graph_in_one_titan_instance.html)

## Cassandra and Elasticsearch

As per [compatibility matrix](http://s3.thinkaurelius.com/docs/titan/1.0.0/version-compat.html), the supported Cassandra version is 2.1 and the supported Elasticsearch version is 1.5.

## Shell

In the Gremin REPL examples like this one:

```
$  bin/gremlin.sh
         \,,,/
         (o o)
-----oOOo-(3)-oOOo-----
plugin activated: tinkerpop.server
plugin activated: tinkerpop.hadoop
plugin activated: tinkerpop.utilities
plugin activated: aurelius.titan
plugin activated: tinkerpop.tinkergraph
gremlin> :remote connect tinkerpop.server conf/remote.yaml
==>Connected - localhost/127.0.0.1:8182
gremlin> :> graph.addVertex("name", "stephen")
==>v[256]
gremlin> :> g.V().values('name')
==>stephen
```

The token `:>` is not part of the _shell_, but an actual _command_. It is required to run the command on the remote server.
Also, entering invalid commands in the shell results in an exception on the server.

## Channelizers

Using the `HttpChannelizer` 

```
channelizer: org.apache.tinkerpop.gremlin.server.channel.HttpChannelizer
```

allows for HTTP access to Titan using e.g.

```bash
curl "http://localhost:8182/?gremlin=100-1"
```

However, this seems to prevent the Gremlin REPL shell from talking to the server:

```
Jul 17, 2017 10:52:00 PM java.util.prefs.FileSystemPreferences$1 run
INFO: Created user preferences directory.

         \,,,/
         (o o)
-----oOOo-(3)-oOOo-----
plugin activated: aurelius.titan
plugin activated: tinkerpop.server
plugin activated: tinkerpop.utilities
plugin activated: tinkerpop.hadoop
plugin activated: tinkerpop.tinkergraph
gremlin> :remote connect tinkerpop.server conf/remote.yaml
22:52:14 WARN  org.apache.tinkerpop.gremlin.driver.handler.WebSocketClientHandler  - Exception caught during WebSocket processing - closing connection
io.netty.handler.codec.http.websocketx.WebSocketHandshakeException: Invalid handshake response getStatus: 400 Bad Request
	at io.netty.handler.codec.http.websocketx.WebSocketClientHandshaker13.verify(WebSocketClientHandshaker13.java:182)
	at io.netty.handler.codec.http.websocketx.WebSocketClientHandshaker.finishHandshake(WebSocketClientHandshaker.java:202)
	at org.apache.tinkerpop.gremlin.driver.handler.WebSocketClientHandler.channelRead0(WebSocketClientHandler.java:73)
	at io.netty.channel.SimpleChannelInboundHandler.channelRead(SimpleChannelInboundHandler.java:105)
	at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:308)
	at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:294)
	at io.netty.handler.codec.MessageToMessageDecoder.channelRead(MessageToMessageDecoder.java:103)
	at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:308)
	at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:294)
	at io.netty.handler.codec.ByteToMessageDecoder.channelInactive(ByteToMessageDecoder.java:241)
	at io.netty.handler.codec.http.HttpClientCodec$Decoder.channelInactive(HttpClientCodec.java:212)
	at io.netty.channel.CombinedChannelDuplexHandler.channelInactive(CombinedChannelDuplexHandler.java:132)
	at io.netty.channel.AbstractChannelHandlerContext.invokeChannelInactive(AbstractChannelHandlerContext.java:208)
	at io.netty.channel.AbstractChannelHandlerContext.fireChannelInactive(AbstractChannelHandlerContext.java:194)
	at io.netty.channel.DefaultChannelPipeline.fireChannelInactive(DefaultChannelPipeline.java:828)
	at io.netty.channel.AbstractChannel$AbstractUnsafe$5.run(AbstractChannel.java:576)
	at io.netty.util.concurrent.SingleThreadEventExecutor.runAllTasks(SingleThreadEventExecutor.java:380)
	at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:357)
	at io.netty.util.concurrent.SingleThreadEventExecutor$2.run(SingleThreadEventExecutor.java:116)
	at java.lang.Thread.run(Thread.java:748)
22:52:14 ERROR org.apache.tinkerpop.gremlin.driver.Handler$GremlinResponseHandler  - Could not process the response - correct the problem and restart the driver.

```

The REPL shell does seem to work with the `WebSocketChannelizer` though:

```
channelizer: org.apache.tinkerpop.gremlin.server.channel.WebSocketChannelizer
```


