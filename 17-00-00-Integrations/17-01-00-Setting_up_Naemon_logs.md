# Setting up Naemon logs
### Logstash
1. In `op5naemon_beat.conf` set up `ELASTICSEARCH_HOST`, `ES_PORT`, `FILEBEAT_PORT`
2. Copy `op5naemon_beat.conf` to `/etc/logstash/conf.d`
3. Based on "FILEBEAT_PORT" if firewall is running:
```bash
sudo firewall-cmd --zone=public --permanent --add-port=FILEBEAT_PORT/tcp
sudo firewall-cmd --reload
```
4. Based on amount of data that elasticsearch will receive you can also choose whether you want index creation to be based on moths or days:
```json
index => "op5-naemon-%{+YYYY.MM}"
or
index => "op5-naemon-%{+YYYY.MM.dd}"
```
5. Copy `naemon` file to /etc/logstash/patterns and make sure it is readable by logstash process
6. Restart *logstash* configuration e.g.:
```bash
sudo systemct restart logstash
```
### Elasticsearch
1. Connect to Elasticsearch node via SSH and Install index pattern for naemon logs. Note that if you have a default pattern covering *settings* section you should delete/modify that in naemon_template.sh:
```json
  "settings": {
    "number_of_shards": 5,
    "auto_expand_replicas": "0-1"
  },
```
2. Install template by running:
`./naemon_template.sh`

### OP5 Monitor
1. On OP5 Monitor host install filebeat (for instance via rpm `https://www.elastic.co/downloads/beats/filebeat`)
2. In `/etc/filebeat/filebeat.yml` add:
```yaml
#=========================== Filebeat inputs =============================
filebeat.config.inputs:
  enabled: true
  path: configs/*.yml
```
3. You also will have to configure the output section in `filebeat.yml`. You should have one logstash output:
```yaml
#----------------------------- Logstash output --------------------------------
output.logstash:
  # The Logstash hosts
  hosts: ["LOGSTASH_IP:FILEBEAT_PORT"]
```
If you have few logstash instances - `Logstash` section has to be repeated on every node and `hosts:` should point to all of them:
```yaml
  hosts: ["LOGSTASH_IP:FILEBEAT_PORT", "LOGSTASH_IP:FILEBEAT_PORT", "LOGSTASH_IP:FILEBEAT_PORT" ]
```
4. Create `/etc/filebeat/configs` catalog.
5. Copy `naemon_logs.yml` to a newly created catalog.
6. Check the newly added configuration and connection to logstash. Location of executable might vary based on os:
```bash
/usr/share/filebeat/bin/filebeat --path.config /etc/filebeat/ test config
/usr/share/filebeat/bin/filebeat --path.config /etc/filebeat/ test output
```
7. Restart filebeat:
```bash
sudo systemctl restart filebeat # RHEL/CentOS 7
sudo service filebeat restart # RHEL/CentOS 6
```
### Elasticsearch
At this moment there should be a new index on the Elasticsearch node:
```bash
curl -XGET '127.0.0.1:9200/_cat/indices?v'
```
Example output:
```
health status index                 uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   op5-naemon-2018.11    gO8XRsHiTNm63nI_RVCy8w   1   0      23176            0      8.3mb          8.3mb
```
If the index has been created, in order to browse and visualise the data, "index pattern" needs to be added in Kibana.
