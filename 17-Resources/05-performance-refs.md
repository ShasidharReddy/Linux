# Performance References

This guide collects performance engineering references, benchmark resources, and tuning guides.

## Performance

<table>
  <thead>
    <tr>
      <th>Resource</th>
      <th>URL</th>
      <th>Purpose/Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Brendan Gregg</td>
      <td>https://www.brendangregg.com/</td>
      <td>The go-to site for Linux performance engineering, observability, and practical troubleshooting methods.</td>
    </tr>
    <tr>
      <td>USE Method</td>
      <td>https://www.brendangregg.com/usemethod.html</td>
      <td>A fast framework for checking utilization, saturation, and errors across every major resource.</td>
    </tr>
    <tr>
      <td>Flame Graphs</td>
      <td>https://www.brendangregg.com/flamegraphs.html</td>
      <td>Explains one of the most powerful ways to visualize CPU stack samples and hotspots.</td>
    </tr>
    <tr>
      <td>Linux Perf Wiki</td>
      <td>https://perfwiki.github.io/main/</td>
      <td>Good reference for perf tooling, event sampling, and performance counter analysis.</td>
    </tr>
    <tr>
      <td>Phoronix Test Suite</td>
      <td>https://www.phoronix-test-suite.com/</td>
      <td>Useful for reproducible benchmarking and comparative performance testing across systems.</td>
    </tr>
    <tr>
      <td>perf-tools</td>
      <td>https://github.com/brendangregg/perf-tools</td>
      <td>A practical toolbox of scripts for tracing latency, scheduling, and I/O behavior quickly.</td>
    </tr>
    <tr>
      <td>BCC</td>
      <td>https://github.com/iovisor/bcc</td>
      <td>Provides eBPF-based tracing tools that make kernel and application analysis dramatically easier.</td>
    </tr>
    <tr>
      <td>bpftrace</td>
      <td>https://bpftrace.org/</td>
      <td>A concise tracing language for eBPF that speeds up ad hoc performance investigations.</td>
    </tr>
    <tr>
      <td>Performance Co-Pilot Docs</td>
      <td>https://pcp.readthedocs.io/</td>
      <td>Strong for time-series performance telemetry, historical analysis, and long-lived system profiling.</td>
    </tr>
    <tr>
      <td>fio Documentation</td>
      <td>https://fio.readthedocs.io/</td>
      <td>Essential for designing realistic storage and filesystem benchmarks instead of guesswork tests.</td>
    </tr>
    <tr>
      <td>stress-ng</td>
      <td>https://wiki.ubuntu.com/Kernel/Reference/stress-ng</td>
      <td>Useful for safely stressing CPU, memory, and I/O paths to validate resilience and tuning.</td>
    </tr>
    <tr>
      <td>iperf3</td>
      <td>https://software.es.net/iperf/</td>
      <td>The standard network throughput test tool for link validation and performance comparisons.</td>
    </tr>
  </tbody>
</table>
