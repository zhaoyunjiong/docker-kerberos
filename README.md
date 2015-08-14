## Docker kerberos
This image
is designed to support the Hadoop clusters launched by Cloudbreak. The default realm is `NODE.DC1.CONSUL` and the default admin principal is `admin/admin`. All the default values can be modified with environment variables.

### Usage

The image automatically bootstrap kerberos with the given environment variables, which means
when the container is launched first time it will create the DB for kerberos along with the admin user and start the KDC.
This use-case is convenient for a quick start.

#### Quick start
```
docker run -d --net=host -v /etc/krb5.conf:/etc/krb5.conf -v /dev/urandom:/dev/random --name kerberos sequenceiq/kerberos
```
The containers have a pretty bad entropy level so the KDC won't start because of this. We can overcome this by using `/dev/urandom` which is less secure but does not care about entropy.
The `/etc/krb5.conf` is shared with the host so the generated configuration will be present on the host as well. We need to share this configuration with the `ambari-server` container as well or you need to take care of the copying.
Once the container is running you can enable kerberos with `Ambari`.

### Test
Once kerberos is enabled you need a `ticket` to execute any job on the cluster. Here's an example to get a ticket:
```
kinit -V -kt /etc/security/keytabs/yarn.service.keytab yarn/karalabe-0-1426768348448.node.dc1.consul@NODE.DC1.CONSUL
```
Example job:
```java
export HADOOP_LIBS=/usr/hdp/current/hadoop-mapreduce-client
export JAR_JOBCLIENT=$HADOOP_LIBS/hadoop-mapreduce-client-jobclient.jar

hadoop jar $JAR_JOBCLIENT mrbench -baseDir /user/ambari-qa/smallJobsBenchmark -numRuns 5 -maps 10 -reduces 5 -inputLines 10 -inputType ascending
```
