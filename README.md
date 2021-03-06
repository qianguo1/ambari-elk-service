# ambari-elk-service 

Ambari service for installing and managing ELK stack on HDP clusters.

- Ambari Version=2.2.2
- Elasticsearch Version=2.3.5
- Logstash Version=2.3.4
- Kibana Version=4.5.4

## Add ELK Repository

Edit */var/lib/ambari-server/resources/stacks/HDP/2.4/repos/repoinfo.xml* to add additional <repo> entries for ELK.

```
  <os family="redhat6">
    <repo>
      <baseurl>http://public-repo-1.hortonworks.com/HDP/centos6/2.x/updates/2.4.0.0</baseurl>
      <repoid>HDP-2.4</repoid>
      <reponame>HDP</reponame>
    </repo>
    <repo>
      <baseurl>http://public-repo-1.hortonworks.com/HDP-UTILS-1.1.0.20/repos/centos6</baseurl>
      <repoid>HDP-UTILS-1.1.0.20</repoid>
      <reponame>HDP-UTILS</reponame>
    </repo>
    <repo>
      <baseurl>https://packages.elastic.co/elasticsearch/2.x/centos</baseurl>
      <repoid>elasticsearch-2.3</repoid>
      <reponame>elastic</reponame>
    </repo>
    <repo>
      <baseurl>http://packages.elastic.co/kibana/4.5/centos</baseurl>
      <repoid>kibana-4.5</repoid>
      <reponame>kibana</reponame>
    </repo>
  </os>
```

## Add ELK Service
1. On the Ambari Server, browse to the */var/lib/ambari-server/resources/stacks/HDP/2.4/services* directory. In this case, we will browse to the HDP 2.4 Stack definition.

```
cd /var/lib/ambari-server/resources/stacks/HDP/2.4/services
```

2. Create a directory named */var/lib/ambari-server/resources/stacks/HDP/2.0.6/services/__ELK__* that will contain the service definition for ELK.

```
mkdir /var/lib/ambari-server/resources/stacks/HDP/2.4/services/ELK
cd /var/lib/ambari-server/resources/stacks/HDP/2.4/services/ELK
```

3. Download and copy this project to service definition directory.

```
cp -r ambari-elk-service/* /var/lib/ambari-server/resources/stacks/HDP/2.4/services/ELK
```

4. Restart Ambari Server for this new service definition to be distributed to all the Agents in the cluster.

```
ambari-server restart
```

## Install ELK Service  (via Ambari Web "Add Services")

1. In Ambari Web, browse to Services and click the Actions button in the Service navigation area on the left.
2. The "Add Services" wizard launches. You will see an option to include ELK.
3. Select ELK and click Next.
4. Assign master components to hosts you want to run them on and click Next.
    * Select more than one hosts for Elastic DataNode to set up a production Elasticsearch cluster.
    * Select one host for Kibana Server.
5. Assign slave and client components to hosts you want to run them on and click Next.
    * Select all hosts for Logstash Agents.
    * If Kibana Server host is not an Elasticsearch DataNode, select Elastic ClientNode for Kibana Server host. Otherwise, don't select Elastic ClientNode.
6. Customize services and click Next. 
   If you have installed all following services and their comopnents, you don't have to change any ELK service configuration.
   Otherwise, you need remove uninstalled services/componnets in *logstash-data-source*.
    * HDFS: NameNode, DataNode, JournalNode
    * YARN: ResourceManager, NodeManager, App Timeline Server
    * Hive: Hive Metastore, HiveServer2, WebHCat Server
    * HBase: Master, RegionServer, Phoenix Query Server
    * Zookeeper: Zookeeper Server
7. Review the configuration and click Deploy.
8. Once complete, the ELK service will be available in the Service navigation area.

## Interactively Explore MapReduce Data (via Kibana UI)

A link of Kibana UI is added as Quick Links of service ELK.

1. In Ambari Web, browse to Services and click the ELK service in the Service navigation area on the left.
2. The summary of ELK service is displayed on the right. Click *Quick Links* and select *Kibana UI* in the dropdown box.
3. Logon the host where kibana is installed; run following script to create Kibana index patterns:
```
/opt/kibana/config/kibana-create-index-patterns.sh
```
4. In Kibana UI, browse to *Settings* on the top and click *Indices* in the Settings navigation area on the left. You will see following index patterns.
5. Browse to *Discover* on the top; now you can interactively explore your data from the Discover page.
    * logstash-mapred-running is an index of running mapred jobs.
    * logstash-mapred-finished is an index of finished mapred jobs.
    * logstash-mapred-aggregated-queue is an index of aggregated memory usage per queue.
    * logstash-mapred-aggregated-user is an index of aggregated memory usage per user.
    * logstash-yarn-allocation is an index of allocated memory/vcores per queue among yarn cluster.
    * logstash-hdfs-namenode-log is an index of warning, error or fatal hdfs namodenode log.
    * logstash-hdfs-secondarynamenode-log  is an index of warning, error or fatal hdfs secondarynamenode log.
    * logstash-hdfs-datanode-log is an index of warning, error or fatal hdfs datanode log.
    * logstash-hdfs-journalnode-log is an index of warning, error or fatal hdfs journalnode log.
    * logstash-yarn-nodemanager-log  is an index of warning, error or fatal yarn nodemanager log.
    * logstash-yarn-resourcemanager-log  is an index of warning, error or fatal yarn resourcemanager log.
    * logstash-yarn-timelineserver-log  is an index of warning, error or fatal yarn timelineserver log.
    * logstash-hbase-master-log is an index of warning, error or fatal hbase master log.
    * logstash-hbase-regionserver-log is an index of warning, error or fatal hbase regionserver log.
    * logstash-hbase-phoenix-log is an index of warning, error or fatal hbase phoenix query server log.
    * logstash-zookeeper-server-log is an index of warning, error or fatal zookeeper server log.
    * logstash-hive-metastore-log is an index of warning, error or fatal hive metastore log.
    * logstash-hive-server2-log is an index of warning, error or fatal hive server2 log.
    * logstash-hive-webhcat-log is an index of warning, error or fatal hive webhcat log.

## MapReduce Jobs Dashboard (via Kibana UI)

As an example, load JSON visualization and dashboard to view the MapReduce Jobs.

1. In Ambari Web, browse to *Settings* and click *Objects*.
2. Click *Import* in the Objects area at top right corner. Select JSON file *ambari-elk-service/package/templates/kibana-import.json*.
3. Browse to *Dashboard* and click the *Load Saved Dashboard* button. Select the imported dashboard *dashboard-demo* in the dropdown box.
4. Now you will see following graphs/tables:
    * Allocated memory/vcores per queue in yarn cluster.
    * The top-10 long finished mapred jobs.
    * The top-10 long running mapred jobs.
    * The average running time per queue of mapred jobs.
    * The average running time per user of mapred jobs.
    * The memory usage per queue of mapred cluster.
    * The memory usage per user of mapred cluster.
4. Select the imported dashboard *logs* in the dropdown box.
5. Now you will see warning, error or fatal logs of different service components.
6. At the upper right corner you can change the time range to view at different time.

## MapReduce Job Alert (via Ambari UI web "Alerts")
1. In Ambari Web, browse to Alerts and select "ELK Default" in Groups dropdown box.
2. Click "MapReduce Job Execution Time".
3. This alert is triggered if the execution time of finished MapReduce Jobs is greater than 100 seconds.
