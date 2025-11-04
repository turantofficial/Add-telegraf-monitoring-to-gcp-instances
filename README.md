# Add-telegraf-monitoring-to-gcp-instances

- Add firewall ip of client Port e.g. 9273

**Install Telegraf** in GCP VM:

```
curl --silent --location -O \
https://repos.influxdata.com/influxdata-archive.key \
&& echo "943666881a1b8d9b849b74caebf02d3465d6beb716510d86a39f6c8e8dac7515  influxdata-archive.key" \
| sha256sum -c - && cat influxdata-archive.key \
| gpg --dearmor \
| sudo tee /etc/apt/trusted.gpg.d/influxdata-archive.gpg > /dev/null \
&& echo 'deb [signed-by=/etc/apt/trusted.gpg.d/influxdata-archive.gpg] https://repos.influxdata.com/debian stable main' \
| sudo tee /etc/apt/sources.list.d/influxdata.list
sudo apt-get update && sudo apt-get install telegraf
```


**Configure telegraf.conf** (Server Metrics und Nginx):

```
global_tags]
  environment = "dev"
[agent]
  interval = "10s"
  round_interval = true
  metric_batch_size = 1000
  metric_buffer_limit = 10000
  collection_jitter = "0s"
  flush_interval = "10s"
  flush_jitter = "0s"
  precision = "0s"
  hostname = "hostname1"
  omit_hostname = false
[[outputs.influxdb_v2]]
  urls = ["http://10.10.10.10:8086"]
  token = "secret"
  organization = "Company A"
  bucket = "jobvector_server/autogen"
 [[outputs.prometheus_client]]
   listen = ":9273"
[[inputs.cpu]]
  percpu = true
  totalcpu = true
  collect_cpu_time = false
  report_active = false
  core_tags = false
[[inputs.disk]]
  ignore_fs = ["tmpfs", "devtmpfs", "devfs", "iso9660", "overlay", "aufs", "squashfs"]
[[inputs.diskio]]
[[inputs.kernel]]
[[inputs.mem]]
[[inputs.processes]]
[[inputs.swap]]
[[inputs.system]]
#[[inputs.nginx]]
#  urls = ["http://127.0.0.1:9090/nginx_status"]
#  response_timeout = "5s"

```

**Edit hostname, environment** (staging, production).

```
systemctl enable telegraf
systemctl start telegraf
```


Differentiation between staging and production hosts in the Grafana dashboard:

**User Variables Query**:

```
import "influxdata/influxdb/schema"
schema.tagValues(
  bucket: "${bucket}",
  tag: "host",
  predicate: (r) => r.environment == "${environment}"
)
```

**Add environment variable and query**:

```
import "influxdata/influxdb/schema"

schema.tagValues(
  bucket: "${bucket}",
  tag: "environment"
)
```

set global_tags environment in telegraf.conf:

```
[global_tags]
  environment = "dev" #or production
```
restart telegraf.



Further commands:
```
influx auth list --org Jobvector --token $INFLUXDB_TOKEN
influx config
influx ping
printenv | grep INFLUX
influx org list --token $INFLUXDB_TOKEN
influx org list --token $INFLUXDB_TOKEN
influx bucket create --name jobvector_staging --org Jobvector --token $INFLUXDB_TOKEN
influx user list --token $INFLUXDB_TOKEN
influx user create --name "tosun" --token $INFLUXDB_TOKEN
influx user password --name "tosun" --token $INFLUXDB_TOKEN
#eg
influx query 'from(bucket: "jobvector_staging") |> range(start: -1h)' --org "Jobvector" --token $INFLUXDB_TOKEN
influx query 'from(bucket: "jobvector_server/autogen") |> range(start: -1h)' --org "Jobvector" --token $INFLUXDB_TOKEN
echo "cpu,host=db-staging usage=0.5" | influx write --bucket jobvector_staging --org "Jobvector" --token $INFLUXDB_TOKEN
```
