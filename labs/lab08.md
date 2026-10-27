# Lab 08: Monitoring with Prometheus and Grafana

How would you know whether your recommendation service is receiving requests or getting slower? **Monitoring** collects measurements over time to answer these questions.

- **Prometheus** periodically reads (**scrapes**) numerical metrics from web endpoints and stores them as time series. **PromQL** queries those series.
- **Grafana** queries Prometheus and displays the results in dashboard **panels**.
- **Exporters** expose metrics: node exporter reports machine measurements; our Python script reads Kafka recommendation events and reports request counts and latency.

```text
Kafka → Python script → Prometheus → Grafana
Node exporter ────────→ Prometheus
```

**Plan: 40 minutes of work + 5 minutes troubleshooting buffer + 5 minutes checkoff.** Complete installation before recitation. Notes and query examples are allowed during checkoff.

## Before recitation

Clone the [starter repository](https://github.com/VedikaAgarwal2003/mlip-lab8-monitoring) and enter its directory. It includes `docker-compose.yaml`, `requirements.txt`, `kafka-monitoring.py`, and `prometheus/prometheus.yml`.

1. Install Python 3.10 or 3.11 and Docker with Compose; start Docker. In the revised starter directory:

   ```sh
   python3 -m venv .venv
   source .venv/bin/activate
   python -m pip install -r requirements.txt
   docker compose pull
   docker compose config --quiet
   ```

   Windows PowerShell activation: `.venv\Scripts\Activate.ps1`.

2. Establish the Kafka tunnel using your team's details in Canvas:

   ```sh
   ssh -o ExitOnForwardFailure=yes -L <local_port>:localhost:<remote_port> <user>@<remote_server> -NT
   ```

   Keep this terminal open. In another activated terminal, check that recommendation events arrive:

   ```sh
   python kafka-monitoring.py --topic movielogN --bootstrap localhost:<local_port> --check
   ```

   The check prints the first valid recommendation event without requiring the counter TODOs. It times out after about 30 seconds without one; connection failures may appear earlier. A unique consumer group is created for each run, so teammates receive independent streams. Do not reuse your production consumer group.

3. If there are no events, bring the error/output to the TA. The lab supports `--demo` as a synthetic-data fallback; no Kafka tunnel is needed for that mode. This demonstrates monitoring mechanics, not your team's service health.

## 1. Expose and collect metrics — 10 minutes

A **target** is an endpoint Prometheus scrapes; a **job** names a group of targets. The configuration file sets addresses and scrape intervals; metric data is stored separately.

1. Complete the two counter TODOs in `kafka-monitoring.py`: define `Counter('request_count', ..., ['http_status'])`, then increment the matching status label using `.labels(...).inc()`. The exported metric is `request_count_total`. Keep the supplied latency histogram and observation code.
2. In `prometheus/prometheus.yml`, use the self-scrape example to add two separate jobs. Keep the 5-second interval. Use these targets:
   - node exporter: `node-exporter:9100`
   - Python script: `host.docker.internal:8765`

   Containers use Compose service names to reach each other. `host.docker.internal` reaches the host; its Linux mapping is supplied. `localhost` inside Prometheus means the Prometheus container. Targets use `host:port`; `/metrics` is the default path.
3. Start the stack and script in separate terminals:

   ```sh
   docker compose up -d
   python kafka-monitoring.py --topic movielogN --bootstrap localhost:<local_port>
   ```

   If live events are unavailable, stop the script and run `python kafka-monitoring.py --demo`. Use just one script process. Label your dashboard **Synthetic data** if using demo mode.
4. Open [Prometheus](http://localhost:9090), find **Targets**, and confirm three targets are UP. At [the script endpoint](http://localhost:8765/metrics), find `request_count_total` and `request_latency_seconds_bucket`, `_sum`, and `_count`. The script prints the first observation; the labeled counter appears after an event.

Node exporter is preconfigured on the Compose network: do not switch it to host networking. On Docker Desktop, its CPU metrics describe Docker's Linux environment, not native macOS/Windows CPU utilization.

Example log record (illustrative, not a measurement):

```text
2026-10-27T10:00:00,42,recommendation request team-vm:8082, status 200, result: movie-a, movie-b, 87 ms
```

The fourth comma-separated field contains status `200`; the final field contains latency `87` milliseconds. The script converts milliseconds to seconds, ignores unrelated events, and warns about malformed recommendation records.

## 2. Explore PromQL — 10 minutes

A **time series** contains samples over time. **Labels** distinguish series: `http_status="200"` selects successful requests. Prometheus also adds `job` and `instance` labels.

A **counter** accumulates events until its process restarts. Read its raw value for the accumulated count; use `rate()` for events per second or `increase()` for an estimated count during a window. `Counter` is a Python metric type; **there is no PromQL `counter()` function**.

### Query menu

These are examples of what you can ask, not a list to memorize. `{...}` filters labels; `[5m]` selects five minutes of samples. Apply `rate()` before aggregation so resets are handled separately for each series.

```promql
# Did the latest scrape succeed? 1 = yes, 0 = no.
up

# Current counters, one series per status (and target).
request_count_total

# Current successful-request count across series.
sum(request_count_total{http_status="200"})

# Total requests per second.
sum(rate(request_count_total[5m]))

# Estimated requests during the last five minutes.
sum(increase(request_count_total[5m]))

# Request rate broken down by status.
sum by (http_status) (rate(request_count_total[5m]))
```

**Do:** run `up`, the raw counter, and total request rate in Prometheus. Inspect the table labels, then view a graph. Adapt the rate query to successful requests only. Explain its units and which labels remain.

`rate(...[5m])` means **per second**, not per five minutes. It needs at least two scrape samples and adjusts for observed resets. `increase()` extrapolates and may be fractional. Neither reconstructs events missed between scrapes. These metrics describe events processed by this monitor; Kafka lag can delay when they appear.

## 3. Understand latency histograms — 5 minutes

Keeping every request duration becomes expensive at high traffic. A **histogram** instead updates a fixed set of bucket counts, a duration sum (`_sum`), and an observation count (`_count`). Storage still grows with scrape history, buckets, and label combinations, but each new request does not create a new series.

Buckets are cumulative: an observation of 0.08 seconds increments every bucket with an upper bound of at least 0.08, including `+Inf`. The `le` label means “less than or equal to.” Inspect these boundaries in the script and exported metrics.

**p95** estimates the duration at or below which 95% of observations fall. This query retains bucket boundaries (`le`) while combining series:

```promql
histogram_quantile(0.95, sum by (le) (rate(request_latency_seconds_bucket[5m])))
```

Bucket spacing limits precision. A histogram supports mean latency and estimated percentiles, but loses individual durations. **Can it recover exact individual-request standard deviation?** Generally no: this classic histogram has no squared sum or individual values. For example, `[1.2, 1.8]` and `[1.5, 1.5]` give identical counts, sums, and buckets with this script's boundaries at 1 and 2 seconds, but different spreads.

## 4. Build four Grafana panels — 15 minutes

1. Open [Grafana](http://localhost:3000); a fresh installation uses `admin` / `admin`. Add a **Prometheus data source** under **Connections → Data sources**. Set its URL to `http://prometheus:9090` and save/test. This address is reached from Grafana's container.
2. Create a dashboard and add visualizations using that data source. Switch the query editor to **Code**. Use **Last 15 minutes**, refresh every **5 seconds**, and save the dashboard. Menu wording may vary by version.
3. Create these panels with clear titles and units. Reuse the query menu and scaffolds; styling is not graded.

- **Successful requests since monitor restart:** sum the current status-200 counters. Use **Stat**, query type **Instant**, and a count unit. This is not a lifetime service total.
- **Recommendation request rate:** total request rate from the menu. Use **Time series**, query type **Range**, and requests/second units.
- **System-mode CPU utilization:** use the scaffold below. Use **Time series**, **Range**, and **Percent (0–100)**. It measures operating-system CPU time, averaged over cores, rather than all CPU activity.

  ```promql
  100 * avg by (instance) (rate(node_cpu_seconds_total{mode="system"}[5m]))
  ```

- **p95 recommendation latency:** use the histogram query above. Use **Time series**, **Range**, and **seconds**.

Optional exploration: recent mean latency is `sum(rate(request_latency_seconds_sum[5m])) / sum(rate(request_latency_seconds_count[5m]))`.

## 5. TA checkoff — 5 minutes

Show three UP targets and four labeled panels. Explain your two new scrape jobs. Tell the TA whether your data is live or synthetic. The TA chooses **one query task and one conceptual question**; you may use this writeup.

Query tasks: show successful requests per second; show rate by status; or estimate successful requests in the last five minutes.

Conceptual questions: when use a raw counter, `rate()`, or `increase()` and what happens after restart? Why summarize latency in a histogram instead of retaining every duration? Why can't this histogram recover exact latency standard deviation? What does p95 mean and why is it approximate?

## Quick troubleshooting

- **Missing/DOWN target:** inspect its target error and check the address, port, and process. Browser access does not prove container access.
- **Configuration edits:** run `docker compose run --rm --no-deps --entrypoint promtool prometheus check config /etc/prometheus/prometheus.yml`, then `docker compose restart prometheus`. For Compose changes, use `docker compose up -d` to recreate affected services.
- **No application data:** check script output, topic, tunnel, and fresh events. The monitor uses a new group and starts at the end of the log; `--demo` is the fallback. An existing group with committed offsets would resume there instead.
- **Zero rate/undefined p95:** wait for multiple scrapes and fresh observations. A successful scrape does not prove Kafka events are arriving.
- **No status 200:** inspect actual statuses. Do not change topics to hide failures; use the labeled demo fallback if needed.
- **Grafana connection:** use `http://prometheus:9090`. **Unexpected graphs:** check Range/Instant, time range, units, and refresh settings.

References: [PromQL basics](https://prometheus.io/docs/prometheus/latest/querying/basics/), [functions](https://prometheus.io/docs/prometheus/latest/querying/functions/), [histograms](https://prometheus.io/docs/practices/histograms/).
