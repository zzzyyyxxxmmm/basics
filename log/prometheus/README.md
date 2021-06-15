# Overview
<div align=center>
<img src="https://www.oreilly.com/library/view/prometheus-up/9781492034131/assets/prur_0101.png" width="700" height="500">
</div>

Service discovery and relabelling give us a list of targets to be monitored. 

Now Prom‐ etheus needs to fetch the metrics. Prometheus does this by sending a HTTP request called a scrape. The response to the scrape is parsed and ingested into storage. Several useful metrics are also added in, such as if the scrape succeeded and how long it took. Scrapes happen regularly; usually you would configure it to happen every 10 to 60 seconds for each target.

### prometheus.yml
```yaml
global: scrape_interval: 10s
scrape_configs:
    - job_name: prometheus  # added as a label and be grouped in Targets
        static_configs: 
            - targets:
                - localhost:9090
```

### prometheus.yml add node exporter
```yaml
global: scrape_interval: 10s
scrape_configs:
    - job_name: prometheus
        static_configs:
            - targets:
                - localhost:9090
    - job_name: node 
        static_configs:
            - targets:
                - localhost:9100
```

### prometheus.yml add alert manager
```yaml
global:
scrape_interval: 10s 
evaluation_interval: 10s
rule_files: - rules.yml
alerting: alertmanagers:
- static_configs:
- targets:
- localhost:9093
scrape_configs:
- job_name: prometheus
static_configs: - targets:
- localhost:9090 - job_name: node
static_configs: - targets:
- localhost:9100

```

```yaml
groups:
- name: example
rules:
- alert: InstanceDown
     expr: up == 0
     for: 1m
```

The InstanceDown alert will be evaluated every 10 seconds in accordance with the evaluation_interval. If a series is continuously returned for at least a minute9 (the for), then the alert will be considered to be firing.
# Type

### Gauge
show a real time situation
Examples of gauges include:
* the number of items in a queue
* memory usage of a cache
* number of active threads
* the last time a record was processed
* average requests per second in the last minute

### Counter

```
In addition to _total, the _count, _sum, and _bucket suffixes also have other mean‐ ings and should not be used as suffixes in your metric names to avoid confusion.
It is also strongly recommended that you include the unit of your metric at the end of its name. For example, a counter for bytes processed might be myapp_requests_ processed_bytes_total.
```

Counters track how many events have happened, or the total size of all the events.

Counters are always increasing. This creates nice up and to the right graphs, but the values of counters are not much use on their own. What you really want to know is how fast the counter is increasing, which is where the rate function comes in. The rate function calculates how fast a counter is increasing per second. Adjust your expression to rate(prometheus_tsdb_head_samples_appended_total[1m]), which will calculate how many samples Prometheus is ingesting per second averaged over one minute 

### Sum
Knowing how long your application took to respond to a request or the latency of a backend are vital metrics when you are trying to understand the performance of your systems. 
```python
import time
from prometheus_client import Summary
LATENCY = Summary('hello_world_latency_seconds', 'Time for a request Hello World.')

class MyHandler(http.server.BaseHTTPRequestHandler): 
    def do_GET(self):
    start = time.time() 
    self.send_response(200) 
    self.end_headers() 
    self.wfile.write(b"Hello World") 
    LATENCY.observe(time.time() - start)
```

If you look at the /metrics you will see that the hello_world_latency_seconds metric has two time series: ```hello_world_latency_seconds_count``` and ```hello_world_latency_seconds_sum```.
hello_world_latency_seconds_count is the number of observe calls that have been made, so ```rate(hello_world_latency_seconds_count[1m])``` in the expression browser would return the per-second rate of Hello World requests.
```hello_world_latency_seconds_sum``` is the sum of the values passed to observe, so ```rate(hello_world_latency_seconds_sum[1m])``` is the amount of time spent responding to requests per second.
If you divide these two expressions you get the average latency over the last minute. The full expression for average latency would be:
```
rate(hello_world_latency_seconds_sum[1m]) /
rate(hello_world_latency_seconds_count[1m])
```
Let’s take an example. Say in the last minute you had three requests that took 2, 4, and 9 seconds. The count would be 3 and the sum would be 15 seconds, so the average latency is 5 seconds. rate is per second rather than per minute, so you in principle need to divide both sides by 60, but that cancels out.

