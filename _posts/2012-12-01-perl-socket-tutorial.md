---
layout: post
title: "Perl Socket Tutorial"
---

## Perl socket programming tutorial
### Socket programming in perl can be done using the low level socket functions or the IO::Socket module.The IO::Socket module provides an object-oriented interface to the socket functions.
In this tutorial we are going to use the low level socket functions.

### Example 1：Connecting the baidu server with port 80

### Perl code

{% highlight perl %}
#!/usr/bin/perl -w

use strict;
# The socket function can be used to create a socket.In the following code example we shall create a tcp socket.
use Socket;

my $proto = getprotobyname('tcp'); # get the tcp protocol
# Create a socket handle(descriptor)
socket(SOCKET,AF_INET,SOCK_STREAM,$proto) or die "cant create socket:$!\n";
# Connecting to a remote server
my $remote = 'www.baidu.com';
my $port = 80;
my $iaddr= inet_aton($remote) or die "unable to resolve hostname: $remote\n";
my $paddr = sockaddr_in($port,$iaddr); # socket address structure

connect(SOCKET,$paddr) or die "connect failed : $!\n";
print "Connected to $remote on port $port\n";

# Close a socket
close SOCKET,
exit(0);

{% endhighlight %}

### explain:

In the above example we connected to baidu.com on port 80. 
Before the connect function is called the sockaddr_in structure has to be setup. 
The sockaddr_in structure stores the information about the remote address, 
address type and port number. The inet_aton function converts a hostname/ip to 
ip address in long number format.

### Example 2

### Perl code

{% highlight perl %}
#!/usr/bin/perl -w

use strict;
# The socket function can be used to create a socket.In the following code example we shall create a tcp socket.
use Socket;
use Encode;

my $proto = getprotobyname('tcp'); # get the tcp protocol
# Create a socket handle(descriptor)
socket(SOCKET,AF_INET,SOCK_STREAM,$proto) or die "cant create socket:$!\n";
# Connecting to a remote server
my $remote = 'www.baidu.com';
my $port = 80;
my $iaddr= inet_aton($remote) or die "unable to resolve hostname: $remote\n";
my $paddr = sockaddr_in($port,$iaddr); # socket address structure

connect(SOCKET,$paddr) or die "connect failed : $!\n";
print "Connected to $remote on port $port\n";

# Send data
# Once connected it's time to start communication by sending some data.The send function can be used for this.
# Send some data to remote server - the HTTP GET command
send(SOCKET,"GET / HTTP/1.0\n\n",0) or die "send failed:$!\n";

# Receive reply from server - perl way of reading from stream.
# Can also do recv(SOCKET,$msg,2000,0).
while(my $line = <SOCKET>) {
#       $line = encode_utf8($line);
        $line = encode("utf-8",decode("gbk",$line));#（translate gbk code to utf-8 code）
        print "$line\n";
}
# Close a socket
close SOCKET,
exit(0);

{% endhighlight %}

### explain:

Revise

In the above examples we learned the following socket operations.

1. Create a socket
2. Connect to remote server/system.
3. Send data
4. Receive a reply

The socket program we wrote above is called a client. 
A client connects to a remote server for exchange of information/data. 
Apart from the client the other type of socket program we are going to write is called a server. 
The server serves data to clients connecting to it. 
So lets move on to the next section to do some server programming in perl.

### Server

The basic steps for making a server are :

1. Create a socket
2. Bind to a local address and local port
3. Listen for incoming connections
4. Accept incoming connections
5. Communicate the with the newly connected client - send and receive data.

So a server does not connect out to a system, instead it waits for incoming connections. 
We have already seen how to create a socket. 
So the next task to make a server would be to bind the socket. 
The bind function can be used for this task.

Lets take a quick example

### Example 3

### Perl code

{% highlight perl %}
#!/usr/bin/perl -w
use strict;
use Socket;
$| = 1;

#create a socket handle(descriptor)
socket(SERVER,AF_INET,SOCK_STREAM,(getprotobyname('tcp'))[2]) or die "cant create socket :$!\n";
#bind to local port 10000
my $port = 10000;
#the socket is bound to port 10000 and address "any".After bind the socket has to be put in listen mode.
bind(SERVER,sockaddr_in($port,INADDR_ANY)) or die "bind failed:$!\n";
# listening the incoming connections
listen(SERVER,10);
print "Server is now listening...\n";
# After calling listen,the socket is ready to accept incoming connections using the accept function.
This shall be done in a loop so that the program can accept connections again and again.

#accept incoming connections and talt to clients
while(1) {
        my $addrinfo = accept(TALK,SERVER);
        my($port,$iaddr) = sockaddr_in($addrinfo);
        my $clienthost = gethostbyaddr($iaddr,AF_INET);
        print "Connection accepted from $clienthost:$port\n";
        # send some message to the client
        print TALK "hello client how are you\n";
        close TALK;
}

