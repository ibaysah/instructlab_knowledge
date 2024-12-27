Architect for Error Reporting and Fault Isolation
It is not feasible to detect or isolate every possible fault or combination of faults that a server
might experience, though it is important to invest in error detection and build a coherent
architecture for how errors are reported and faults isolated. The sub-section on processor and
memory error detection and fault isolation details the IBM Power approach for these system
elements.
It should be pointed out error detection may seem like a well understood and universal hardware
design goal. However, it is not always the goal of every computer sub-system
design. Hypothetically, for instance, graphics processing units (GPU s) whose primary purpose is
rendering graphics in non-critical applications may have options for turning off certain error
checking (such as ECC in memory) to allow for better performance. The expectation in such case
is that there are applications where a single dropped pixel on a screen is of no real importance,
and a solid fault is only an issue if it is noticed.
In general, I/O adapters may also have less hardware error detection capability where they can
rely on a software protocol to detect and recover from faults when such protocols are used.

Leverage Technology and Design for Soft Error Management
In a very real sense errors detected can take several forms. The most obvious is a functional fault
in the hardware – a silicon defect, or a worn component that over time has failed.
Another kind of failure is what is broadly classified as a soft error. Soft errors are faults that
occur in a system and are either occasional events inherent in the design or temporary faults that
are due to an external cause.
Data cells in caches and memory, for example, may have a bit-value temporarily upset by an
external event such as caused by a cosmic ray generated particle. Logic in processors cores can
also be subject to soft errors where a latch may also flip due to a particle strike or similar event.
Busses transmitting data may experience soft errors due to clock drift or electronic noise.
The susceptibility to soft errors in a processor or memory subsystem is very much dependent on
the design and technology used in these devices. This should be the first line of defense.
Choosing latches that are less susceptible to upsets due to cosmic ray events was discussed
extensively in previous whitepapers for example.
Methods for interleaving data so that two adjacent bits in array flipping won’t cause undetected
multi-bit flips in a data word is another design technique that might be used.
Ultimately when data is critical, detecting soft error events that occur needs to be done
immediately, in-line to avoid relying on bad data because periodic diagnostics is insufficient to
catch an intermittent problem before damage is done.
The simplest approach to detecting many soft error events may simply be having parity
protection on data which can detect a single bit flip. When such simple single bit error detection
is deployed, however, the impact of a single bit upset is bad data. Discovering bad data without
being able to correct it will result in termination of an application, or even a system so long as
data correctness is important.
To prevent such a soft error from having a system impact it is necessary not simply to detect a
bit flip, but also to correct. This requires more hardware than simple parity. It has become
common now to deploy a bit correcting error correction code (ECC) in caches that can contain modified
data. Because such flips can occur in more than just caches, however, such ECC codes are
widely deployed in Power10 processors in critical areas on busses, caches and so forth.
Protecting a processor from more than just data errors requires more than just ECC checking and
correction. CRC checking with a retry capability is used on a number of busses, for example.
When a fault is noticed maximum protection is achieved when not only is a fault noticed but
noticed quickly enough to allow processor operations to be retried. Where retry is successful, as
would be expected for temporary events, system operation continues without application
outages.

Deploy Strategic Spare Capacity to Self-Heal Hardware
Techniques that protect against soft errors are of limited protection against solid faults due to a
real hardware failure. A single bit error in a cache, for example can be continually corrected by
most ECC codes that allow double-bit detection and single bit correction.
However, if a solid fault is continually being corrected, the second fault that occurs will typically
cause data that is not correctable. This would result in the need to terminate, at least, whatever is
using the data.
In many system designs, when a solid fault occurs in something like a processor cache, the
management software on the system (the hypervisor or OS) may be signaled to migrate the
failing hardware off the system.
This is called predictive deallocation. Successful predictive deallocation may allow for the
system to continue to operate without an outage. To restore full capacity to the system, however,
the failed component still needs to be replaced, resulting in a service action.
Within Power, the general philosophy is to go beyond simple predictive deallocation by
incorporating strategic sparing or micro-level deallocation of components so that when a hard
failure that impacts only a portion of the sub-system occurs, full error detection capabilities can
be restored without the need to replace the part that failed.
Examples include a spare data lane on a bus, a spare bit-line in a cache, having caches split up
into multiple small sections that can be deallocated, or a spare DRAM module on a DIMM.
With the general category of self-healing can be the use of spares. Redundancy can also be
deployed to avoid outages. The concepts are related but there are differences between
redundancy and use of spares in the Power approach.

