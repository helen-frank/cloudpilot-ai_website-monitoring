# website-monitoring deployed on kubernetes

## Dependencies

* [kubectl](https://kubernetes.io/docs/tasks/tools/)

## Usage

* `git clone https://github.com/cloudpilot-ai/website-monitoring.git && cd website-monitoring`
* Edit `sample/prometheus-cm1-configmap.yaml` (see targets.yml.example) modify or add `baidu.com` as your desired service address
* Create Namespace `kubectl create namespace prometheus`
* Starting services `kubectl apply -n prometheus -f sample`
* Visualize dashboards http://10.105.212.73:3000/ `kubectl -n prometheus get svc blackbox-exporter` get request address
    ```bash
    # kubectl -n prometheus get svc grafana
    NAME      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
    grafana   ClusterIP   10.105.212.73   <none>        3000/TCP   9m51s
    ```

If you already have Prometheus and Prometheus blackbox exporter up and running just import the dashboards ([website-monitoring](dashboards/website-monitoring.json) or [overview](dashboards/overview.json)) and use the right [datasource](screenshots/import.png) and [jobs](screenshots/import.png) (http_job and icmp_job)

## Dashboards

###  Website monitoring

* HTTP status code
* HTTP redirects
* HTTP version
* TLS version
* Certificate validity
* ICMP
* DNS lookup time
* Availability over the last 24 hours, 3 days and 7 days
* Probe duration and status code history

![web-2](screenshots/website-monitoring_2.png)
![web-3](screenshots/website-monitoring_3.png)

### Overview

* Total number of targets
* Percentage of HTTP 200 status code
* Percentage of targets using SSL
* Global invalid status code history

![overview](screenshots/overview_1.png)

## Tips and tricks

### PromQL

Some useful PromQL queries

* Number of days till certificate expiration
  * `(probe_ssl_earliest_cert_expiry{instance=~"$target",job="$http_job"} - time()) / (60*60*24)`
* Display bad HTTP status code
  * `probe_http_status_code{job="$http_job",instance=~"$target"} != 200`
* Count the number of each status code
  * `count_values("code", probe_http_status_code)`
* Percentage of HTTP 200
  * `((count(count by (instance) (probe_http_status_code == 200))) / (count(count by (instance) (probe_http_status_code)))) * 100`

### Misc

* Request blackbox exporter
  * `kubectl -n prometheus get svc blackbox-exporter` get request address
    ```bash
    # kubectl -n prometheus get svc blackbox-exporter
    NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
    blackbox-exporter   ClusterIP   10.97.216.40   <none>        9115/TCP   6m20s
    ```
  * `curl -s "10.97.216.40:9115/probe?module=http_2xx&target=target.tld"`

## Uninstall

* `kubectl delete -n prometheus -f sample`
