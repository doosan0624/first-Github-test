# Netty 개념 정리


 * Netty란? 
   
   - Netty는 프로토콜 서버 및 클라이언트와 같은 네트워크 어플리케이션을 빠르고 쉽게 개발하는 것을 가능하게 해준다!
     - 그만큼 제공하는 api들이 편리하다는 것!

   - 유지하기 쉬운 높은 성능의 프로토콜 서버 및 클라이언트를 신속한 개발을 위한 "비동기" 네트워크 프레임워크이다! 
   - NIO 프레임워크. TCP 또는 UDP 소켓 서버와 같은 네트워크 프로그래밍을 간단하게 할 수 있다.  

 * 비동기이면서 이벤트 기반

   * tomcat < 10,000(connection) |  netty<100,000 ~ 1,000,000(connection) 
     
     - 숫자로만 봐도 Netty가 우세해보인다!
 
 * I/O 처리를 위한 추상화 API 제공
 
   * NIO, OIO, AIO 등의 처리에 대해 쉽게 바꿀수 있게 Netty I/O API 단에서 제공을 해준다.

 * 정의된 이벤트 모델과 스레드 모델 제공 
 
   * 접속을 맺고, 데이터 송수신, 접속을 끊을 때 각각 상황마다 어떤 이벤트가 발생할지 Netty가 정의하여 제공함
   * 사용자의 코드가 어느 스레드에서 언제 실행 될 지, 불필요한 동기화 방지... 등등 Netty가 정의하여 제공함.

 * 직접 구현한 버퍼풀을 이용한 성능 이점 제공 

   * Buffer Zero-file-copy, Buffer Automatic Capacity Extession 등등...


 ### Netty 핵심 컨포넌트 
 
   * Channel
   
      * 하나 이상의 입출력 작업(Read, Write)을 수행할 수 있는 하드웨어 장치/파일/네트워크 소켓/ 프로그램 컴포넌트와 같은 Entity에 대한 
        열린 연결, 들어오는 Inbound, 나가는 OutBound를 위한 운송수단으로 생각하면 될 듯 하다! 
        
   
   * CallBack 
   
      * 콜백은 관심 대상에게 작업 완료를 알리는 가장 일반적인 방법. 네티는 이벤트를 처리할 떄 내부적으로 콜백을 이용함! 
        콜백 트리거가 되면 ChannelHandler 인터페이스 구현을 통해 이벤트를 처리할 수 있다. 
         
   * Future 
    
      * 퓨처는 작업이 완료되면 어플리케이션에 이를 알리는 방법. 비동기 작업의 결과를 접근할 수 있게 해줌. 
        java.util.concurrent.Future 인터페이스를 제공하지만, 해당 구현은 작업 완료 여부 확인과 완료전까지 블록킹하는 기능만 존재.
        네티는 이를 개선한 ChannelFuture를 사용함. ChannelFuture에는 ChannelFutureListener 인스턴스를 하나 이상 등록 가능하며, 
        완료시점에 operationComplete() 콜백 메서드가 호출이 된다. 해당 콜백 실햄 시정에 완료/오류 등이 확인이 가능하다!
        네티의 모든 아웃바운드 입출력은 ChannelFuture를 반환하고 진행에 블록킹 작업 X. 모든것은 비동기 + 이벤트 기반. 
        
         * 최근 네티를 공부하며 정리해보았는데, 네트워크 프로그래밍에서 매우 좋은 프레임워크인 것 같다!

    
   * 이벤트와 핸들러

       * 네티는 작업 상태 변화를 알리기 위해 고유한 이벤트를 사용한다. 
    
          * 로깅, 데이터 변환, 흐름 제어, 어플리케이션 논리 등의 동작을 포함함..
       
       * 이벤트들은 크게 인바운드와 아웃바운드 데이터 흐름의 연관성을 기준으로 분류함.
   
       * 모든 이벤트는 핸들러 클래스의 사용자 구현 메서드로 전달 가능! 

          * 다시 말해, 각 핸들러 인스턴스는 특정 이벤트에 반응하여 실행하는 일종의 콜백이라고 이해하면 될 듯 하다! 

          * InBound : 연결 활성화/비활성화, 데이터 읽기, 사용자 이벤트, 오류 이벤트
          * OutBound : 원격 피어 연결 열기/닫기, 소켓에 데이터쓰기/플러시



   
   * 네티로 구현해본 예제코드!


     * EchoServer 
   
     ``` java
     public class EchoServer { 
        private final int port; 
        
        
        public EchoServer(int port) { 
            this.port = port; 
        }
            
        public static void main(String[] args) throws InterruptedException {
        
            if (args.length < 1) { 
                System.err.println("Usage: " + EchoServer.class.getSimpleName() + "<port>"); 
            } 
            int port = Integer.parseInt(args[0]); n
            ew EchoServer(port).start(); 
        } 
        
        public void start() throws InterruptedException { 
        
            final EchoServerHandler echoServerHandler = new EchoServerHandler(); 
            EventLoopGroup group = new NioEventLoopGroup(); 
            
            try {
                ServerBootstrap b = new ServerBootstrap(); 
                b.group(group) 
                    .channel(NioServerSocketChannel.class)
                    .localAddress(new InetSocketAddress(port)) 
                    .childHandler(new ChannelInitializer<SocketChannel>() { 
                        @Override 
                        protected void initChannel(SocketChannel ch) throws Exception { 
                            System.out.println("initChannel"); 
                            ch.pipeline().addLast(echoServerHandler); 
                        } 
                     }); 
                     
                 ChannelFuture future = b.bind().sync(); 
                 future.channel().closeFuture().sync(); 
              } finally { 
                  group.shutdownGracefully().sync(); 
              } 
          } 
     }
     
     
      ``` 
    
    
     * EchoServerHandler 

     ``` java
     @Sharable //여러 Channel에서 공유할 수 있음을 나타나는 마커 인터페이스
     public class EchoServerHandler extends ChannelInboundHandlerAdapter {
     @Override
     public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        final ByteBuf in = (ByteBuf) msg;
        System.out.println(
            "Server received: " + in.toString(CharsetUtil.UTF_8)
        };
        ctx.write(in);
     }
     
     @Overrid
     public void channelReadCOmplete(ChannelHandlerContext ctx) throws Exception {
        ctx.writeAndFlush(Unpooled.EMPTY_BUFFER)
             .addListener(ChannelFutureListener.CLOSE);
     }
     
     @Override
     public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close;
     }
     }
     ```
         
     * EchoClientHandler
     ``` java
     @Sharable
     public class EchoCLientHandler extends SimpleChannelInboundHandler<ByteBuf> {
        @Override
        public void channelActive(ChannelHandlerCOntext ctx) throws Exception {
            ctx.writeAndFlush(
            Unpooled.copiedBuffer( "Netty rocks!", CharsetUtil.UTF_8)
            );
        }
        
        @Override
        protected void channelRead0 (ChannelHandlerContext ctx, ByteBuf msg) throws Exception {
            System.out.println(
                    "Client received: " + msg.toString(CharsetUtil.UTF_8)
            );
        }
        
        @Override
        public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
            cause.printStackTrace();
            ctx.close();
        }
     }
     
     ```
     
     * EchoCLient
     
     ``` java
     public class EchoClient {
         private final String host;
         private final int port;
         
         public EchoClient(String host, int port) {
             this.port = port;
             this.host = host;
         }
         
         public void start() throws InterruptedException {
             EventLoopGroup group = new NioEventLoopGroup ();
             try {
                  Bootstrap b = new Bootstrap();
                  b.group(group)
                           .channel(NioSocketChannel.class)
                           .remoteAdderss(new InetSocketAddress(host, port))
                           .handler(new ChannelInitializer<SocketChannel>() {
                               @Override
                               protected void initChannel(SocketChannel ch) throws Exception {
                                   ch.pipeline()
                               .addLast(new EchoClientHandler());
                               }
                          });
                  ChannelFuture f = b.connect().sync();
                  f.channel().closeFuture().sync();
               } finally {
                   group.shutdownGracefully().sync();
               }
          }
           
          public static void main(String [] args) throws InterruptedException {
              if (args.length < 1) {
                  System.err.println(
                          "Usage : " +EchoClient.class.getSimpleName() + "<host> <port>"      
                  );
              }
              String host = args[0];
              int port = Integer.parseInt(args[1]);
              new EchoClient(host,port).start();
            }
     }
     
     ```
