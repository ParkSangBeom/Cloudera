# Flume 사용법

### 1. Telnet
yum install telnet</br>
Telnet localhost 9999</br>
Hello world!</br>
/var/log/flume-ng/﻿flume-cmf-flume-AGENT-quickstart.cloudera.log</br>
====================================</br>
﻿2015-06-05 15:45:55,561 INFO org.apache.flume.sink.LoggerSink: </br>
Event: { headers:{} body: 48 65 6C 6C 6F 20 77 6F 72 6C 64 21 0D Hello world!. }</br>
====================================</br>
</br>
HDFS에 Event를 저장 폴더 생성</br>
/user/flume/event</br>
hadoop fs -chown -R flume:flume /user/flume</br>
hadoop fs -chmod -R 775 /user/flume</br>
</br>
Cloudera Manager 접속</br>
flume ->  Configuration -> Configuration File</br>
====================================</br>
tier1.sinks.sink1.type= HDFS
tier1.sinks.sink1.fileType=DataStream
tier1.sinks.sink1.channel      = channel1
tier1.sinks.sink1.hdfs.path = /user/flume/event
====================================</br>
