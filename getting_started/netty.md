# NioEventLoop
run方法调用processSelectedKeysOptimized或processSelectedKeysPlain，最终在processSelectedKey方法中调用unsafe处理CONNECT、READ和WRITE事件;
具体的逻辑实现参见unsafe.read(),unsafe.forceFlush()和unsafe.finishConnect();

* 读事件处理逻辑
   1. NioByteUnsafe.read
   
* 写事件处理逻辑
* 连接事件处理逻辑

# ServerBootstrapAcceptor
负责建立Socket连接


#ServerBootstrap启动流程
1.AbstractBootstrap.doBind方法：
2.ReflectiveChannelFactory的newInstance方法，通过反射，创建NioServerSocketChannel方法；
3.NioServerSocketChannel默认构造函数通过provider.openServerSocketChannel方法创建ServerSocketChannel
创建channelId=DefaultChannelId,由machineId+processId+序列号＋时间戳＋随机数生成
创建unsafe=NioMessageUnsafe
创建pipeline=DefaultChannelPipeline
配置channel为non-block,关注的selectionKey为SelectionKey.OP_ACCEPT
ServerBootstrap的init方法设置channel的option和attrs,pipeline中增加ChannelInitializer(inbound)
AbstractChannel的register方法，将channel注册到EventLoop,调用AbstractChannel的register0方法，


wrk

HeartBeatHandler->AllChannelHandler->DecodeHandler

客户端流程:
HeaderExchangeClient->HeaderExchangeChannel(组装Request和Future对象)->MinaClient(AbstractPeer.send->AbstractClient.send)->MinaChannel(调用IoSession写数据)->ProtocolCodecFilter->MinaCodecAdapter->ExchangeCodec


服务端流程:
ProtocolCodecFilter->MinaCodecAdapter->ExchangeCodec->MinaHandler(messageReceived)->MinaServer(AbstractPeer.received->)->
MultiMessageHandler->HeartbeatHandler->AllChannelHandler(组装任务对象，提交到线程池进行处理)->HeaderExchangeHandler->ExchangeHandlerDispatcher->RelierDispatcher



1.Fixed ResultMergerTest:MergerFactory.getMerger，return null when input is array;
2.Fixed RegistryDirectoryTest
3.Fixed DubboProtocolTest
4.Fixed ReferenceCountExchangeClientTest

