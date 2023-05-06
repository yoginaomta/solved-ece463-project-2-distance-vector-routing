Download Link: https://assignmentchef.com/product/solved-ece463-project-2-distance-vector-routing
<br>
<strong>Overview:</strong>

The goal of this project is to implement a simplified version of the path vector protocol.

<strong>Path Vector Routing:</strong>

In a distance vector routing protocol, each router tries to find the shortest path to all other routers in the system in a distributed manner. For this lab, you will be implementing the Path Vector variant. The highlights are summarized below:

<ul>

 <li><strong>Routing Table</strong>: Each router A maintains a routing table that includes one entry for every router in the system including itself. In a distance vector routing protocol, each entry consists of the following fields: (i) the remote router id (say B); (ii) the cost of the path to router B; and (iii) the next hop to get to router B. In a Path Vector implementation, however, it is necessary to also include the full path required to reach the destination. For example, say A wants to send a packet to B. If this packet must be forwarded from A to C, from C to D, and then from D to B, A’s routing table must contain the full path A → C → D →</li>

 <li><strong>Initialization (bootstrap): </strong>The router is initialized with a set of neighbors and the cost of the links to the neighbors.</li>

 <li><strong>Routing Update Messages: </strong>Each router periodically exchanges routing update messages with each of its neighbors.

  <ul>

   <li>When router A sends a routing update message to router B, it must for each router C (that exists in A’s routing table) indicate the cost, next hop, and full path that it uses to reach C. On receiving the update from A, B updates its table, for each router C that is being advertised by router A, in the following fashion: o If router B does not have an entr y for C, then B must add an entr y for C. o If it has an entr y, then it uses the below two rules.</li>

   <li><strong>Path Vector Rule</strong>: When B receives the update message from A, it checks if A advertises B’s router id an ywhere in the path vector for C. If so, the update pertaining to entr y C is ignored. If not, B’s route to C is updated if it is possible to obtain a better route b y going through A. This is a generalization of the split horizon rule.</li>

   <li><strong>Forced update rule: </strong>When B receives an update message from A, if B alread y uses A as next hop to reach C, and A now advertises a higher cost to C, then B must update its cost to reflect the higher cost.</li>

   <li><strong>(</strong><em>Note</em>: You may assume that only periodic updates are to be supported. No triggered updates need to be implemented)</li>

  </ul></li>

 <li><strong>Router Failures: </strong>Each router must check that it is receiving updates from its neighbors on a regular basis. If no updates have been received from a neighboring router for a certain amount of time, the link between them is marked as inactive. The normal path vector operations will eventuall y lead to finding alternate paths to replace the invalidated routes.</li>

</ul>




<strong>Implementation</strong><strong>:</strong>

In your implementation, each of the routers will run as a user-level process and each router binds to a distinct UDP port. The <strong><em>Network Emulator</em></strong>, which will be given to you, binds to its own distinct UDP port as well. The Network Emulator maintains information regarding the entire network topolog y, provides each router with bootstrap information, and helps achieve communication between routers. The details regarding the interaction between the Network Emulator, and Router Processes are summarized below:

<strong><em>Network Emulator [you do not have to implement this]</em></strong><em>:</em>

<ul>

 <li>The Network emulator reads a configuration file that specifies the number of routers and links in the topology, information regarding which routers are connected b y links, and costs of the links. The exact format of the configuration file is specified in a separate section.</li>

</ul>




<strong><em>Routers’ Initialization:</em></strong>

<ul>

 <li>When a router (to be implemented in file <strong>c</strong>) starts up, it sends an INIT_REQUEST message to the Network Emulator, which includes only its router-id. The router ’s ID should be between 0 and <strong><em>MAX_ROUTERS </em></strong><em>– 1 </em>, where <strong><em>MAX_ROUTERS </em></strong>is the maximum number of routers that the s ystem can support. This ID is fed by you in the command prompt as an argument when you run your router binary so <strong>make sure you supply consistent values</strong>.</li>

 <li>The Network Emulator waits until receiving an INIT_REQUEST from <strong>ALL </strong>routers (to ensure all routers are alive before exchanging messages), after which it sends each router an</li>