Redundant Definition
Redundancy is generally a means of continuing operation in the presence of certain faults by
providing more components/capacity than is needed to avoid outages but where a service action
will be taken to replace the failed component after a fault.
Sometimes redundant components are not actively in use unless a failure occurs. For example, a
processor may only actively use one clock source at a time even when redundant clock sources
are provided.
In contrast, fans and power supplies are typically all active in a system. If a system is said to
have “n+1” fan redundancy, for example, all “n+1” fans will normally be active in a system
Power10 processor-based systems RAS Page 37
absence a failure. If a fan fails occurs, the system will run with “n” fans. In cases where there are
fans or power supply failures, power and thermal management code may compensate by
increasing fan speed or making other adjustments according to operating conditions per power
management mode and power/thermal management policy.

Spare Definition
A spare component is similar in nature though when a “spare is successfully used” the system
can continue to operate without the need to replace the component.
As an example, for voltage regulator output modules, if five output phases are needed to
maintain the power needed at the given voltage level, seven could be deployed initially. It would
take the failure of three phases to cause an outage.
If on the first phase failure, the system continues to operate, and no call out is made for repair,
the first failing phase would be considered spare. After the failure (spare is said to be used), the
VRM could experience another phase failure with no outage. This maintains the required n+1
redundancy. Should a second phase fail, a “redundant” phase would then have been said to fail
and a call-out for repair would be made.

Focus on OS Independence
Because IBM Power has long been designed to support multiple operating systems, the hardware
RAS design is intended to allow the hardware to take care of the hardware largely independent of
any operating system involvement in the error detection or fault isolation (excluding I/O adapters
and devices for the moment.)
To a significant degree this error handling is contained within the processor hardware itself.
However, service diagnostics firmware, depending on the error, may aid in the recovery. When
fully virtualized, specific OS involvement in such tasks as migrating off a predictively failed
component can also be performed transparent to the OS.
The PowerVM hypervisor is capable of creating logical partitions with virtualized processor and
memory resources. When these resources are virtualized by the hypervisor, the hypervisor has
the capability of deallocating fractional resources from each partition when necessary to remove
a component such a processor core or logical memory block (LMB).
When an I/O device is directly under the control of the OS, the error handling of the device is the
device driver responsibility. However, I/O can be virtualized through the VIOS offering meaning
that I/O redundancy can be achieved independent of the OS.
Build System Level RAS Rather Than Just Processor and Memory RAS
IBM builds with the understanding that every item that can fail in a system is a potential source
of outage.
While building a strong base of availability for the computational elements such as the
processors and memory is important, it is hardly sufficient to achieve application availability.
The failure of a fan, a power supply, a voltage regulator, or I/O adapter might be more likely
than the failure of a processor module designed and manufactured for reliability.
Power10 processor-based systems RAS Page 38
Scale-out servers will maintain redundancy in the power and cooling subsystems to avoid system
outages due to common failures in those areas. Concurrent repair of these components is also
provided.
For the Enterprise system, a higher investment in redundancy is made. The Power E1080, for
example is designed from the start with the expectation that the system must be largely shielded
from the failure of these other components causing persistent system unavailability;
incorporating substantial redundancy within the service infrastructure (such as redundant service
processors, redundant processor boot images, and so forth.) An emphasis on the reliability of
components themselves are highly reliable and meant to last.
This level of RAS investment extends beyond what is expected and often what is seen in other
server designs. For example, at the system level such selective sparing may include such
elements as a spare voltage phase within a voltage regulator module.
Error Reporting and Handling

First Failure Data Capture Architecture
Power processor-based systems are designed to handle multiple software environments
including a variety of operating systems. This motivates a design where the reliability and
response to faults is not relegated to an operating system.
Further, the error detection and fault isolation capabilities are intended to enable retry and other
mechanisms to avoid outages due to soft errors and to allow for use of self-healing features. This
requires a very detailed approach to error detection.
This approach is beneficial to systems as they are deployed by end-users, but also has benefits in
the design, simulation, and manufacturing test of systems as well.
Putting this level of RAS into the hardware cannot be an after-thought. It must be integral to the
design from the beginning, as part of an overall system architecture for managing errors.
Therefore, during the architecture and design of a processor, IBM places a considerable
emphasis on developing structures within it specifically for error detection and fault isolation.
Each subsystem in the processor hardware has registers devoted to collecting and reporting fault
information as they occur.
The exact number of checkers and type of mechanisms isn’t as important as is the point that the
processor is designed for very detailed error checking; much more than is required simply to
report during run-time that a fault has occurred.
All these errors feed a data reporting structure within the processor. There are registers that
collect the error information. When an error occurs, that event typically results in the generation
of an interrupt.
The error detection and fault isolation capabilities maximize the ability to categorize errors by
severity and handle faults with the minimum impact possible. Such a structure for error handling
can be abstractly illustrated by the figure below and is discussed throughout the rest of this
section.

