# 1. Sqoop to HDFS
### 1. 특정 테이블
sqoop import --connect jdbc:mysql://172.16.31.180:3306/hive_schema --username root --password root --table hive_table1 --target-dir /user/root/abc</br>

### 2. 전체 테이블
sqoop import-all-tables --connect jdbc:mysql://172.16.31.180:3306/hive_schema --username root --password root --exclude-tables table1,table2 --warehouse-dir /teststore/input/</br>

### 3. 테이블 Query
sqoop import --connect jdbc:mysql://172.16.31.180:3306/hive_schema --username root --password root --query 'SELECT id, name FROM hive_table1 WHERE age > 50 AND $CONDITIONS' --target-dir /user/root/abcd --split-by id</br>

# 2. Sqoop to Hive
### 1. 특정 테이블
sqoop import --connect  jdbc:mysql://172.16.31.180:3306/hive_schema --username root --password root --table test1 --hive-import</br>

### 2. 테이블 이름 정하기.
sqoop import --connect  jdbc:mysql://172.16.31.180:3306/test_schema --username root --password root --table test2 --hive-import --create-hive-table --hive-table default.custom</br>
