# JVM flags cheatsheet

## Memory

| Flag | Meaning |
|---|---|
| -Xms / -Xmx | heap min / max (set equal to avoid resize pauses) |
| -Xss | thread stack size (1 m default on HotSpot x64) |
| -XX:MetaspaceSize | initial metaspace; raise to dodge first-cycle GC storms |
| -XX:MaxDirectMemorySize | cap for ByteBuffer.allocateDirect |
| -XX:+UseG1GC | G1 (default since JDK 9); G1MaxGCPauseMillis tunes the target |
| -XX:+UseZGC | low-pause alternative for large heaps (Xmx above 8 g or so) |

## Diagnostics

| Flag | What you get |
|---|---|
| -XX:+HeapDumpOnOutOfMemoryError | dump on OOM (pair with -XX:HeapDumpPath) |
| -Xlog:gc* | GC log (JDK 9+ unified logging) |
| -XX:+AlwaysPreTouch | touch all heap pages at startup, avoid faults in production |
| -XX:+ExitOnOutOfMemoryError | fail fast instead of limping |
| -XX:+PrintSafepointStatistics | safepoint latency forensics (needs UnlockDiagnosticVMOptions) |

## Containers

- -XX:MaxRAMPercentage=75.0 — size the heap from the cgroup limit instead of Xmx.
- -XX:ActiveProcessorCount=N — pin visible cores when the JVM over-reads the host.

## Thread dumps that matter

- jstack pid twice, 10 s apart — only stable blocked states are signal.
- jcmd pid GC.heap_info — quick heap state without a full dump.

## Common mistakes

1. Xmx smaller than the cgroup limit and MaxRAMPercentage left at default:
   both apply, the min wins, and the heap is silently smaller than expected.
2. G1 with a very large heap (400 g and up) — consider ZGC or tune the region size.
3. Eden churn from short-lived allocation spikes is normal. Chase the survivor
   promotion rate, not the raw allocation rate.