First Failure Data Capture Analysis (Processor Runtime Diagnostics)
The first failure data capture design is meant for catching faults during run-time as they occur
and provide isolation, mitigations and other actions at the time of detection. To provide full
isolation, and take appropriate service actions, code is run which accesses the fault information
available. This code is known as Processor Runtime Diagnostics (PRD). PRD is different from
RPD which is described in the next section.
Ideally this code primarily handles recoverable errors including orchestrating the implementation
of certain “self-healing” features such as use of spare DRAM modules in memory, purging and
deleting cache lines, using spare processor fabric bus lanes, and so forth.
Code within a hypervisor does have control over certain system virtualized functions,
particularly as it relates to I/O including the PCIe controller and certain shared processor
accelerators. Generally, errors in these areas are signaled to the hypervisor.
In addition, there is still a reporting mechanism for what amounts to the more traditional
machine-check or checkstop handling.
In a Power7 generation server, PRD and other service code was all run within the dedicated
service processor used to manage these systems. The dedicated service processor was in charge
of the IPL process used to initialize the hardware and bring the servers up to the state where the
hypervisor could begin to run. The dedicated service processor was also in charge, as previously
described, to run the PRD code during normal operation.
In the rare event that a system outage resulted from a problem, the service processor had access
not only to the basic error information stating what kind of fault occurred, but also access to
considerable information about the state of the system hardware – the arrays and data structures
that represent the state of each processing unit in the system, and additional debug and trace
arrays that could be used to further understand the root cause of faults.
Even if a severe fault caused system termination, this access provided the means for the service
processor to determine the root cause of the problem, deallocate the failed components, and
allow the system to restart with failed components removed from the configuration.
Power8 gained a Self-Boot-Engine which allowed processors to run code and boot using the
Power8 processors themselves to speed up the process and provide for parallelization across
multiple nodes in the high-end system. During the initial stages of the IPL process, the boot
engine code itself handled certain errors and the PRD code ran as an application at later stages if
necessary.
In Power9 the design has changed further so that during normal operation the PRD code itself
runs in a special hypervisor-partition under the management of the hypervisor. This has the
advantage of continuing to allow the PRD code to run even if the service processor is non-
functional (important in non-redundant environments). Should the code running fail, the
hypervisor can restart the partition (reloading and restarting the PRD.)
The system service processors are also still monitored at run-time by the hypervisor code and can
report errors if the service processors are not communicating. The Power10 based servers
maintain the Power9 design.

