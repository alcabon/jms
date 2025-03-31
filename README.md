# jms

```mermaid
%%{init: {'theme':'neutral'}}%%
flowchart TD
    subgraph Writer_Application[JMS/JNDI Client]
        A[Writer App] -->|JMS API| B[JNDI Context]
        B -->|Lookup| C[(Connection Factory)]
        B -->|Lookup| D[(Destination Queue)]
        C -->|Bindings Mode| E[Queue Manager]
    end

    subgraph MQ_Server
        E --> F[Local Queue]
        E --> G[Channel]
        E --> H[Security Policies]
    end

    subgraph Reader_Application[MQ Client]
        I[Reader App] -->|MQ Client Libs| E
    end

    A -->|PUT Message| F
    I -->|GET Message| F

    style A fill:#e6f3ff,stroke:#4a90e2
    style I fill:#ffe6e6,stroke:#e24a4a
    style E fill:#f0f0f0,stroke:#666
```


Key Components Explained:

JMS/JNDI Writer Configuration ():

Uses JNDI Context to lookup:

ConnectionFactory (defines transport type, queue manager)

Destination (target queue)

Example JNDI properties:

text
java.naming.provider.url=file:///mq/jndi
java.naming.factory.initial=com.sun.jndi.fscontext.RefFSContextFactory
MQ Server Essentials ():

Queue Manager: Manages queues/channels

Local Queue: Persistent message storage

Channel: Communication endpoint (e.g., SYSTEM.DEF.SVRCONN)

Security: TLS/SSL channel encryption

MQ Client Reader ():

Direct connection using IBM MQ client libraries

Requires:

```java
MQQueueManager qMgr = new MQQueueManager("QMGR_NAME");
MQQueue queue = qMgr.accessQueue("QUEUE_NAME", CMQC.MQOO_INPUT_AS_Q_DEF);
Implementation Steps:
```

JNDI Setup ():

shell
# Using JMSAdmin tool
DEFINE QCF(APP_QCF) QMANAGER(QM1) TRANSPORT(BINDINGS)
DEFINE Q(REQUEST_QUEUE) QUEUE(APP.REQUESTS)
Writer Code Snippet ():

```java
Context ctx = new InitialContext(); // JNDI context
ConnectionFactory cf = (ConnectionFactory) ctx.lookup("APP_QCF");
Queue queue = (Queue) ctx.lookup("REQUEST_QUEUE");
Connection conn = cf.createConnection();
Session session = conn.createSession();
MessageProducer producer = session.createProducer(queue);
TextMessage msg = session.createTextMessage("Payload");
producer.send(msg);
Reader Code Snippet ():
```

```java
MQEnvironment.hostname = "mqhost.example.com";
MQEnvironment.port = 1414;
MQEnvironment.channel = "APP.CHANNEL";
MQQueueManager qMgr = new MQQueueManager("QM1");
MQQueue queue = qMgr.accessQueue("APP.REQUESTS", CMQC.MQOO_INPUT);
MQMessage receivedMessage = new MQMessage();
queue.get(receivedMessage);
Security Considerations ():
```

text
# Add to JNDI properties
java.naming.security.principal=appuser
java.naming.security.credentials=secret
The diagram shows two distinct connection patterns:

Writer: Uses JNDI-administered objects for portability

Reader: Direct MQ client connection for simplicity

Both applications share the same physical queue (APP.REQUESTS) on the MQ Server, enabling decoupled communication.
