+++
title = "2024 02 04 Introduction"
description = ""
date = "2024-02-04T17:02:07-08:00"
tags = ["", ""]
categories = ["", ""]
draft = true
+++
# Introduction

I was thinking about the process of scheduling the lowestpriority tasks, colloquially called “lazy tasks”.

Let’s call the locus of control and data thatdecides what gets to run and when it gets to run the Executive Control, (EC).
What is the nature of EC?  It could be an RTOS or something else.
I’m using the term EC as a way to abstract itand not assume anything about the nature of it.
I’m assuming that the EC will be a polymorphous entity with different implementationaspects at different levels of processing.  The Atomic Execution Units, (AEU) are under the directcontrol of the EC.
The assumption isthat control passes from the EC to an AEU and then back again to the EC whenthe AEU completes.  The EC is responsiblefor selecting which AEUs to run in whatever particular order is required at thetime.
One of the differences between an AEU and a lazy task isthat the AEU is well enough defined that its processing time has somewhatdefined boundaries.
The EC can turncontrol over to something for which the amount of processing time is effectivelyunbounded, but the result will almost always be a context switch caused by aninterrupt.
One point of analysis hereshould be to measure the cost of a context switch and compare it to the workthat would need to be done to avoid unneeded context switches.
For example, currently there are 32 floatingpoint registers in an ARM Cortex M33.
The burden for a context switch would then be somewhere at least 100 memory operations along with attendant instruction fetches.  The EC should be able to gather data and exercise metrics torun efficiently without outside intervention.
Lazy tasks should be gathered in priority bands.
Lazy tasks should self-regulate to the point of calling EcYield()at appropriate times to allow lazy tasks in priority bands at it’s own level orhigher bands to run.
If there is no suchlazy task, no context switch occurs and control returns to the calling lazytask.
An EC time unit is the amount of time that serves as a countingunit for lazy task execution.
Lazy tasks record the amount of EC time units it takes oneach iteration.
Developers will use thisdata to determine a predictable run schedule for the lazy task.
For example, a lazy task may take more ECtime units on some incarnations and fewer on others.
The characteristic pattern will be determinedand used in scheduling.
Ideally, a lazytask would require some fixed number of EC time units, plus or minus some otherquantity.
The lazy task would be scheduledfor the maximum EC time units expected, plus some variance.
What needs to be avoided is a lazy task that hasa distribution where there are orders of magnitude between the smallest andlargest number of expected EC time units.
The number of EC time units required for a lazy task will beperturbed by the time required to perform interrupt processing.
Data should be gathered on the number ofinterrupts and amount of time required for handling and used to adjust the amountof time the current lazy task is allowed.
It is assumed that the assignment of lazy tasks to prioritybands is well enough understood that lazy tasks will not need to be movedbetween priority bands during execution.
This assumption needs to be verified.
In any case, this should be approached with care.
Any movement in priority bands should bebounded lest all lazy tasks end up in the same band, no matter where it is.
On the topic of context switches, it would appear that thereare two approaches.
The first approachis to perform a context save on each interrupt, run the interrupt on the ECstack, execute any attendant AEU on the EC stack, and restore the appropriate lazytask as dictated by standard priority evaluation.
The run of the lazy task in this case maythen be truncated and that would need to be recorded.
The second approach would be to run the interrupton the lazy task’s stack and when the lazy task reaches a EcYield() point, storethat lazy task context, perform AEU processing, then restore a lazy task context that is priorityappropriate.
The benefit of the firstapproach is that it will probably guarantee lower latency and higher throughputat the expense of more context saves and restores.
That cost benefit analysis needs to measuredand documented.
It should be clear thatone difference between the two approaches is that AEU processing would occur duringlazy task context switches in the latter case, and would be driven by interruptsin the former.
It should also be clearthat some combination of these two approaches could be implemented.
For example, some interrupt handlers couldconditionally call EcStoreLtContextAndBeginAeuProcessing() on exit.
The EC time unit duration should be selected so it has thefollowing characteristics:1)     A multiple of EC time units can take the placeof spin loops.
2)     A multiple of EC time units will be less thanthe minimum latency period for the shortest latency required by a lazy task.
  If there is a latency requirement that issufficiently short that it cannot be handled in three EC time units, it shouldbe handled in an AEU.
3)     AEU should run to completion in much less thanone EC time unit.
 
