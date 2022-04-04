# gopsutil: Go 实现的 psutil

[![Test](https://github.com/shirou/gopsutil/actions/workflows/test.yml/badge.svg)](https://github.com/shirou/gopsutil/actions/workflows/test.yml) [![Coverage Status](https://coveralls.io/repos/github/shirou/gopsutil/badge.svg?branch=master)](https://coveralls.io/github/shirou/gopsutil?branch=master) [![Go Reference](https://pkg.go.dev/badge/github.com/shirou/gopsutil.svg)](https://pkg.go.dev/github.com/shirou/gopsutil)

这是对 psutil 代码的移植 (https://github.com/giampaolo/psutil)，项目是挑战在多个架构中移植所有 psutil 函数。

## v3 迁移

从 v3.20.10 开始, gopsutil v3 版本破坏了向后兼容。具体看 [v3Changes.md](_tools/v3migration/v3Changes.md) 文件。

## 版本号

gopsutil 的版本策略和大多数一样，但是自增长的版本号和 Ubuntu 类似。

例如，v2.17.04 的意思是

- v2: 主要版本号
- 17: 发布年份
- 04: 发布月份

gopsutil 旨在保持向后兼容，直到版本发生重大变化。

每月会更新版本号，但若只有少数提交，那么通常会被忽略。

## 支持的架构

- FreeBSD i386/amd64/arm
- Linux i386/amd64/arm(raspberry pi)
- Windows i386/amd64/arm/arm64
- Darwin i386/amd64
- OpenBSD amd64 (感谢 @mpfz0r!)
- Solaris amd64 (由 SmartOS/Illumos 开发和测试, 感谢
  @jen20!)

还有部分支持:

- CPU on DragonFly BSD (#893, 感谢 @gballet!)
- host on Linux RISC-V (#896, 感谢 @tklauser!)

所有工作都是在没有 cgo 的情况下将 C 结构移植到 Golang 结构来实现的。

## 使用

```go
package main

import (
    "fmt"

    "github.com/shirou/gopsutil/v3/mem"
    // "github.com/shirou/gopsutil/mem"  // 使用 v2 版本
)

func main() {
    v, _ := mem.VirtualMemory()

    // 几乎所有返回值都是结构体类型
    fmt.Printf("Total: %v, Free:%v, UsedPercent:%f%%\n", v.Total, v.Free, v.UsedPercent)

    // 转换为 JSON.String() 也是允许的
    fmt.Println(v)
}
```

下面是输出：

    Total: 3179569152, Free:284233728, UsedPercent:84.508194%
    {"total":3179569152,"available":492572672,"used":2895335424,"usedPercent":84.50819439828305, (snip...)}

你可以通过设定 `HOST_PROC` 环境变量来为 `/proc` 设置一个替代位置。

你可以通过设定 `HOST_SYS` 环境变量来为 `/sys` 设置一个替代位置。

你可以通过设定 `HOST_ETC` 环境变量来为 `/etc` 设置一个替代位置。

你可以通过设定 `HOST_VAR` 环境变量来为 `/var` 设置一个替代位置。

你可以通过设定 `HOST_RUN` 环境变量来为 `/var` 设置一个替代位置。

你可以通过设定 `HOST_DEV` 环境变量来为 `/dev` 设置一个替代位置。

你可以通过设定 `HOST_PROC_MOUNTINFO` 环境变量来为 `/proc/N/mountinfo` 设置一个替代位置。

## 文档

具体看这里 http://godoc.org/github.com/shirou/gopsutil

## 要求

- 支持 go1.16 或更新的版本。

## 更多信息

增加了一些 psutil 中没有的方法，下面提供了这些方法的信息。

- host/HostInfo() (linux)
  - Hostname
  - Uptime
  - Procs
  - OS (ex: "linux")
  - Platform (ex: "ubuntu", "arch")
  - PlatformFamily (ex: "debian")
  - PlatformVersion (ex: "Ubuntu 13.10")
  - VirtualizationSystem (ex: "LXC")
  - VirtualizationRole (ex: "guest"/"host")
- IOCounters
  - Label (linux only) The registered [device mapper
    name](https://www.kernel.org/doc/Documentation/ABI/testing/sysfs-block-dm)
- cpu/CPUInfo() (linux, freebsd)
  - CPU (ex: 0, 1, ...)
  - VendorID (ex: "GenuineIntel")
  - Family
  - Model
  - Stepping
  - PhysicalID
  - CoreID
  - Cores (ex: 2)
  - ModelName (ex: "Intel(R) Core(TM) i7-2640M CPU @ 2.80GHz")
  - Mhz
  - CacheSize
  - Flags (ex: "fpu vme de pse tsc msr pae mce cx8 ...")
  - Microcode
- load/Avg() (linux, freebsd, solaris)
  - Load1
  - Load5
  - Load15
- docker/GetDockerIDList() (linux only)
  - container id list ([]string)
- docker/CgroupCPU() (linux only)
  - user
  - system
- docker/CgroupMem() (linux only)
  - various status
- net_protocols (linux only)
  - system wide stats on network protocols (i.e IP, TCP, UDP, etc.)
  - sourced from /proc/net/snmp
- iptables nf_conntrack (linux only)
  - system wide stats on netfilter conntrack module
  - sourced from /proc/sys/net/netfilter/nf_conntrack_count

有些代码是从 Ohai 移植来的，非常感谢。

## 当前状态

- x: 能工作
- b: 也能工作，但会出问题

|name                  |Linux  |FreeBSD  |OpenBSD  |macOS   |Windows  |Solaris  |Plan 9   |
|----------------------|-------|---------|---------|--------|---------|---------|---------|
|cpu\_times            |x      |x        |x        |x       |x        |         |b        |
|cpu\_count            |x      |x        |x        |x       |x        |         |x        |
|cpu\_percent          |x      |x        |x        |x       |x        |         |         |
|cpu\_times\_percent   |x      |x        |x        |x       |x        |         |         |
|virtual\_memory       |x      |x        |x        |x       |x        | b       |x        |
|swap\_memory          |x      |x        |x        |x       |         |         |x        |
|disk\_partitions      |x      |x        |x        |x       |x        |         |         |
|disk\_io\_counters    |x      |x        |x        |        |         |         |         |
|disk\_usage           |x      |x        |x        |x       |x        |         |         |
|net\_io\_counters     |x      |x        |x        |b       |x        |         |         |
|boot\_time            |x      |x        |x        |x       |x        |         |         |
|users                 |x      |x        |x        |x       |x        |         |         |
|pids                  |x      |x        |x        |x       |x        |         |         |
|pid\_exists           |x      |x        |x        |x       |x        |         |         |
|net\_connections      |x      |         |x        |x       |         |         |         |
|net\_protocols        |x      |         |         |        |         |         |         |
|net\_if\_addrs        |       |         |         |        |         |         |         |
|net\_if\_stats        |       |         |         |        |         |         |         |
|netfilter\_conntrack  |x      |         |         |        |         |         |         |


### 进程类

|name                |Linux  |FreeBSD  |OpenBSD  |macOS  |Windows  |
|--------------------|-------|---------|---------|-------|---------|
|pid                 |x      |x        |x        |x      |x        |
|ppid                |x      |x        |x        |x      |x        |
|name                |x      |x        |x        |x      |x        |
|cmdline             |x      |x        |         |x      |x        |
|create\_time        |x      |         |         |x      |x        |
|status              |x      |x        |x        |x      |         |
|cwd                 |x      |         |         |x      |         |
|exe                 |x      |x        |x        |       |x        |
|uids                |x      |x        |x        |x      |         |
|gids                |x      |x        |x        |x      |         |
|terminal            |x      |x        |x        |       |         |
|io\_counters        |x      |x        |x        |       |x        |
|nice                |x      |x        |x        |x      |x        |
|num\_fds            |x      |         |         |       |         |
|num\_ctx\_switches  |x      |         |         |       |         |
|num\_threads        |x      |x        |x        |x      |x        |
|cpu\_times          |x      |         |         |       |x        |
|memory\_info        |x      |x        |x        |x      |x        |
|memory\_info\_ex    |x      |         |         |       |         |
|memory\_maps        |x      |         |         |       |         |
|open\_files         |x      |         |         |       |         |
|send\_signal        |x      |x        |x        |x      |         |
|suspend             |x      |x        |x        |x      |         |
|resume              |x      |x        |x        |x      |         |
|terminate           |x      |x        |x        |x      |x        |
|kill                |x      |x        |x        |x      |         |
|username            |x      |x        |x        |x      |x        |
|ionice              |       |         |         |       |         |
|rlimit              |x      |         |         |       |         |
|num\_handlers       |       |         |         |       |         |
|threads             |x      |         |         |       |         |
|cpu\_percent        |x      |         |x        |x      |         |
|cpu\_affinity       |       |         |         |       |         |
|memory\_percent     |       |         |         |       |         |
|parent              |x      |         |x        |x      |x        |
|children            |x      |x        |x        |x      |x        |
|connections         |x      |         |x        |x      |         |
|is\_running         |       |         |         |       |         |
|page\_faults        |x      |         |         |       |         |

### 原始指标

|item             |Linux  |FreeBSD  |OpenBSD  |macOS   |Windows |Solaris  |
|-----------------|-------|---------|---------|--------|--------|---------|
|**HostInfo**     |       |         |         |        |        |         |
|hostname         |x      |x        |x        |x       |x       |x        |
|uptime           |x      |x        |x        |x       |        |x        |
|process          |x      |x        |x        |        |        |x        |
|os               |x      |x        |x        |x       |x       |x        |
|platform         |x      |x        |x        |x       |        |x        |
|platformfamily   |x      |x        |x        |x       |        |x        |
|virtualization   |x      |         |         |        |        |         |
|**CPU**          |       |         |         |        |        |         |
|VendorID         |x      |x        |x        |x       |x       |x        |
|Family           |x      |x        |x        |x       |x       |x        |
|Model            |x      |x        |x        |x       |x       |x        |
|Stepping         |x      |x        |x        |x       |x       |x        |
|PhysicalID       |x      |         |         |        |        |x        |
|CoreID           |x      |         |         |        |        |x        |
|Cores            |x      |         |         |        |x       |x        |
|ModelName        |x      |x        |x        |x       |x       |x        |
|Microcode        |x      |         |         |        |        |x        |
|**LoadAvg**      |       |         |         |        |        |         |
|Load1            |x      |x        |x        |x       |        |         |
|Load5            |x      |x        |x        |x       |        |         |
|Load15           |x      |x        |x        |x       |        |         |
|**GetDockerID**  |       |         |         |        |        |         |
|container id     |x      |no       |no       |no      |no      |         |
|**CgroupsCPU**   |       |         |         |        |        |         |
|user             |x      |no       |no       |no      |no      |         |
|system           |x      |no       |no       |no      |no      |         |
|**CgroupsMem**   |       |         |         |        |        |         |
|various          |x      |no       |no       |no      |no      |         |

- future work
  - process_iter
  - wait_procs
  - Process class
    - as_dict
    - wait

## 开源协议

New BSD License (与 psutil 一致)

## 相关作品

我受到了以下伟大作品的影响：

- psutil: https://github.com/giampaolo/psutil
- dstat: https://github.com/dagwieers/dstat
- gosigar: https://github.com/cloudfoundry/gosigar/
- goprocinfo: https://github.com/c9s/goprocinfo
- go-ps: https://github.com/mitchellh/go-ps
- ohai: https://github.com/opscode/ohai/
- bosun:
  https://github.com/bosun-monitor/bosun/tree/master/cmd/scollector/collectors
- mackerel:
  https://github.com/mackerelio/mackerel-agent/tree/master/metrics

## 如何贡献

1.  Fork 它
2.  创建一个新的分支 (git checkout -b my-new-feature)
3.  提交你的更改 (git commit -am 'Add some feature')
4.  推送到新分支上 (git push origin my-new-feature)
5.  创建一个新的 PR（Pull Request）

English is not my native language, so PRs correcting grammar or spelling
are welcome and appreciated.