#close the socket
close SERVER;
exit(0);

{% endhighlight %}

### excute the code

{% highlight bash %}
[root@dou socket]# telnet localhost 10000
Trying 127.0.0.1...
Connected to localhost.localdomain (127.0.0.1).
Escape character is '^]'.
hello client how are you
Connection closed by foreign host.

{% endhighlight %}

### Another example about Server(set the ip addr of the server)

### Perl code

{% highlight perl %}
#!/usr/bin/perl -w
use strict;
use Socket;
$| = 1;

#create a socket handle(descriptor)
socket(SERVER,AF_INET,SOCK_STREAM,(getprotobyname('tcp'))[2]) or die "cant create socket :$!\n";
#bind to local port 10000
my $port = 10000;
my $server = "192.168.0.6";
my $iaddr = inet_aton($server);
my $paddr = sockaddr_in($port,$iaddr);
#the socket is bound to port 10000 and address "any".After bind the socket has to be put in listen mode.
bind(SERVER,$paddr) or die "bind failed:$!\n";
# listening the incoming connections
listen(SERVER,10);
print "Server is now listening...\n";
# After calling listen,the socket is ready to accept incoming connections using the accept function. 
This shall be done in a loop so that the program can accept connections again and again.

#accept incoming connections and talt to clients
while(1) {
        my $addrinfo = accept(TALK,SERVER);
        my($port,$iaddr) = sockaddr_in($addrinfo);
        my $clienthost = gethostbyaddr($iaddr,AF_INET);
        print "Connection accepted from $clienthost:$port\n";
        # send some message to the client
        print TALK "hello client how are you\n";
        close TALK;
}

#close the socket
close SERVER;
exit(0);
{% endhighlight %}


### Advanced Skills:

Asynchronouse I/O
Multiple clients can be handled by asynchronous event driven I/O. IO::Select can be used for this.

### Example 1:

### Perl code

{% highlight perl %}
#!/usr/bin/perl -w
use strict;
use Socket;
use IO::Select;

$| = 1;
my $socket;
my $servtoclient;
#create a socket handle(descriptor)
my $proto = getprotobyname('tcp');
socket($socket,PF_INET,SOCK_STREAM,$proto) or die "create socket failed:$!\n";

#bind to local port
my $port = 10001;
my $server = "192.168.0.6";
my $iaddr = inet_aton($server);
my $paddr = sockaddr_in($port,$iaddr);
bind($socket,$paddr) or die "bind failed:$!\n";

#listen incoming connections
listen($socket,10);
print "Server $server on port $port is now listening ...\n";

#accept incoming connections and talk to clients
my $select = IO::Select->new();
$select->add($socket);
while(1) {
        my @ready = $select->can_read(0);
        foreach my $select_io(@ready) {
                #new connection read
                if($select_io == $socket) {
                        my $clientinfo = accept($servtoclient,$socket);
                        my($clientport,$clientiaddr) = sockaddr_in($clientinfo);
                        my $clienthost = gethostbyaddr($clientiaddr,AF_INET);
                        print "Connection accepted from $clienthost : $clientport\n";

                        # send some message to the client
                        send($servtoclient,"Hello client[$clienthost]! Nice to meet you !\n",0);
                        $select->add($servtoclient);
                }
                # existing client read
                else {
                        chop(my $clientinput = <$select_io>);
                        chop($clientinput);
                        if ($clientinput =~ /quit/i) {
                                $select->remove($select_io);
                                $select_io->close;
                        }else {
                                print "Received -- $clientinput\n";

                                # send reply back to client
                                send($select_io,"OK : $clientinput\n",0);
                        }
                }
        }
}

#close the socket
close $socket;
exit(0);
{% endhighlight %}


### Execuete the command on the server side

{% highlight bash %}
[root@dou Socket]# perl iosocket.pl &
[1] 3580
[root@dou Socket]# Server 192.168.0.6 on port 10001 is now listening ...

[root@dou Socket]#

{% endhighlight %}

### Access the server from the client:

{% highlight bash %}
[root@dou ~]# telnet 192.168.0.6 10001
Trying 192.168.0.6...
Connected to dou.perl.gov (192.168.0.6).
Escape character is '^]'.
Hello client[dou.perl.gov]! Nice to meet you !
{% endhighlight %}


### When the client access the server,the following will be shown.

{% highlight bash %}
[root@dou Socket]# Connection accepted from dou.perl.gov : 42617
{% endhighlight %}

### Some other messages from the test.