</ul>

INIT_RESPONSE message that includes information regarding the neighbors of the router, and the cost of the links to the neighbors. The Network Emulator stores a mapping between the router id and router port so it can properl y forward an y packet tagged with a destination router id.




<strong><em>Routers’ Failures:</em></strong>

<ul>

 <li>It is possible that a router fails (or the router process is explicitly killed). If no updates have been received b y a router <strong><em>A </em></strong>from a neighbor <strong><em>B </em></strong>for <strong>FAILURE_DETECTION </strong>seconds, then, the router <strong><em>A </em></strong>marks the link to <strong><em>B </em></strong>as inactive. <strong><em>A </em></strong>must then modify its table to indicate that an y route for which <strong><em>B </em></strong>is included in the full path is no longer valid and has infinite cost.</li>

 <li>The Network Emulator   is   implemented   such   that   it   responds   with   the   appropriate INIT_RESPONSE whenever the dead (killed) router is restarted at a later point.</li>

 <li>Upon the receipt of INIT_RESPONSE, a router initializes its routing table with costs to neighbor routers. The known entries of the routing table will grow as the router gains knowledge of other non-neighboring routers in the s ystem.</li>

</ul>




<strong><em>Routers’ Updates:</em></strong>

Each router periodicall y, every UPDATE_INTERVAL seconds, sends an RT_UPDATE message, including its complete routing table, to neighboring routers. To achieve this, each router sends its RT_UPDATE message to the Network Emulator. The Network Emulator in turn checks <strong>only </strong>the destination id of the packet and forwards the message to the appropriate destination router. The emulator itself does not duplicate an y message; it simply ensures an incoming packet is sent to the correct neighbor of the sender.




<strong><em>Convergence:</em></strong>

We consider a routing table to have converged if it has not been modified for CONVERGE_TIMEOUT seconds, even though several update messages may have been received.

<strong>Packet Types and Formats:</strong>

The following packet types must be exchanged:

<table width="690">

 <tbody>

  <tr>

   <td width="140"> <strong>INIT_REQUEST</strong></td>

   <td width="550">Sent by a router to the Network Emulator when it starts up. The router sends its id, and requests information regarding neighboring routers and link costs</td>

  </tr>

  <tr>

   <td width="140"><strong>INIT_RESPONSE</strong></td>

   <td width="550">Sent by the Network Emulator to the router in response to theINIT_REQUEST, containing information regarding the router ’s neighbors and link costs</td>

  </tr>

  <tr>

   <td width="140"><strong>RT_UPDATE</strong></td>

   <td width="550">Each router periodically sends a route update message to its neighbors. Note that these messages are sent to the Network Emulator, which in turn forwards to the appropriate router. A route update message sent by router A to router B includes an entr y for every router in the network that A knows, and for eachentr y, the appropriate path vector information such as the cost, next hop, and fullpath</td>

  </tr>

 </tbody>

</table>

For detailed specifications regarding the format of each of these messages, please refer to the header file <strong>ne.h </strong>that we are providing you with.




<strong>Binaries and Source Files:</strong>

To start this lab, go to blackboard and download the tar file lab2-files.

<ul>

 <li><strong>Files provided to you</strong>:

  <ul>

   <li>router – The <em>Router </em>binary file based on our solution o ne – The <em>Network Emulator </em>binary file</li>

   <li>h – Header file that defines functions to manipulate the routing table o ne.h – Header file that defines packet structures &amp; functions to perform endian conversions</li>

   <li>c – Skeleton file including function declarations for required functions.</li>

  </ul></li>

</ul>

PrintRoutes function included  o unit-test.c – Source file which contains unit test for the routing table update functions that you will implement