Periodic Processor Exerciser/Diagnostics Program (Runtime Processor Diagnostics)
As hyperscalers consolidate more and more computing power some have expressed a concern
about the need to run periodic diagnostics for processors in addition to what is provided by the
built-in error detection and fault isolation capabilities. Such diagnostics might better isolate
faults that would otherwise be detected but poorly isolated. They may also detect faults that
otherwise were not caught by hardware checkers. In such a case, since the diagnostics are only
run periodically, incorrect operation or results could have occurred before the periodic
diagnostics find an issue.
Google published an article talking about software and hardware defenses against the impact of
faults ( https://support.google.com/cloud/answer/10759085?hl=en). The document also describes
a set of periodic diagnostics exercises for CPUs called, CPU Check, “focusing primarily on the
x86_64 architecture.”
Intel has released a diagnostic tool
(https://www.intel.com/content/www/us/en/support/articles/000005567/processors.html) that can
be downloaded and run as an application in an operating system to, among other functions,
“Verify the functionality of all the cores of Intel® Processor.”
Power customers may ask whether IBM has similar periodic diagnostic capabilities.
The possible exposure to failed hardware and the detectability can depend on the method of
manufacture and test as well as the underlying micro-architecture of the processor including the
capabilities of any First Failure Data Capture architecture.
While First Failure Data Capture (catching faults at the source where fault isolation and
mitigation can best be performed) is considered vital to the Power approach to RAS, around the
mid-2000s, a limited set of diagnostics tests, focused primarily on floating point calculations,
were available to run periodically on Power servers. These tests were intended to complement
the FFDC approach.
While the processor architecture, FFDC and recovery techniques were refined over several
subsequent generations, these added tests were still available in each generation through Power9.
In Power9 these diagnostics can be enabled by the customers choosing options to execute the
tests on a periodic basis, staggering the execution over a period of time, or be set to run at a
particular time.
During the development and functional test of Power servers, and during the manufacturing of
the servers, IBM runs a diagnostics exerciser suite capable of exercising hardware cores.
To more directly align any periodic processor core diagnostics with what is used during test and
manufacturing, starting with the FW 1050 release for Power10 servers, Power servers
implemented a new run-time processor diagnostic (RPD) capability for periodic testing of the
functionality of processor cores. RPD uses functions from the previously mentioned test code
base with expanded coverage over the legacy periodic diagnostic capability aligned with the
Power10 processor capabilities. These tests run within a special partition under PowerVM
control and allow for testing processor cores in small time-slices during system operation.
Unlike the approach other vendors may recommend where such diagnostics would have to be
downloaded and executed in each partition, the Power10 processor diagnostics running under the
PowerVM can test cores without customer intervention beyond selecting the desired method of
operation, and regardless of how the system is partitioned. There is no need to have a copy
running in each VM of the system.
Using run-time processor diagnostics to exercise cores can result in faults handled by the First
Failure Data Capture capabilities of the processor design which will be treated according to the
previously outlined FFDC processor error handling approach.
Should the run-time diagnostics itself detect and isolate a core fault, an SRC will be posted as a
service actionable event. (B7005400 for Power9 and B7005194 for Power10). This will be
handled as an unrecoverable processor core error.
Further explanation for the SRCs: Background diagnostics have detected an uncorrectable error
on a processor core. The failing processor core has been marked defective. Prior to detecting
this failure, the failing processor may have been assigned to a running partition and it is
recommended to run the appropriate application data consistency checks. Please contact your
next level of support with any questions.
The server operator can control if and how the runtime diagnostics are used. To take advantage
of these periodic processor diagnostics in Power9 and Power10 with the FW 1050 release, or
later, customers should verify that the function is enabled with the option desired.
Power10 processor-based systems RAS Page 42
For Power10, RPD runs in a special service partition created by the PowerVM hypervisor. In the
background, using limited cycles, PowerVM can assign processor cores one at a time to the
service partition to assess for faults while customer workload is running.
From the service processor menu, customers can select one of the following RPD operation
modes:
1. Run Now – Begin testing processors and end once all processors have been tested.
2. Staggered – Periodically test all the processors and begin again when finished.
3. Scheduled – Periodically test processors only during a scheduled time window each day.
4. Disabled – Do not run
Generally, Power10 servers shipping with RPD will have RPD enabled by default in the
Staggered mode. However, Because RPD requires system resources, servers with less than
128G of installed memory will default to Disabled mode. Depending on the system
configuration and the RPD mode selected, the testing may take multiple days to exercise all
processor cores.
PowerVM Partitioning and Outages
The PowerVM hypervisor provides logical partitioning allowing multiple instances of an
operating system to run in a server. At a high level, a server with PowerVM runs with a single
copy of the PowerVM hypervisor regardless of the number of CEC nodes or partitions.
The PowerVM hypervisor uses a distributed model across the server’s processor and memory
resources. In this approach some individual hypervisor code threads may be started and
terminated as needed when a hypervisor resource is required. Ideally when a partition needs to
access a hypervisor resource, a core that was running the partition will then run a hypervisor
code thread.
Certain faults that might impact a PowerVM thread will result a system outage if they should
occur. This can be by PowerVM termination or by the hardware determining that for, PowerVM
integrity, the system will need to checkstop.
The design cannot be viewed as a physical partitioning approach. There are not multiple
independent PowerVM hypervisors running in a system. If for fault isolation purposes, it is
desired to have multiple instances of PowerVM and hence multiple physical partitions, separate
systems can be used.
Not designing a single system to have multiple physical partitions reflects the belief that the best
availability can be achieved if each physical partition runs in completely separate hardware.
Otherwise, there is a concern that when resources for separate physical partitions come together
in a system, even with redundancy, there can be some common access point and the possibility
of a “common mode” fault that impacts the entire system.