### The Histogram
A summary will provide the average latency, but what if you want a quantile? Quan‐ tiles tell you that a certain proportion of events had a size below a given value. For example, the 0.95 quantile being 300 ms means that 95% of requests took less than 300 ms.

For example, the 0.95 quantile (95th percentile) would be:
```histogram_quantile(0.95, rate(hello_world_latency_seconds_bucket[1m]))```
The rate is needed as the buckets’ time series are counters.

### Buckets
custom range by yourself
```
LATENCY = Histogram('hello_world_latency_seconds',
'Time for a request Hello World.', buckets=[0.0001, 0.0002, 0.0005, 0.001, 0.01, 0.1])
```

If you have looked at a /metrics for a histogram, you probably noticed that the buck‐ ets aren’t just a count of events that fall into them. The buckets also include a count of events in all the smaller buckets, all the way up to the +Inf, bucket which is the total number of events. This is known as a cumulative histogram, and why the bucket label is called le, standing for less than or equal to.

# What should be exposed as metric

## Online
Online-serving systems are those where either a human or another service is waiting on a response. These include web servers and databases. The key metrics to include in service instrumentation are the request rate, latency, and error rate. Having request rate, latency, and error rate metrics is sometimes called the RED method, for Requests, Errors, and Duration. These metrics are not just useful to you from the server side, but also the client side. If you notice that the client is seeing more latency than the server, you might have network issues or an overloaded client.

## Offline 
Offline-serving systems do not have someone waiting on them. They usually batch up work and have multiple stages in a pipeline with queues between them. A log pro‐ cessing system is an example of an offline-serving system. For each stage you should have metrics for the amount of queued work, how much work is in progress, how fast you are processing items, and errors that occur. These metrics are also known as the USE method for Utilisation, Saturation, and Errors. Utilisation is how full your ser‐ vice is, saturation is the amount of queued work, and errors is self-explanatory. If you are using batches, then it is useful to have metrics both for the batches, and the indi‐ vidual items.

Batch jobs are the third type of service, and they are similar to offline-serving sys‐ tems. However, batch jobs run on a regular schedule, whereas offline-serving systems run continuously. As batch jobs are not always running, scraping them doesn’t work too well, so techniques such as the Pushgateway (discussed in “Pushgateway” on page 71) are used. At the end of a batch job you should record how long it took to run, how long each stage of the job took, and the time at which the job last succeeded. You can add alerts if the job hasn’t succeeded recently enough, allowing you to tolerate indi‐ vidual batch job run failures.

Thread and worker pools should be instrumented similarly to offline-serving sys‐ tems. You will want to have metrics for the queue size, active threads, any limit on the number of threads, and errors encountered.

# Labels
Labels come from two sources, instrumentation labels and target labels. When you are working in PromQL there is no difference between the two, but it’s important to dis‐ tinguish between them in order to get the most benefits from labels.
Instrumentation labels, as the name indicates, come from your instrumentation. They are about things that are known inside your application or library, such as the type of HTTP requests it receives, which databases it talks to, and other internal specifics.

Target labels identify a specific monitoring target; that is, a target that Prometheus scrapes. A target label relates more to your architecture and may include which application it is, what datacenter it lives in, if it is in a development or production environment, which team owns it, and of course, which exact instance of the application it is. Target labels are attached by Prometheus as part of the process of scraping metrics.

## Name
library_name_unit_suffix

Another thing to avoid is having a time series that is a total of the rest of the metric such as:
```
some_metric{label="foo"} 7 
some_metric{label="bar"} 13 
some_metric{label="total"} 20
or
some_metric{label="foo"} 7 
some_metric{label="bar"} 13 
some_metric{} 20
```

Both of these break aggregation with sum in PromQL as you’d be double counting.


## Example
1. running target
```
up == 0
```

2. aggregate labels
sum without (key) (ack_latency_sum)