<ul>

 <li>c – Source file that has some useful routines for doing the endian conversions o Makefile – Make file that helps in compiling and building the <em>unit-test </em>and <em>router</em> binaries</li>

</ul>

<ul>

 <li><strong>Files you should implement:</strong>

  <ul>

   <li>c – Implement the functions to manipulate the routing table as defined in router.h</li>

   <li>c – Implement the path vector variant of the distance vector protocol using function in routingtable.c</li>

  </ul></li>

</ul>




A complete example is also provided to help you debug your code. Your implementation <strong>must be</strong> <strong>compatible </strong>with our binaries, and with the header files that we have provided. <strong>You are not allowed to</strong> <strong>make any changes to the header files</strong>. The header files will guide your work, by providing more detailed information regarding useful functions and packet formats to be used.




<strong>Important Design Rules</strong>:

<ul>

 <li><strong>In </strong><strong>c you must implement </strong>useful routines that <strong>manipulate the routing table </strong>of the router. The prototypes of these functions are defined in the file <strong>router.h </strong>that we are providing. <strong>NOTE: </strong>We won’t accept your code if everything is implemented in router.c; use both routingtable.c and router.c. The functions that you implement in routingtable.c are stand-alone routines that can be tested for basic correctness using the unit-test code that we have provided. This means you have to build and run <strong>unit-test </strong>to test your logic to manipulate the routing table. The details of the functions are given below: o <strong>InitRoutingTbl() </strong><strong>: </strong>This function initializes the routing table with the neighbor information received from the <em>Network Emulator</em>. This routing table initialization should include a self-route (route to the same router) in the routing table.

  <ul>

   <li><strong>UpdateRoutes(): </strong>This function is invoked on receiving a route update message from a neighbor. For routes that were previousl y unknown, it installs the new routes into the routing table. For known routes, it finds the shortest path and updates the routing table if necessar y. It also implements the forced update and path vector rules. It returns ‘1’ if the routing table changes during the process and ‘0’ otherwise. o <strong>ConvertTabletoPkt(): </strong>This function is invoked on timer expiry to advertise the routes to neighbors. It fills the routing table information into a struct pkt_RT_UPDATE, which is passed as an input argument.</li>

   <li><strong>PrintRoutes(): </strong>This function is provided to you and should be invoked whenever the routing table changes. It prints the current routing table information to a log file that is passed as an input argument. The format of the log message to be printed is explained in detail in the section “Format of Router Log File”. o <strong>UninstallRoutesOnNbrDeath(): </strong>This function is invoked on detecting an</li>

  </ul></li>

</ul>

inactive link to a neighbor (dead nbr). It changes all routes that use the detected dead neighbor in their path vector and changes their cost to INFINITY.




<ul>

 <li>Please note that you must define the following two global variables in <strong>c.</strong></li>

</ul>

The variables must be used only in  <strong>routingtable.c.</strong> o <strong>struct route_entry routingTable[MAX_ROUTERS]</strong><strong>: </strong>This is the routing

table that contains all the route entries. It is initialized by InitRoutingTbl() and updated by UpdateRoutes().

<ul>

 <li><strong>int NumRoutes : </strong>This variable tells the number of routes in the routing table. It is initialized by InitRoutingTbl() and updated by UpdateRoutes().</li>

</ul>




Please <strong>read through the comments in </strong><strong>router.h carefully</strong>, for further details on the functions and variables.




<ul>

 <li><strong>In </strong><strong>c you must use threads</strong> to monitor the UDP file descriptor and to implement all time based functionality. The expected functionality of each threads is described below in the “Thread Implementation” section. Additionally, you <strong>must not use select()</strong> <strong>or fork() </strong>in your code. If the select() OR fork() function is found anywhere within your code, <strong>you will not receive ANY credit</strong>.</li>

</ul>




<strong>Your code must run on Linux</strong>. As was the case in the first project, all code will be graded on ECN servers. Thus, it is expected that you will use these servers to build and test your code prior to submitting it.







