Download Link: https://assignmentchef.com/product/solved-project-4-cpu-profiler
<br>
<h6><span style="font-size: 2.61792em; letter-spacing: -1px;"><span style="font-size: 2.61792em;">Introduction</span></span></h6>

The goal of this project is to design a CPU profiling tool. The tool will be designed as a kernel module which when loaded, keeps track of the time spent on CPU for each task.

<h1>Recommended Background Reading</h1>

<ul>

 <li><a href="https://pointer-overloading.blogspot.com/2013/09/linux-creating-entry-in-proc-file.html">Linux /proc file system</a></li>

 <li><a href="https://github.com/torvalds/linux/blob/master/fs/cifs/cifs_debug.c">Sample /proc implementation: cifs_debug.c</a></li>

 <li>Kprobe: <a href="https://github.com/torvalds/linux/blob/master/Documentation/kprobes.txt">documentation</a>, <a href="https://github.com/torvalds/linux/tree/master/samples/kprobes">examples</a></li>

 <li>x86_64 calling convention: <a href="https://en.wikipedia.org/wiki/x86_calling_conventions">documentation</a></li>

 <li>spinlock: <a href="https://github.com/torvalds/linux/blob/master/include/linux/spinlock.h">API</a></li>

 <li>Jenkins hash: <a href="https://github.com/torvalds/linux/blob/master/include/linux/jhash.h">API</a></li>

 <li>Time measurement (rdtsc): <a href="https://github.com/torvalds/linux/blob/master/arch/x86/include/asm/msr.h">API</a></li>

</ul>

<h1>Part 1. Monitoring task scheduling</h1>

In part 1, you will design a kernel module, named <em>perftop</em>, which will monitor the <em>pick_next_task_fair </em>function of Completely Fair Scheduler (CFS). To this end, You will use <em>Kprobe</em>, a debugging tool in linux kernel. With Kprobes, you can place a pre-event and post-event handlers (callback functions) on a certain kernel instruction address (similar to gdb’s breakpoint). The module will display profiling result using the <em>proc</em> file system.

Program your module in perftop.c and perftop.h (as needed). Create Makefile that support <em>all</em>, <em>clean</em>, <em>install</em> and <em>uninstall</em> rules (and more as needed), similar to that of project 2.

<h2>Part 1.1. Setup procfs</h2>

The first task is to setup a proc file (<em>procfs</em>) where the results of the profiler can be displayed.

<ul>

 <li>Review the Linux kernel documents and sample codes about <em>proc</em> file system. The links provided in the above <em>Recommended Background Reading</em> would be a good staring point.</li>

 <li>Write a kernel module named <em>perftop</em></li>

 <li>The module should create a proc file named <em>perftop</em></li>

 <li><em>cat /proc/perftop</em> should display “Hello World” Deliverables:</li>

 <li>Load <em>perftop</em> module</li>

 <li>Invoke <em>cat /proc/perftop</em></li>

 <li>Take a screenshot of the output. Name your screenshot as png</li>

</ul>

<h2>Part 1.2. Setup Kprobe</h2>

We will set up Kprobe for the CFS’s <em>pick_next_task_fair</em> function.

Tasks:

<ul>

 <li>Review the Linux kernel documents and sample codes about <em>KProbe</em>. The links provided in the above <em>Recommended Background Reading</em> would be a good staring point.</li>

 <li>Set a kprobe hook on the <em>pick_next_task_fair</em> Register a pre-event handler named <em>entry_pick_next_fair</em> and a post-event handler called <em>ret_pick_next_fair</em>.</li>

 <li>The event handler should increment its own counter, named <em>pre_count</em> and <em>post_count</em>, repsectively.</li>

 <li>The counter should be displayed by <em>cat /proc/perftop</em>.</li>

</ul>

Deliverables:

