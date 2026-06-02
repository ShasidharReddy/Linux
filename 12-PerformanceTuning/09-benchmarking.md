# Benchmarking

[Back to guide index](README.md)

Benchmarking proves whether tuning helped.

This section covers:

- benchmark design
- popular tools
- methodology best practices
- interpretation

## 9.1 Benchmarking principles

- define the goal
- define the metric
- match production-like load
- warm up first
- run long enough
- repeat runs
- capture variance
- collect system metrics in parallel

## 9.2 Record these for every benchmark

- hardware
- kernel
- distro
- governor
- NUMA layout
- storage type
- filesystem
- NIC type
- tool version
- command line
- duration
- warmup duration
- concurrency
- latency percentiles
- errors
- CPU usage

## 9.3 Common pitfalls

- short tests only
- cached reads mistaken for storage performance
- no warmup
- load generator saturation
- averages only
- different environments compared unfairly

## 9.4 `sysbench`

### 9.4.1 CPU example

```bash
sysbench cpu --cpu-max-prime=20000 run
```

### 9.4.2 Memory example

```bash
sysbench memory --memory-block-size=1M --memory-total-size=10G run
```

### 9.4.3 Threads example

```bash
sysbench threads --threads=64 --time=30 run
```

### 9.4.4 File I/O example

```bash
sysbench fileio --file-total-size=10G prepare
sysbench fileio --file-total-size=10G --file-test-mode=rndrw --time=60 --max-requests=0 run
sysbench fileio --file-total-size=10G cleanup
```

## 9.5 `fio`

Use for realistic storage tests.

Focus on:

- block size
- queue depth
- numjobs
- direct vs buffered I/O
- p95 and p99 latency

## 9.6 `iperf3`

Use for network bandwidth and UDP loss testing.

Test:

- both directions
- single stream
- parallel streams
- realistic duration

## 9.7 HTTP tools

Common tools:

- `ab`
- `wrk`
- `hey`

### 9.7.1 `ab`

```bash
ab -n 100000 -c 200 http://127.0.0.1:8080/
```

### 9.7.2 `wrk`

```bash
wrk -t8 -c400 -d60s http://127.0.0.1:8080/
```

### 9.7.3 `hey`

```bash
hey -n 100000 -c 200 http://127.0.0.1:8080/
```

## 9.8 `stress-ng`

Useful for:

- contention tests
- thermal tests
- alert validation
- reproducing resource pressure

Examples:

```bash
stress-ng --cpu 8 --timeout 60s
stress-ng --vm 4 --vm-bytes 80% --timeout 60s
stress-ng --hdd 4 --timeout 60s
```

## 9.9 `unixbench`

Useful as a broad synthetic suite.

Do not rely on it alone for production decisions.

## 9.10 Percentiles

Always collect:

- p50
- p90
- p95
- p99
- max if useful

## 9.11 Coordinated omission

Some load generators under-report latency during pauses.

Know your tool.

## 9.12 Benchmark interpretation

| Observation | Meaning |
|---|---|
| throughput rises with stable latency | headroom remains |
| throughput plateaus and latency rises | saturation |
| CPU low and latency high | non-CPU bottleneck |
| one core hot and others idle | poor parallelism |
| errors appear before saturation | stability limit reached |

## 9.13 Sample benchmark plan

1. define baseline
2. warm up for 10 minutes
3. run 5 trials
4. collect host metrics
5. collect service metrics
6. compare medians and percentiles
7. change one variable
8. repeat using same method

## 9.14 Benchmarking practices

- use realistic concurrency
- use representative payloads
- capture environment details
- benchmark after each change
- keep result logs and commands together

## 9.15 Benchmarking quick commands

```bash
sysbench cpu --cpu-max-prime=20000 run
sysbench memory --memory-block-size=1M --memory-total-size=10G run
fio --name=randread --filename=testfile --size=4G --rw=randread --bs=4k --iodepth=64 --direct=1 --runtime=60 --time_based
iperf3 -c server -P 4 -t 30
wrk -t8 -c400 -d60s http://127.0.0.1:8080/
hey -n 100000 -c 200 http://127.0.0.1:8080/
stress-ng --cpu 8 --timeout 60s
```

---
