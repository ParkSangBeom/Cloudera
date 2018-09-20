# Flume 사용법

### 1. Local 전송.
yum install telnet</br>
Telnet localhost 9999</br>
Hello world!</br>
/var/log/flume-ng/﻿flume-cmf-flume-AGENT-quickstart.cloudera.log</br>
====================================</br>
﻿2015-06-05 15:45:55,561 INFO org.apache.flume.sink.LoggerSink: </br>
Event: { headers:{} body: 48 65 6C 6C 6F 20 77 6F 72 6C 64 21 0D Hello world!. }</br>
====================================</br>
</br>
HDFS에 Event저장 폴더 생성</br>
/user/flume/event</br>
hadoop fs -chown -R flume:flume /user/flume</br>
hadoop fs -chmod -R 775 /user/flume</br>
</br>
Cloudera Manager 접속</br>
flume ->  Configuration -> Configuration File</br>
====================================</br>
tier1.sinks.sink1.type= HDFS</br>
tier1.sinks.sink1.fileType=DataStream</br>
tier1.sinks.sink1.channel      = channel1</br>
tier1.sinks.sink1.hdfs.path = /user/flume/event</br>
====================================</br>
</br>
Telnet localhost 9999</br>
Hello world!</br>
hadoop fs -ls /user/flume/event/*</br>

### 2. Avro 전송.
##### 보내는 Server
====================================</br>
agentDataSource.sources = otvSource</br>
agentDataSource.channels = otvChannel</br>
agentDataSource.sinks = avroSink</br>
</br> 
\# Source : Spooling Directory</br>
agentDataSource.sources.otvSource.type = netcat</br>
agentDataSource.sources.otvSource.channels = otvChannel</br>
agentDataSource.sources.otvSource.bind     = 127.0.0.1</br>
agentDataSource.sources.otvSource.port     = 9999</br>
</br> 
\# Sink : Avro</br>
agentDataSource.sinks.avroSink.serializer = AVRO_EVENT</br>
agentDataSource.sinks.avroSink.type = avro</br>
agentDataSource.sinks.avroSink.channel = otvChannel</br>
agentDataSource.sinks.avroSink.hostname = 172.16.31.181</br>
agentDataSource.sinks.avroSink.port = 10000</br>
</br> 
\# Channel : Memory</br>
agentDataSource.channels.otvChannel.type = memory</br>
agentDataSource.channels.otvChannel.capacity = 100</br>
====================================</br>
</br>
##### 받는 Server
====================================</br>
agentDataCollector.sources = targetSource</br>
agentDataCollector.channels = targetChannel</br>
agentDataCollector.sinks = targetSink</br>
</br>
\# Source : Avro</br>
agentDataCollector.sources.targetSource.type = avro</br>
agentDataCollector.sources.targetSource.channels = targetChannel</br>
agentDataCollector.sources.targetSource.bind = 172.16.31.181</br>
agentDataCollector.sources.targetSource.port = 10000</br>
</br>
agentDataCollector.sinks.targetSink.type         = logger</br>
agentDataCollector.sinks.targetSink.channel      = targetChannel</br>
</br>
\# Channel : Memory</br>
agentDataCollector.channels.targetChannel.type = memory</br>
agentDataCollector.channels.targetChannel.capacity = 100</br>
====================================</br>