<strong>Thread Implementation</strong>:

<ul>

 <li>This project requires the use of threads. It is expected that your program takes the proper steps to initialize by sending the INIT_REQUEST packet and receiving and processing the</li>

</ul>

INIT_RESPONSE packet. From here, it is required that you instantiate two threads. A UDP file descriptor polling thread and a timer thread.




<ul>

 <li>In your UDP file descriptor polling thread, you are to wait to receive an RT_UPDATE packet. Upon receiving such a packet, <strong>UpdateRoutes</strong> should be called to modif y the routing table according to the path vector protocol. When receiving routes, it is important to make note of the time of the last received update packet from a node to check for dead neighbors. Similarly, it is important to track the time of the last routing table update to detect convergence.</li>

</ul>







<ul>

 <li>In your timer thread, it is expected that you constantly monitor all time-based constraints and implement ‘timers’ to control the frequency and convergence of events. You must ensure that once the UPDATE_INTERVAL timer expires, first <strong>ConvertTabletoPkt </strong>is called, followed b y an RT_UPDATE packet being sent to all neighbors. Additionally, you must monitor the activity of your neighbors, and ensure that if you do not hear from some neighbor for</li>

</ul>

FAILURE_DETECTION seconds or more, you mark the neighbor dead. This should be accomplished b y calling the <strong>UninstallRouteOnNbrDeath </strong>function. Lastl y, you need to check for the convergence of your routing table, b y checking that no changes have been made for CONVERGE_TIMEOUT seconds, and appropriately marking the log file as such.




<ul>

 <li>Note that data will need to be shared between threads. Thus, it will be necessary to make use of</li>

</ul>

mutex’s to protect data from being overwritten.







<strong>Configuration File Format:</strong>

The Network Emulator reads the topology information from a configuration file. The configuration file has the following format.




&lt;Number of Routers&gt;

&lt;Router1&gt; &lt;Router2&gt;  &lt;LinkCost&gt;        ## Information about Link1

………                                                      ## Information about other links, 1 line per link

For example, suppose we have the following topology

The configuration file would look like this:

<table width="392">

 <tbody>

  <tr>

   <td width="14"> 4</td>

   <td width="22">  </td>

   <td width="356"> # number of routers</td>

  </tr>

  <tr>

   <td width="14">0</td>

   <td width="22"> 1</td>

   <td width="356"> 4</td>

  </tr>

  <tr>

   <td width="14">0</td>

   <td width="22"> 2</td>

   <td width="356"> 50</td>

  </tr>

  <tr>

   <td width="14">1</td>

   <td width="22"> 2</td>

   <td width="356"> 1</td>

  </tr>

  <tr>

   <td width="14"> 2</td>

   <td width="22"> 3</td>

   <td width="356"> 6</td>

  </tr>

 </tbody>

</table>




<strong>Deliverables</strong><strong>:</strong>

To successfully complete the project, you must implement and turn in only routingtable.c and router.c files. Your implementation must be compatible with our files. In addition, you need to adhere to the format of the configuration file described below, and have your routers produce logs in the manner we describe.

<strong>Format of Router Log File:</strong>

Each router with id &lt;id&gt; <strong>must </strong>produce a log file <strong>router&lt;id&gt;.log</strong>. The routing table of the router must be printed to the log file <strong>whenever the table changes </strong>(e.g., when a new route is added to the table, or the cost, next hop, or path to an existing router changes). The log file contains the history of routing table changes for a given router. The routing table logged has to be in the format described below. If a routing table has not changed for <strong>CONVERGE_TIMEOUT </strong>seconds, append “x:Converged” to the end of the routing table, where x is the number of seconds (rounded down to the nearest integer), that has elapsed since the router receives an INIT_RESPONSE from the Network Emulator.




<strong><em>TIP: </em></strong><em>Use </em><em>fflush to ensure routing changes are immediately written to the log so even if a router is killed with Ctrl-C, the routing table changes are still logged</em>.