{% highlight bash %}
客户端开始发送信息：
ls---->send
OK : ls ---->receive
hello---->send
OK : hello ---->receive
hello world---->send
OK : hello world ---->receive
同时server端也返回信息：
Received -- ls
Received -- hello
Received -- hello world
{% endhighlight %}

### The final example:

### Perl code

{% highlight perl %}
#!/usr/bin/perl -w
use strict;
use Socket;
use IO::Select;

$| = 1;
my $path = "/usr/kerberos/sbin:/usr/kerberos/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin";
my @paths = split(/:/,$path);
my $hit = 0;
my $command;
my @lines;
my $line;
my $socket;
my $servtoclient;
my $reply;
#create a socket handle(descriptor)
my $proto = getprotobyname('tcp');
socket($socket,PF_INET,SOCK_STREAM,$proto) or die "create socket failed:$!\n";

#bind to local port
my $port = 10001;
my $server = "192.168.0.6";
my $iaddr = inet_aton($server);
my $paddr = sockaddr_in($port,$iaddr);
bind($socket,$paddr) or die "bind failed:$!\n";

#listen incoming connections
listen($socket,10);
print "Server $server on port $port is now listening ...\n";

#accept incoming connections and talk to clients
my $select = IO::Select->new();
$select->add($socket);
while(1) {
        my @ready = $select->can_read(0);
        foreach my $select_io(@ready) {
                if($select_io == $socket) {
                        my $clientinfo = accept($servtoclient,$socket);
                        my($clientport,$clientiaddr) = sockaddr_in($clientinfo);
                        my $clienthost = gethostbyaddr($clientiaddr,AF_INET);
                        print "Connection accepted from $clienthost : $clientport\n";
                        send($servtoclient,"Hello client[$clienthost]! Nice to meet you !\n",0);
                        $select->add($servtoclient);}
                else {
                        while(chop(my $clientinput = <$select_io>)) {
                                chop($clientinput);
                                $clientinput =~ s/^[ \t]*//;
                                if ($clientinput =~ /quit/i) {
                                        $select->remove($select_io);
                                        $select_io->close;
                                        last;   # (跳出while循环jump out the loop of while)
                                        #close $socket;
                                        #exit 0;
                                }else {
                                        @lines = split(/[ ]/,$clientinput);
                                        chomp($command = $lines[0]);
                                        foreach my $dir (@paths) {
                                                search($dir);
                                                if ($hit == 1) {
                                                        last; # （跳出foreach循环jump out the loop of foreach）
                                                }
                                        }
                                        if ($hit != 0) {
                                                send($select_io, "Command is [$command]:\n",0);
                                                 $reply = `$clientinput`;
                                                 #send reply back to client
                                                 send($select_io,"$reply",0);

                                        } else {
                                                send($select_io,"you enter string [$clientinput]\n",0);
                                        }
                                }
                        }
                }
        }
}

#close the socket
close $socket;
exit(0);

sub search {
        my $dir = shift;
        opendir(DIR,"$dir") or die "cant open dir:$dir:$!\n";
        my @files = readdir(DIR);
        closedir(DIR);
        foreach my $file (@files) {
                next if ($file =~ m/^\.$/ || $file =~ m/^\.\.$/);
                if ("$file" eq "$command") {
                $hit += 1;
                return(0);
                }  else {
                        $hit = 0;
                }
        }
}
{% endhighlight %}


### Execute the command from the server side:
{% highlight bash %}
[root@dou Socket]# perl 7.pl &
[1] 3641
[root@dou Socket]# Server 192.168.0.6 on port 10001 is now listening ...

[root@dou Socket]# Connection accepted from dou.perl.gov : 33965

{% endhighlight %}
 
### The client-end
{% highlight bash %}
[root@dou ~]# telnet 192.168.0.6 10001
Trying 192.168.0.6...
Connected to dou.perl.gov (192.168.0.6).
Escape character is '^]'.
Hello client[dou.perl.gov]! Nice to meet you !
ls
Command is [ls]:
1.pl
1.py
2.pl
44.pl
4.pl
5.pl
6.pl
7.pl
aaa
client.pl
ht.pl
iosocket.pl
iosocket.pl.orig
open_client.pl
server.pl
tk.pl
     hostname
Command is [hostname]:
dou.perl.gov
                          hostname
Command is [hostname]:
dou.perl.gov
quit
Connection closed by foreign host.
{% endhighlight %}

### Explain:
1) Execute the command(perl 7.pl) on the server-end
2) Using telnet tool to access the server from the client-end
3) Enter some linux commands(eg:ls、hostname、ifconfig eth0...)
4)The program(7.pl) tell what you enter whether is a linux command through searching the system executable path($PATH).
if so,then the variable $hit will be increased one, if not,....


