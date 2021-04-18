---
layout:     post
title:      转：RedHatEnterpriseLinuxsystemsusingtheTSCclocksourcerebootsorpanicswhen'sched_clock()'overflowsafteranuptimeof208.5days
date:       2014-04-29
---
虽说是老贴了，但是最近还接触到

原文：https://access.redhat.com/site/solutions/68466

## 问题

- Linux Kernel panics when `sched_clock()` overflows after an uptime of around 208.5 days
- Red Hat Enterprise Linux system reboots with sched_clock() overflow after an uptime of around 208.5 days
- This symptom may happen on a system using the Time Stamp Counter (TSC) clock source
- Some processes may generate following log message:

```
<code>BUG: soft lockup - CPU#N stuck for 4278190091s!
</code>
```

## 环境

<li>Red Hat Enterprise Linux (RHEL) 6
<ul>
- Red Hat Enterprise Linux 6.0 (some kernels)
- Red Hat Enterprise Linux 6.1 (some kernels)
- Red Hat Enterprise Linux 6.2 (some kernels)
- With TSC clock source

- Red Hat Enterprise Linux 5.0
- Red Hat Enterprise Linux 5.1
- Red Hat Enterprise Linux 5.2
- Red Hat Enterprise Linux 5.3 (some kernels)
- Red Hat Enterprise Linux 5.4
- Red Hat Enterprise Linux 5.5
- Red Hat Enterprise Linux 5.6 (some kernels)
- Red Hat Enterprise Linux 5.7
- Red Hat Enterprise Linux 5.8 (some kernels)
- With TSC clock source

## 决议

<li>**Red Hat Enterprise Linux 6**
<ul>
<li>RHEL 6.x
<ul>
- Update to `kernel-2.6.32-279.el6` or later ([RHSA-2012-0862](http://rhn.redhat.com/errata/RHSA-2012-0862.html))
- **This kernel is already part of Red Hat Enterprise Linux 6.3 GA**

- Update to `kernel-2.6.32-220.4.2.el6` or later ([RHBA-2012-0124](http://rhn.redhat.com/errata/RHBA-2012-0124.html))

- Update to `kernel-2.6.32-131.26.1.el6` or later ([RHBA-2012-0424](http://rhn.redhat.com/errata/RHBA-2012-0424.html))

<li>RHEL 5.x
<ul>
- Update to `kernel-2.6.18-348.el5` or later ([RHBA-2013-0006](http://rhn.redhat.com/errata/RHBA-2013-0006.html))
- **Red Hat Enterprise Linux 5.9 GA and later already contain this fix**

- Update to `kernel-2.6.18-308.11.1.el5` or later ([RHSA-2012-1061](http://rhn.redhat.com/errata/RHSA-2012-1061.html))

- Update to `kernel-2.6.18-238.40.1.el5` or later ([RHSA-2012-1087](http://rhn.redhat.com/errata/RHSA-2012-1087.html))

- Update to `kernel-2.6.18-128.39.1.el5` or later ([RHBA-2012-1093](http://rhn.redhat.com/errata/RHBA-2012-1093.html))

<li>RHEL 5.x
<ul>
- Update to `kernel-2.6.18-348.el5` or later ([RHBA-2013-0006](http://rhn.redhat.com/errata/RHBA-2013-0006.html))
- **Red Hat Enterprise Linux 5.9 GA and later already contain this fix**

- Update to `kernel-2.6.18-308.13.1.el5` or later ([RHSA-2012-1174](http://rhn.redhat.com/errata/RHSA-2012-1174.html))

- Update to kernel-2.6.18-238.40.1.el5 or later ([RHSA-2012-1087](http://rhn.redhat.com/errata/RHSA-2012-1087.html))

- Update to kernel-2.6.18-128.39.1.el5 or later ([RHBA-2012-1093](http://rhn.redhat.com/errata/RHBA-2012-1093.html))

<li>Red Hat Enterprise MRG 1.3 Realtime kernel
<ul>
- Update to kernel kernel-rt-2.6.33.9-rt31.86.el5rt or later ([RHBA-2013:0927](http://rhn.redhat.com/errata/RHBA-2013-0927.html))

## 根源

<li>
An insufficiently designed calculation in the CPU accelerator in the previous kernel caused an arithmetic overflow in the `sched_clock()` function
<ul>
- This overflow led to a kernel panic or any other unpredictable trouble on the systems using the TSC clock source
- This problem will occur only when system uptime reaches or exceeds 208.5 days
- This update corrects the aforementioned calculation so that this arithmetic overflow and kernel panic can no longer occur under these circumstances

On RHEL5, this problem is a timing issue and is very unlikely to be encountered.

- The TSC is a fast access clock, whereas the HPET and PMTimer are both slow access clocks
- Using notsc would be a significant performance hit
- In RHEL5, the affected `sched_clock()` uses the TSC regardless of clock source selection.
- Also, in some situation, the system may hit this issue even if you set notsc to current_clocksource.

## 诊断步骤

<li>
Note: this issue could occur in numerous locations that deal with time in the kernel
<ul>
- For example, a user running a non-Red Hat kernel could have a kernel panic with a soft lockup in `__ticket_spin_lock`

The system must be booting a kernel that is a version prior to releases mentioned in the "Resolution" field
