---
layout: post
title: cloudhopper ssl configuration
---

## cloudhopper-smpp ssl option 설정  

### openmarket 에서 기존의 ssl 사용에서 TLS 1.2 를 사용하겠다고 요청이 왔기 때문에 이에 대한 문제를 업데이트 해주어야 했다.  

    SslConfiguration sslConfig = new SslConfiguration();
    sslConfig.setValidateCerts(true);
    sslConfig.setValidatePeerCerts(true);

    sessionConfig.setSslConfiguration(sslConfig);
    sessionConfig.setUseSsl(true);
    ...

### 이러한 코드로 설정하여 처리 중이였던 부분을 업데이트 해주어야 했지만 cloudhopper 가 많이 사용되지 않기 대문에 이에 대한 정보를 찾아야 했다.

    sslConfig.setProtocol("TLSv1.2");
    
### protocol 설정을 하였지만 동작하지 않았기 때문에 openmarket 에 문의한 결과 SSLv2Hello 를 사용한다고 답변이 왔기에 이에 따른 프로토콜 처리를 한다. 

    SslConfiguration sslConfig = new SslConfiguration();
    sslConfig.addExcludeProtocols("TLSv1.0", "TLSv1.1", "SSLv2Hello");
    sslConfig.setValidateCerts(true);
    sslConfig.setValidatePeerCerts(true);

    sessionConfig.setSslConfiguration(sslConfig);
    sessionConfig.setUseSsl(true);
    ...    
  
## 참조 
   
   * fizzed/cloudhopper-smpp
     > <https://github.com/fizzed/cloudhopper-smpp/blob/master/SSL.md> 