<strong><em>Note</em></strong><em>: Print your routing table only whenever there is a change. Otherwise we will deduct points if you keep printing it continuously every second or so. </em>




The format of the log is as follows and is implemented in PrintRotues for your conviencnce alread y. It is highly recommended that you do NOT alter this function :

Routing Table:

&lt;&lt;SRC&gt; -&gt; &lt;DST&gt;&gt;, Path:&lt;SRC&gt; -&gt; &lt;NEXT HOP&gt; -&gt; … -&gt; &lt;DST&gt;, Cost: &lt;COST&gt;

…  # 1 line for each destination it knows

<em>Note: if there is an entry with INFINITY cost in the routing table, print out this entry with (i) the cost being the value of INFINITY as defined in <strong>ne.h</strong>; (ii) the path vector printed is not important for this situation and does not need to be changed. </em>




<strong>Examples:</strong> router 0 would have the following routing table at bootstrap

Routing Table:

&lt;R0 -&gt; R0&gt;, Path: R0, Cost: 0

&lt;R0 -&gt; R1&gt;, Path: R0 -&gt; R1, Cost: 4 &lt;R0 -&gt; R2&gt;, Path: R0 -&gt; R2, Cost: 50




The converged table would look something like:




Routing Table:

&lt;R0 -&gt; R0&gt;, Path: R0, Cost: 0

&lt;R0 -&gt; R1&gt;, Path R0 -&gt; R1, Cost: 4

&lt;R0 -&gt; R2&gt;, Path: R0 -&gt; R1 -&gt; R2, Cost: 5   &lt;R0 -&gt; R3&gt;, Path: R0 -&gt; R1 -&gt; R3, Cost: 11

Make sure the logs written to the file don ’t have unnecessary print statements. Please use the <strong>DEBUG </strong>flag that is provided to you in the <em>Makefile, </em>to add an y extra debug statements in your code. By default this flag is turned off, so that only the required logs are printed.




A set of log files for this 4-router topology is included in the example folder of the gz file for clarit y.







<strong>How to run the router and ne:</strong>

<ul>

 <li>router &lt;router id&gt; &lt;ne hostname&gt; &lt;ne UDP port&gt; &lt;router UDP port&gt;</li>

</ul>




<ul>

 <li>ne &lt;ne UDP Port&gt; &lt;ConfigFile&gt;</li>

</ul>




<strong>Example of execution of 4 routers and network emulator on same host: </strong>

<strong> </strong>

<table width="671">

 <tbody>

  <tr>

   <td width="292">Network Emulator</td>

   <td width="378">Routers</td>

  </tr>

  <tr>

   <td width="292"><u>./ne &lt;2000+classID&gt; 4_routers.conf</u></td>

   <td width="378"><u>./router 0 localhost &lt;2000+classID&gt; &lt;3000+classID&gt;</u></td>

  </tr>

  <tr>

   <td width="292"> </td>

   <td width="378">./router 1 localhost &lt;2000+classID&gt; &lt;4000+classID&gt;</td>

  </tr>

  <tr>

   <td width="292"> </td>

   <td width="378"><u>./router 2 localhost &lt;2000+classID&gt; &lt;5000+classID&gt;</u></td>

  </tr>

  <tr>

   <td width="292"> </td>

   <td width="378">./router 3 localhost &lt;2000+classID&gt; &lt;6000+classID&gt;</td>

  </tr>

 </tbody>

</table>







<strong>Unit Testing:</strong>

We have provided code that contains unit tests for the functions that you will implement in routingtable.c.

On running the unit-test code, you will get messages such as:

<em>Test Case 1: PASS Initialize routing table</em>

<em>Test Case 1: FAILED to initialize routing table</em>

The first message indicates that the test case passed and the second one indicates that it failed. You will get similar messages for each of the test cases. The string after the <em>PASS/FAILED </em>(e.g. <em>Initialize routing table</em>) will tell you what functionality is being tested.  The command-line argument to run the unit test is

