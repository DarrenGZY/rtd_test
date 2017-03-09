# Networking 
> This topic is FAQ to networking API, please see [[Activation File|ActivationFile]] for real guide of remote communications. 

In NPL, you do not need to create any sockets or writing message loops for communication with remote computers. 
Instead, all communications are managed by NPL very efficiently (see [[NPLArchitecture]] for details). Users only need one [single function](ActivationFile) to establish connection, authenticate and communicate with remote computers. 

## Initialize Networking Interface
By default, NPL is started without listening on any port, unless you called some library, such as the [[WebServer]]. 
To enable networking in NPL, one needs to call
```lua
-- a server that listen on 8080 for all IP addresses
NPL.StartNetServer("0.0.0.0", 8080);
```
The client also needs to call `NPL.StartNetServer`. 
```lua
-- a client that does not listen on any port. Just start the network interface. 
NPL.StartNetServer("127.0.0.1", "0");
```
Please see [[Activation File|ActivationFile]] for details. 

### Server Parameters
The following is from [[WebServer]]'s config file. You can have a general picture of what is configurable in NPL. 
```xml
    <!--HTTP server related-->
    <table name='NPLRuntime'>
      <!--whether to use compression for incoming connections. This must be true in order for CompressionLevel and CompressionThreshold to take effect--> 
      <bool name='compress_incoming'>true</bool>
      <!---1, 0-9: Set the zlib compression level to use in case compresssion is enabled. 
      Compression level is an integer in the range of -1 to 9. 
		  Lower compression levels result in faster execution, but less compression. Higher levels result in greater compression, 
		  but slower execution. The zlib constant -1, provides a good compromise between compression and speed and is equivalent to level 6.--> 
      <number name='CompressionLevel'>-1</number>
      <!--when the NPL message size is bigger than this number of bytes, we will use m_nCompressionLevel for compression. 
		  For message smaller than the threshold, we will not compress even m_nCompressionLevel is not 0.--> 
      <number name='CompressionThreshold'>204800</number>
      <!--if plain text http content is requested, we will compress it with gzip when its size is over this number of bytes.-->
      <number name='HTTPCompressionThreshold'>12000</number>
      <!--the default npl queue size for each npl thread. defaults to 500. may set to something like 5000 for busy servers-->
      <number name='npl_queue_size'>20000</number>
      <!--whether socket's SO_KEEPALIVE is enabled.--> 
      <bool name='TCPKeepAlive'>true</bool>
      <!--enable application level keep alive. we will use a global idle timer to detect if a connection has been inactive for IdleTimeoutPeriod-->
      <bool name='KeepAlive'>false</bool>
      <!--Enable idle timeout. This is the application level timeout setting.--> 
      <bool name='IdleTimeout'>false</bool>
      <!--how many milliseconds of inactivity to assume this connection should be timed out. if 0 it is never timed out.-->
      <number name='IdleTimeoutPeriod'>1200000</number>
      <!--queue size of pending socket acceptor-->
      <number name='MaxPendingConnections'>1000</number>
    </table>
``` 
Please note, the `IdleTimeout` should be set for both client and server, because timeout may occur on either side, see below for best timeout practice. `TCPKeepAlive` is only for server side. 

To programmatically set parameters, see following example:
```lua
	local att = NPL.GetAttributeObject();
	att:SetField("TCPKeepAlive", true);
	att:SetField("KeepAlive", false);
	att:SetField("IdleTimeout", false);
	att:SetField("IdleTimeoutPeriod", 1200000);
	NPL.SetUseCompression(true, true);
	att:SetField("CompressionLevel", -1);
	att:SetField("CompressionThreshold", 1024*16);
	-- npl message queue size is set to really large
	__rts__:SetMsgQueueSize(5000);
```
### TCP Keep Alive
If you want a persistent already-authenticated connection even there are no messages sent for duration longer than 20 seconds, you probably want to generate some kind of regular ping messages. Because otherwise, either the server or the client may consider the TCP connection is dead, and you will lose your authenticated session on the server side. 

Although NPL allows you to configure server side application level ping messages globally with `KeepAlive` attribute, it is not recommended to enable it on the server or set its value to a very large value (such as several minutes).

The recommended way is that the client should initiate the ping messages to all remote processes on regular intervals.

For a robust client application, the client also needs to handle connection lost and recover or re-authenticate at application level. 