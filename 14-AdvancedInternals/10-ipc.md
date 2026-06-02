# Inter-Process Communication

This guide covers pipes, shared memory, signals, sockets, and Linux IPC mechanisms.

IPC mechanisms let processes exchange data, synchronize, or signal events.

## 10.1 IPC Families

Linux offers many IPC choices:

- Anonymous pipes
- Named pipes (FIFOs)
- Signals
- Shared memory
- Semaphores
- Message queues
- Unix domain sockets
- Netlink
- D-Bus
- Futex-backed synchronization in shared memory

## 10.2 Anonymous Pipes

Created with `pipe()`.

Characteristics:

- Unidirectional byte stream
- Typically used between related processes
- Kernel-managed buffer

Example shell usage:

```bash
ps aux | grep nginx
```

## 10.3 Named Pipes

Created with `mkfifo`.

Useful for unrelated processes needing simple stream IPC.

## 10.4 Message Boundaries vs Streams

| Mechanism | Boundary Model |
|---|---|
| Pipe | Byte stream |
| TCP | Byte stream |
| Unix datagram socket | Message-oriented |
| POSIX message queue | Message-oriented |

## 10.5 Signals

Signals provide asynchronous notifications.

Examples:

- `SIGTERM`
- `SIGKILL`
- `SIGCHLD`
- `SIGUSR1`
- `SIGSEGV`

## 10.6 Signal Delivery Nuances

Signals may be:

- Process-directed
- Thread-directed
- Blocked and pending
- Queued for realtime signal ranges

## 10.7 Shared Memory

Shared memory provides the highest-throughput IPC because processes can access the same physical pages directly.

Mechanisms:

- POSIX shared memory
- System V shared memory
- `mmap()` shared file or `memfd`

## 10.8 Shared Memory Tradeoff

Data movement is cheap, but synchronization becomes your responsibility.

## 10.9 Semaphores

Semaphores coordinate access or count resource availability.

Linux supports:

- POSIX semaphores
- System V semaphores

## 10.10 Message Queues

POSIX and System V message queues allow message-based IPC with priorities in some variants.

## 10.11 Unix Domain Sockets

Unix sockets enable local process communication with rich semantics.

Benefits:

- Stream or datagram mode
- Credential passing in some cases
- File descriptor passing using ancillary data
- Lower overhead than TCP loopback in many scenarios

## 10.12 File Descriptor Passing

Unix domain sockets can pass open file descriptors between processes with `SCM_RIGHTS`.

This is widely used by service managers and brokers.

## 10.13 D-Bus

D-Bus is a higher-level IPC/message bus common on Linux desktops and some system services.

## 10.14 Netlink

Netlink is a kernel-to-user and user-to-kernel messaging interface used for:

- Networking configuration
- Route updates
- Audit messages
- Generic netlink families

## 10.15 `eventfd` and `signalfd`

Linux provides FD-oriented notification mechanisms:

- `eventfd` for event counters
- `signalfd` for signal delivery through file descriptors
- `timerfd` for timer events through file descriptors

## 10.16 Pipes Internals

A pipe uses kernel buffers and wait queues.

Readers sleep when empty. Writers sleep when full, unless nonblocking mode is enabled.

## 10.17 Shared Memory Example in C

```c
#include <fcntl.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <unistd.h>
#include <string.h>

int main(void)
{
    int fd = shm_open("/demo", O_CREAT | O_RDWR, 0600);
    ftruncate(fd, 4096);
    char *p = mmap(NULL, 4096, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    strcpy(p, "hello from shared memory\n");
    munmap(p, 4096);
    close(fd);
    return 0;
}
```

## 10.18 Signal Handling Caveats

Signal handlers can only safely call **async-signal-safe** functions.

This is a common source of subtle bugs.

## 10.19 Choosing an IPC Mechanism

| Mechanism | Best For |
|---|---|
| Pipe | Simple parent-child stream |
| Shared memory | High-throughput local exchange |
| Unix socket | Rich local service IPC |
| Message queue | Message-oriented local control paths |
| Signal | Notification, not bulk data |
| D-Bus | Structured system/service messaging |

## 10.20 Common IPC Problems

| Symptom | Cause |
|---|---|
| Pipe deadlock | Both sides blocking with full buffers |
| Lost wakeups | Broken synchronization design |
| Signal races | Masking/order issues |
| Shared memory corruption | Missing locking or memory ordering |

## 10.21 Section Summary

Linux IPC is a toolbox, not a single mechanism. The right choice depends on throughput, latency, reliability, topology, security, and programming model.

---
