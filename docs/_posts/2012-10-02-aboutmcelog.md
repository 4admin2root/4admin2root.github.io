---
layout:     post
title:      aboutmcelog
date:       2012-10-02
tags: [cnblogs]
---
[MCE LOG]

If you are seeing messages in my system logs that state "Machine Check Event logged" this could be an indication of a hardware problem or failure.

What are Machine Check Exceptions (or MCE)?

A machine check exception is an error dedected by your system's processor. There are 2 major types of MCE errors, a notice or warning error, and a fatal execption. The warning will be logged by a "Machine Check Event logged" notice in your system logs, and can be later viewed via some Linux utilities. A fatal MCE will cause the machine to stop responding and the details of the MCE will be printed out to the system's console.

What causes MCE errors?

There most common reason for MCE events to occur are:

Memory errors or Error Correction Code (ECC) problems

Inadequate cooling / processor over-heating

Cache errors in the processor or hardware

How do I find out what the errors mean?

If you see the message "Machine Check Events logged" on your console or in your system logs, then you can run the mcelog command to read the message from the kernel. Once you run mcelog you will not be able to re-run it to see the error, so it's best to output the text to a file so you can further analyize it. For example:

root@localhost:/root> /usr/sbin/mcelog > mcelog.outSome systems do this for you on a regular basis and send the output to the file /var/log/mcelog . So if you see the "Machine Check Events logged" message but mcelog does not return any data, please look /var/log/mcelog.

The output received may not always be easy to understand. If you have any questions about the decoded error message please create a support ticket and we will help analyize the problem.

What if I get a fatal machine check event that causes my machine to stop responding?

These errors are almost always caused by faulty hardware. Please capture the mce message and you can later run it through the mcelog program once the machine is back up. Here's an example of a message you might see:

CPU 1: Machine Check Exception:                4 Bank 4:  f600200137080813TSC b0ce27165dd3 ADDR 180ee1b40Paste or type the error message into a file, and then run it through the mcelog for example:

root@localhost:/root> /usr/sbin/mcelog --k8 --ascii < myerrorUse the --k8 option if you are using an AMD Opteron or Athlon 64 processor, or substitute it for --p4 for a Pentium 4 or Xeon. Here is the output from the previous mce error:

HARDWARE ERROR. This is *NOT* a software problem!Please contact your hardware vendorCPU 1 4 northbridge TSC b0ce27165dd3 Northbridge Chipkill ECC error Chipkill ECC syndrome = 3700 bit32 = err cpu0 bit45 = uncorrected ecc error bit57 = processor context corrupt bit61 = error uncorrected bit62 = error overflow (multiple errors) bus error 'local node origin, request didn't time out generic read mem transaction memory access, level generic'STATUS f600200137080813 MCGSTATUS 4

This indicates that an uncorrected ECC error occured. This indicates that one of your memory modules has failed. For further analysis and please submit a support ticket with the complete MCE error message and the output of mcelog.
