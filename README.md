java c
CS 168 Fall 2024

Project 2:Distance-Vector Routing
In this project,you will write a distance-vector routing protocol.We have provided simulator code that you can use to visualize your protocol in action.
Lectures needed for this project:Lecture 6(Routing States),Lecture 7 (Distance-Vector Protocols).You can work on this project alone,or with one partner.You can keep the same partner from Project 1,or choose a different partner.Reminder:  Our course policies page has a collaboration policy that  youmust follow.
Setup
Python Installation
This project has been tested to work on Python 3.8.(It probably works on newer versions of Python as well,but use at your own risk.)
If you run python3 --version or python --version in your terminal and you see output of the form. python 3.8.*, you're all set.
Starter Code
Download a copy of the starter code here.We recommend making frequent backups of your code.Some parts of the project will require you to modify code that you wrote earlier,and you might want to revert your  changes to start over from an earlier save point.
You should only edit dv router.py.There are comments clearly indicating the places where you should fill in code.
Guidelines:
·Don't  modify  any other files. ·Don't add any new files.
·Don't  add any  imports.
·Don't edit any code outside the sections indicated by the comments.
·Don't add any hard-coded global variables.
·Adding helper methods is fine (and encouraged in the last part).
·In general,if we haven't told you to use something,you probably don't need it.Our goal is not to trick you!
In   your   terminal,use    cd   to   navigate    to   the cs168-fa24-proj2-routing/simulator directory.All     of
the Python commands should be run from this directory.
New Spec
Note:This spec has been rewritten for Fall 2024 to be clearer and provide a few extra hints.It may contain bugs.
If you would like to work off the previous version of the spec,you can find ithere.
Project Overview
This project is split into 3 sections,with a total of 10 stages across all sections.
Each stage has its own unit tests,provided to you locally.Your grade will be determined only by these unit tests (no hidden tests).
To run a unit test,run this command,replacing 5 with the number of the stage you want to test:
python3  dv_unit_tests.py  5
Warning:Stage 10 is a good deal longer than the other stages,so plan accordingly
Warm-Up Example:Hub
Some assumptions before we get started:
·In this project,your code will find routes between hosts.You don't need to find routes to other routers.
·In this project.our forwarding tables will map destinations to numbered physical ports,instead of next-hops.For example,a row of the table might say:“Packets   destined for h1 should be sent out of port 1.”To get you started,we've provided an implementation of a hub in examples/hub.py.The hub  is a network device that takes any incoming packet,and forwards that packet out of all its ports (except the port the packet came from).
Let's run the simulator on a linear topology with three hosts:
python3 simulator.py --start --default-switch-type=examples.hub topos.linear--n=3
You can now access the visualizer at http://127.0.0.1:4444 in your browser.You should see the hosts and routers displayed against a purple background.
Now,let's make host h1 send a ping packet to host h3.You can either type in the Python terminal:
h1.ping(h3)
Or,you can send the ping from the visualizer by:(1)clicking h1 and pressing A,(2) clicking h3 and pressing B,and (3)pressing p to send the ping from A to B.
You should see the"ping"packet sent from  h1 to  h3.Then,you should see the  reply
“pong”packet sent from h3 to h1.You should also see both packets delivered to h2,even though h2 isn't the recipient.This behavior. is expected,because the hub floods packets   everywhere.
You can also observe what's going on by reading the log messages printed to the Python terminal (you can ignore the TTL field,you won't touch it in this project).
WARNING:user:h2:NOT            FOR            ME:h3            ttl:17>s1,s2,h2
DEBUG:user:h3:rx:h3                        ttl:16>s1,s2,s3,h3
WARNING:user:h2:NOT           FOR           ME:h3           ttl:16>>s3,s2,h2
DEBUG:user:h1:rx:h3                  ttl:16>>s3,s2,s1,h1
Recall from class that flooding is problematic when the network has loops.Let's see this in action by launching the simulator with the topos.candy topology,which  has  a  loop:
python3 simulator.py --start --default-switch-type=examples.hub topos.candy
Now,send a ping from host h1a to host h2b.You should be seeing a lot more log
messages in the terminal,and the visualizer should be showing routers forwarding
superfluous packets for quite a while.Oops!Your job in this project will be to implement a more capable distance-vector router.
You don't need to modify or submit anything for this section.
Code Overview
You will be implementing the DVRouter class in dv router.py,which represents a single router.
Some useful helper classes are implemented in dv.py (you can read this file,but don't modify it).We've also described them below.
Table
The instance variable self.table is aTable object representing the forwarding table inside the router.
The Table object is a dictionary,where the keys are destinations,and values are TableEntry objects.
Each TableEntry object    contains:
·dst:The destination  for  this  route.
·  port :The port that packets to the destination should be sent out of.
·  latency:The latency("cost")of the route from this router to the destination. ·expire_time:The timestamp (in seconds)at which this route expires.Use
api.current time() to get the current time,and add seconds to indicate a time in the future.
TableEntry   objects    are   immutable,so    if    you   want    to   update    an    entry,you   should    create    a
new TableEntry with      updated      attributes.
Example usage:
t    =Table()t[h1]=TableEntry(dst=h1,port=p1,latency=10,expire_time=api.current_time()+20) t[h2]=TableEntry(dst=h2,port=p2,latency=20,expire_time=api.current_time()+20) for host,entry in t.items(): #<--This is how you iterate through a dict.
print( "Route  to  {}has  latency  {}". format(host,entry.latency)) This corresponds to a table that looks like this:
Key
Value(TableEntry)
Destination
Destination
Port
Latency
Expire Time
h1
h1
p1
10
20 seconds later
h2
h2
p2
20
20 seconds later
Ports
The instance variable self.ports is a  Ports object representing the links connected to the router.
The Ports object contains a dictionary,where each key is a link(port),and the corresponding value is the latency (cost)along that link.
To get the latency along the link connected to port p,you can use:
self.ports.get_latency(p)
To iterate through all the ports,you can use:
for  p in   self .ports.get_all_ports():
Flags
The following flags identify what mode the router is in.We'l tell you when to include them in the code.Your code should not be reassigning these flags to True or False.
● self.SPLIT HORIZON
● self.POISON REVERSE
self.POISON ON LINK DOWN
· self.POISON          EXPIRED 
self.SEND     ON     LINK     UP
Stage 1:Installing Static Routes
Foreach host directly connected to your router,you should record a static route to that host.
Your Task
Implement add static route.
Assign an expire time of FOREVER for the new route.
The framework will magically call the code you write here any time a new static route needs to be installed.
Testing and Debugging
To check your work,run the unit tests:
python3  dv_unit_tests.py  1
To debug,you can start up the simulator,and use the terminal to print out the tables inside each router.For example,to print the table inside s1:
print(s1.table)
If you're debugging in the code that you wrote,you can print out your own table like this:
print(self.table)
Check Your Understanding
Now that we've implemented static routes,what do you expect to happen if we try to send a packet along a path?
To find out,start up the simulator and open http://127.0.0.1:4444 in your browser:
python3 simulator.py --start    --default-switch-type=dv_router topos.simple
Try to send a ping from host h1 to host h2,e.g.by typing h1.ping(h2) in the terminal. What happened?Did it align with your expectation?
Stage 2:Forwarding
We've added entries to the routing table,but we need to actually use them to forward packets.
Your Task
Implement handle_data_packet.
This method will be called each time a data packet arrives at your router. Hint:To   send   a   given packet out   of   a   given   port,you   can   use:
self.send(packet,port=port)If no route exists for the packet's destination,you should drop the packet (do nothing).
If the latency along the outgoing link is greater than or equal to INFINITY,you  should  also drop the packet.
Testing  and  Debugging
To check your work,run the unit tests: python3    dv_unit_tests.py    2
Check Your Understanding
Start the simulator from the previous stage again:
python3 simulator.py --start --default-switch-type=dv_router topos.simple
Again,try to send a ping from host h1 to host h2,e.g.by typing h1.ping(h2) in  the terminal.Hopefully the packet arrived this time!
You've finished the first section!Next,let's add more routes to our table.
Stage 3:Sending Advertisements
Check Your Understanding
Given our current implementation,what should happen when pings are sent from one host to the other in these topologies?
Topology 1:  h1  ---s1 ---h2
Topology 2:h1  ---s1 ---s2 ---h3
Do the pings work in both topologies?Only one of them?Neither of them? To find out,start the simulator again to run through these two scenarios:
python3 simulator.py --start --defaul代 写CS 168 Fall 2024Python
代做程序编程语言t-switch-type=dv_router topos.simple
We have a problem in Topology 2.The routers collectively know about both destinations, but each router by itself doesn't know about both destinations.
To fix this,let's have routers exchange advertisements so they can learn about other destinations in the network.
Your Task
Implement send routes.
For now,you can assume force=True and single port=None,and  ignore  those arguments.
The framework will periodically call this function for you,so that your router periodically advertises routes to all of its neighbors.
Hint:To send a message out of a specific port,saying that you can reach a specific dst with a specific latency,you can  use:
self.send_route(port,dst,latency)
Testing  and  Debugging
To check your work,run the unit tests: python3   dv_unit_tests.py   3
Check Your Understanding
What happens now if we try to send a ping in Topology 1 and Topology 2?
To find out,start the simulator again and try it out.You'll notice that routing advertisements (purple dots)are periodically being sent out of each router.
However,the ping in Topology 2 stilldoesn't work.
Stage 4:Handling AdvertisementsYour router now sends advertisements,but it also needs to handle any advertisements it receives!In particular,we'll need to implement Rule 1(Bellman-Ford update)and Rule 2  (Updates From Next-Hop)of distance-vector.
Your Task
Implement  handle route   advertisement.
The framework will call this for you every time your router receives an advertisement.If the advertised route is equally good as the current route,tiebreak by preferring the   current route.In other words,only accept an advertised route if it is strictly better than the current route.
Reminder:If you get an advertisement for a destination that's not in the table,you should always accept it.
Reminder:If you get an advertisement from the current next-hop,you should always accept it.This is Rule 2 (Updates From Next-Hop)of distance-vector.
If you update the table,set the expire time of the new entry to
api.current time()+self.ROUTE TTL.(By default,this is 15 seconds in the future.)
Testing and Debugging
To check your work,run the unit tests: python3    dv_unit_tests.py    4
Check Your Understanding
Why did we choose to tiebreak by preferring the current route?What if we preferred the new route instead?
There's a trade-off between correctness and stability here.
To see what we mean by stability,temporarily change your implementation to tiebreak by preferring the new route (accept advertised route if it's equally good as the current
route).
Then,start the simulator with the square topology:
python3 simulator.py --start --default-switch-type=dv_router topos.square
Send a bunch of pings between the two corners of the square.Notice that packets
between the same two hosts can take different paths.We consider this to be unstable, and suboptimal.
(Preview:Packets taking different paths might end up arriving out-of-order,and this causes TCP,the Layer 4 reliability protocol,to slowdown.)
Now,undo your temporary change,so that you again tiebreak by preferring the current route (only accept advertised route if it's strictly better).
Then,start the simulator again and send a bunch of pings again.Our packets always take the same path now!
So,what's the trade-off?Preferring the current path is more stable.On the other hand,a new route represents state that is more recent and therefore more indicative of the
current state of the network.
Stage 5:Handling Timeouts
Next,let's implement Rule 3 (Expiring)of distance-vector.
Check Your Understanding
What happens if a link goes down?To find out,start the simulator with the candy topology:
python3 simulator.py --start --default-switch-type=dv_router topos.candy
Wait for all routes to propagate.Then,remove the link between routers s4 and s5.You
can either select the two routers and press E in the browser,or you can type s4.unlinkTo(s5)in the terminal.
Now,try sending a ping from h1a to h2b.Even though there's still a path through s3, you'll see the packet get forwarded to s4 and then dropped!
Router s4 is trying to forward to s5,to which it is no longer connected!
Your Task
Implement expire_routes.
The framework will call this function periodically for you.Your job is to go through the forwarding table entries and delete any entries that are expired.
Optional,ungraded:When you delete an expired route,use self.s_log or self.log to  print out a message in the terminal.
Hint:To remove a table entry with key h,you can use: self.table.pop(h)
Testing  and  Debugging
To check your work,run the unit tests: python3    dv_unit_tests.py    5
Check Your Understanding
Start up the simulator with the candy topology again:
python3 simulator.py --start --default-switch-type=dv_router topos.candy
Again,bring  down  the   link,e.g.s4.unlinkTo(s5).Start sending a bunch of pings from h1a     to  h2b.Roughly  15  seconds  after the  link  goes  down,the old  route will  expire,and future pings should be forwarded correctly along the alternate route!
You've finished the second section!Next,let's  make  our  router  more efficient.
Stage 6:Split Horizon
Next,let's  implement  Rule  5A  (Split  Horizon)of  distance-vector.
Check Your Understanding
Start up the simulator with the linear topology with 3 hosts:
python3 simulator.py --start --default-switch-type=dv_router topos.linear--n=3
Wait for all routes to propagate.
Send a ping from h3 to h1.This should work.
Next,bring  down  the  link  between  s1  and  s2,e.g. s1.unlinkTo(s2).
Now,try sending pings from h3 to h1.You should see packets get dropped at s2.
Eventually,the route at s2 will timeout.and s2 will get a new advertisement from s3. Now,when you send a  ping from  h3 to  h1,what  happens,and why?
How  long will this  routing  loop  continue?What  is  the  larger  problem,and  how  can  we tackle this issue?
Your Task
Modify send routes to support split horizon if the self.SPLIT HORIZON flag is True.
Testing and Debugging
To check your work,run the unit tests: python3    dv_unit_tests.py    6
Check Your Understanding
Set self.SPLIT HORIZON to True  in  your code,and  run  the same demo again.The  routing loop  problem  should  be  solved  now!
Stage 7:Poison Reverse
Next,let's implement Rule 5B(Poison Reverse)of distance-vector.
Check Your Understanding
Set self.SPLIT HORIZON to False,and start up the simulator with the double triangle topology:
python3  simulator.py  --start    --default-switch-type=dv_router   topos.double_triangle Disconnect s2 by removing all links that connect it to other routers:s2.unlinkTo(s1) s2.unlinkTo(s3) s2.unlinkTo(s4)
Let's see if split horizon will save us!Turn on split horizon and re-run the demo. Can we do better than this,and if so,by how much?
Your Task
Modify send routes to implement poison reverse advertisements if the self.POISON REVERSE flag is True.
Note:  self.POISON  REVERSE and self.SPLIT            HORIZON will       never        be       True        at       the       same        time.They
could both be False,or one of them could be True.
Testing and Debugging
To check your work,run the unit tests: python3  dv_unit_tests.py  7
Check Your Understanding
Set self.POISON REVERSE to True,and re-run the double triangle demo from earlier.How long will it take to reach the correct state now?
Stage 8:Counting to Infinity
Next,let's implement Rule 6(Count to Infinity)of distance-vector.
Check Your Understanding
Recall that split horizon and poison reverse can prevent length-2 routing loops,but not longer ones.
To see why,start the simulator with the loopy topology:
python3 simulator.py --start  --default-switch-type=dv_router topos.loopy
Wait for all routes to propagate.
Disconnect s1 from s2,e.g.s2.unlinkTo(s1).
You can print the forwarding tables in the terminal,e.g.print(s1).How  long will the routers count?It seems to be forever...despite all of our hard work!
Will we ever stabilize?Can split horizon or poison reverse save the day?
Your Task
Modify  send  routes so  that  any   latency  greater  than INFINITY is  rounded  down  to INFINITY when advertised.
Testing and Debugging
To check your work,run the unit tests: python3    dv_unit_tests.py    8
Check Your Understanding
Re-run the loopy demo from earlier.How long will the routers count now?
Stage 9:Poisoning Expired Routes
Next,let's implement Rule 4(Poisoning Expired Routes)of distance-vector.
Check Your Understanding
Start up the simulator with the linear topology with 7 hosts:
python3 simulator.py --start --default-switch-type=dv_router topos.linear --n =7
Wait for all routes to converge (e.g.s7's table should have a route to h1).This can take a while!
Then,disconnect s1 from s2,e.g.s1.unlinkTo(s2).
How long will it take for the route to s1 to be updated in the other routers?
Your Task
Modify expire routes.
Any expired routes should get replaced with poison if self.POISON EXPIRED is True.
Set the poisoned entry to have an expire time of api.current time()+self.ROUTE TTL.This ensures the poisoned route gets advertised periodically.
Optional,ungraded:Ideally,the periodic advertisements should halt after awhile (i.e. poisoned routes should be removed eventually),because it's not useful to keep
advertising the nonexistence of a route.We won't test you on this though.
Testing and Debugging
To check your work,run the unit tests: python3   dv_unit_tests.py   9
Check Your Understanding
What do route removals now propagate relative to?You're almost done!Now might be a goodtime to back up your work.The last stage  requires changing your code so far,and you might want to revert to your code before starting Stage 10.



         
加QQ：99515681  WX：codinghelp