unit-test







<strong>System Integration Testing</strong><strong>:</strong>

In  addition  to  testing  your  full  implementation  with  our  example,  you  should  test  your  work  as rigorousl y as possible. To help, we suggest a few possible test scenarios

Small topology, no failures

Large topology, no failures

Failure of node: kill a router and see how routing tables change to reflect the new setting.

Restart of node after failure: other routers update their routing tables to reflect possibl y improved paths




Node failure is achieved by issuing an explicit kill command to the router process or Ctrl -C.




<strong> </strong>

<strong>Makefile Hints</strong><strong>:</strong>

<ul>

 <li>To compile and build the router binar y, run <strong>make clean; make router</strong></li>

</ul>

To compile and build the unit-test binar y, run

<strong>make clean; make unit-test</strong><strong> </strong>

<strong>Advice/Milestones</strong><strong>:</strong>

<ul>

 <li><strong>This project requires dedication. Please work regularly right from the first day on this project and you will find it easy to do.</strong></li>

 <li>Do your work in a modular fashion. For example, check that your router can send and receive INIT_REQUEST/INIT_RESPONSE packets correctl y, before implementing other functionalit y.</li>

 <li>Run <strong><em>unit-test </em></strong>and make sure your route update routines work fine before implementing the complete router framework.</li>

 <li>Here are some milestones for you for each week:</li>

</ul>

<table width="647">

 <tbody>

  <tr>

   <td width="168"><strong>Week 1 </strong></td>

   <td width="480">–  complete functions in routingtable.c–  contact the Network Emulator–  initialize routing table</td>

  </tr>

  <tr>

   <td width="168"><strong>Week 2 </strong></td>

   <td width="480">–  pass unit tests for routingtable.c functions–  implement threading–  verify convergence for basic routing cases</td>

  </tr>

  <tr>

   <td width="168"><strong>Week 3 </strong></td>

   <td width="480">–  handle router failure scenarios–  handle router restart scenarios</td>

  </tr>

  <tr>

   <td width="168"><strong>Week 4 </strong></td>

   <td width="480">– complete rigorous testing using various test cases</td>

  </tr>

 </tbody>

</table>




<ul>

 <li>Note that the above weekly milestones are just suggestions to help you get in pace with the project,</li>

</ul>

but we expect you to do beyond what is specified as a week’s target.







<strong>Checkpoint Submission:</strong>




For checkpoint we minimally require that you complete implementing the following o Contact <em>Network Emulator </em>and initialize routing table with neighbor information o Routing table update functionality in routingtable.c, which passes unit tests.    Submit the following files with the above implementations.

o routing.c o routingtable.c

<strong>Please note that points for the checkpoint submission will be included in the final grade of your project 2.</strong>




<strong>This is just a minimal requirement and we expect that by checkpoint every group</strong> <strong>would have done beyond what is specified, to be on track to complete the project. </strong>

<strong>       </strong>

<strong> </strong>

<strong> </strong>

<strong>Deliverables:</strong>

For each part, please follow the following rules to submit your work:




For part 1 (checkpoint):

<ul>

 <li>Zip the files “routingtable.c” and “router.c” using the following command:</li>

</ul>

<strong>“zip lab2checkpoint.&lt;login1&gt;.&lt;login2&gt; routingtable.c router.c” </strong>

<ul>

 <li>Upload on blackboard in the folder “Project Submission” under “Lab 2 Part 1 (Checkpoint)”</li>

</ul>







For part 2 (final):

<ul>

 <li>Zip the files “routingtable.c” and “router.c” using the following command:</li>

</ul>

<strong>“zip lab2final.&lt;login1&gt;.&lt;login2&gt; routingtable.c router.c” </strong>

<ul>

 <li>Upload on blackboard in the folder “Project Submission” under “Lab 2 Part 2 (Final)”</li>

</ul>