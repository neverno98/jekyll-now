---
layout: post
title: RabbitMq Thread Instance 침범 문제
---

### RabbitMq 를 이용하여 비동기 이벤트 처리를 할 때, 발생했던 문제 해결 과정을 공유한다.  

#### 상황
  * 처리량이 많은 이벤트가 자꾸 사라지는 문제가 발생하였다.
  * 초기엔 Log 를 줄여놓은 상태라, Log 를 추가하여 살펴본 결과 어느 순간 갑자기 event 가 사라진다. 
    > 하지만 시스템에 Error/Exception은 발생하지 않고, 다음 event 는 계속 처리가 되고 있었다.
  * 내부 처리의 안정성을 위해 autoAck = false / qos = 30    
  * autoAck 를 사용하지 않기 때문에, 메시지 유실은 없지만 후추 재발송 하는데 시간이 소요되었다.  
    
#### 원인
  * RabbitMq 는 Consumer 에서 Handler 발식으로 메시지를 발송한다.
     <pre><code>consumer = new DefaultConsumer(channel) {

        @Override
        public void handleDelivery(
            String consumerTag, 
            Envelope envelope, 
            AMQP.BasicProperties properties, 
            byte[] body) throws IOException {
      
            handler.handleDelivery(channel, envelope, properties, body);
        }
    };</code></pre>

  * 이를 Handler 에서 연결하여 처리 하도록 해두었다.
     <pre><code>public interface MqHandler {
        
        void handleDelivery(
            Channel channel, 
            Envelope envelope, 
            AMQP.BasicProperties properties, 
            byte[] body);
    }</code></pre>     

  * 처리량이 많을 경우 Thread 를 생성해서 보내주게 되는데 처리를 할때 같은 객체에서 처리를 할 경우에, 하나의 Thread 가 멈추고 나중에 들어온 메시지가 처리 될 경우에 인스턴스에 멤버를 새로이 넣어서 문제가 발생하는 경우가 있었다.
     <pre><code>private SmsSendFacade smsSendFacade = new SmsSendFacade();
    public void handleDelivery(
        Channel channel, 
        Envelope envelope, 
        AMQP.BasicProperties properties, 
        byte[] body) {
     
        smsSendFacade.send(body);
    }</code></pre>        
  
#### 개선 
* Thread 간에 간섭이 일어나서 Instance 에 문제가 생기는 것이기 때문에 facade 에서 처리할 때, 서로 간섭하지 않도록 처리 하였다.
     <pre><code>public void handleDelivery(
      Channel channel, 
      Envelope envelope, 
      AMQP.BasicProperties properties, 
      byte[] body) {
     
      new SmsSendFacade().send(body);
  }</code></pre>        