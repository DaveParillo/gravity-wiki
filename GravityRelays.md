## Using Gravity Relays - Example 13 ##

### The problem ###
Gravity Relays only start to make sense when you take into account the way the Gravity publishers and subscribers connect to each other.  Gravity uses a peer-to-peer architecture, so a connection is made between each publisher and subscriber pair (there is no intermediary acting as a data repository here).  This is a large part of why Gravity is capable of transmitting large volumes of data quickly between nodes.  In some network layouts though, this can have a negative impact on performance.  Consider the case below where NodeA is on Host1, and NodeB and NodeC are on Host2:  

![Without Relay](https://github.com/aphysci/gravity/blob/DocUpdates/test/examples/13-Relay/doc/WithoutRelay.jpg)

The two subscribers each make a connection to the publisher.  When NodeA publishes a message, that data is sent directly to NodeB and NodeC.  As you can see in the image above, this means that the data is sent across the network twice.  If Host1 and Host2 are connected by a slow network link and/or if the volume of data being published by NodeA is high, this can lead to performance issues.

### The solution: Gravity Relay ###

With a Gravity Relay, you can ensure that data is only passed between hosts once.  Below is the scenario above with the addition of a Gravity Relay:

![With Relay](https://github.com/aphysci/gravity/blob/DocUpdates/test/examples/13-Relay/doc/WithRelay.jpg)

NodeB and NodeC subscribe to and receive the data in exactly the same way that they normally would.  They don't need to know or care that the data was relayed through another component.  If they do care though, there is a flag on the GravityDataProduct that indicates that the data was forwarded via a Relay.  More on this, and other details on how to use Relays below.