<ul>

 <li>Load <em>perftop</em> module</li>

 <li>Invoke <em>cat /proc/perftop</em> two times with some time gaps (e.g., 10 seconds).</li>

 <li>Take a screenshot of the output. Name your screenshot as png</li>

</ul>

<h2>Part 1.3. Count the Number of Context Switches</h2>

We will count the number of cases where the scheduler pick a different task to run: i.e., <em>prev task != next task</em>.

Tasks:

<ul>

 <li>Set up a kprobe hook on the <em>pick_next_task_fair</em> function (same as Part 1.2).</li>

 <li>On a pre-event handler <em>entry_pick_next_fair</em>, obtain the pointer of (prev) <em>task_struct</em> from <em>struct pt_regs *regs</em> using the register calling convention.</li>

 <li>On a post-event handler <em>ret_pick_next_fair</em>, obtain the pointer of (next) <em>task_struct</em> from <em>struct pt_regs *regs</em>. Check if the prev and next tasks are different. If so, increment the counter named <em>context_switch_count</em>.</li>

 <li>The counter should be displayed by <em>cat /proc/perftop</em>.</li>

</ul>

Deliverables:

<ul>

 <li>Load <em>perftop</em> module</li>

 <li>Invoke <em>cat /proc/perftop</em> two times with some time gaps (e.g., 10 seconds).</li>

 <li>Take a screenshot of the output. Name your screenshot as png</li>

 <li>Make a folder named with your SBU ID (e.g., 112233445), put Makefile, c, perftop.h (if any), and the screenshots perftop{1,2,3}.png files in the folder, create a single gzip-ed tarball named [SBU ID].tar.gz, and turn the gzip-ed tarball to Blackboard.</li>

</ul>

<h1>Part 2. Print 10 most scheduled tasks</h1>

In part 2, you will modify the kprobe event handlers in <em>perftop</em> to keep track of time each task spends on CPU and print the 10 most scheduled tasks using <em>proc</em>.

Preliminaries:

<ul>

 <li>We will measure time using <em>rdtsc</em> time stamp counter.</li>

 <li>Set up a hash table where a PID is used as a key and the start tsc (the time a task is scheduled on a CPU) is stored as a value.</li>

 <li>Set up a rb-tree that are ordered by the total tsc (the accumulative time) spent by a task on a CPU.</li>

</ul>

Tasks:

<ul>

 <li>On a post-event handler <em>ret_pick_next_fair</em>, if the prev and next tasks are different, we measure the time spent by each task on CPU as follows.</li>

 <li>(1) For the <em>prev</em> task: we read the current tsc and obtain the start tsc of the <em>prev</em> task from the hash table (using PID as a key). The difference between two timestamps (say, <em>elapsed time</em>) will represent the amount of the time the prev task has been scheduled on a CPU during the scheduling turn.</li>

 <li>(2) For the <em>prev</em> task: we remove the old entry from the rb-tree and add the new entry with the updated total tsc (the accumulative sum of elapsed times) to the rb-tree.</li>

 <li>(3) For the <em>next</em> task: we update the start tsc of the <em>next</em> task in the hash table with the current tsc.</li>

 <li>Modify the open function of proc file to print the top 10 most scheduled tasks. Print the PID and the time (total tsc) spent on a CPU.</li>

</ul>

Deliverables:

<ul>

 <li>Load <em>perftop</em> module</li>

 <li>Invoke <em>cat /proc/perftop</em> two times with some time gaps (e.g., 10 seconds).</li>

 <li>Take a screenshot of the output. Name your screenshot as png</li>

 <li>Make a folder named with your SBU ID (e.g., 112233445), put Makefile, c, perftop.h</li>

</ul>

<table width="120">

 <tbody>

  <tr>

   <td width="120">[SBU ID].tar.gz</td>

  </tr>

 </tbody>

</table>

(if any), and the screenshots perftop4.png file in the folder, create a single gzip-ed tarball named , and turn the gzip-ed tarball to Blackboard.