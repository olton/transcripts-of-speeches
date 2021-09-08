### Hello everyone!

My name is Serhii Pimenov. I’m a web developer from Kyiv, Ukraine (maybe you know me by the nickname olton). 

Before I start, I want to apologize for my English. Currently, it is not good, but I work on developing it and I hope in the near future my English will be better. 
I will try to speak so that everyone understands me. This is my first video in English, so please don't judge me strictly and I beg your pardon if I say any words incorrectly.

Today I'm going to speak about one of my tools for the Mina blockchain - “Mina Monitor”. It’s the first video in the series about Mina and Mina Tools. 
In this video, I will introduce you to my tool for monitoring the Mina nodes.

### Ok, Let’s start... Why it needs

Making a profit in Mina's blockchain is based on the Proof of Stake mechanism. 
To get Mina tokens you must launch the node with the required size of the stake and generate blocks to the blockchain. 
The very important thing - stability of the node. You must control the node state to win slots and blocks. 
This brings us to the heart of my speech.

So, how can we monitor the state of the node? Firstly, it is the usage of the "mina client status" command. 
Secondly, we can use built-in metrics, Grafana and Prometheus to get graphs for any node states. 
Third, use various system utilities to control resource consumption (such as htop, cat meminfo, and others).

But, all these services and utilities in one way or another limit my ability to control the Mina node.

***Show screenshot with a tmux***

In the beginning, I used a console command “mina client status” to get information about a node state, different system 
utilities to get server resources utilization, and system journals to read different logs. 

The main tool for obtaining information about a node state is a “mina client status” command. 

***show screenshot with a “mina client status” command***

The command “mina client status” is a good one, but it’s very boring and not informative enough for me.

We can see that this command gives a lot of information, but:

- This information is static, we must re-run the command to update it
- This information is not fully qualified to determine the full health of the node at all (for example, we can’t see if a node is hanging or not)
- We can’t see information about server resources utilization: CPU and RAM usage, network traffic.
- Maybe, if you are a validator, you want to see information about delegations to your address, total delegated stake, how much you have a reward in the current epoch, and where your address is in the uptime leaderboard. You can’t see this information by the command “mina client status”.
- I would like to see the updatable price of Mina in different currencies and a balance cost in these currencies.
- All in all, I want the node to reboot with minimal loss in time if something doesn’t work correctly without my intervention. 


Of course, we can execute a lot of steps to get information about the node:

- en the page “http://uptime.minaprotocol.com” to get a position in the leaderboard
- n “htop” to see system resources usage statistics and “cat /proc/meminfo” to get information about used memory
- wnload the ledger and calculate the total stake
- sit Gareth's Mina Explorer and get information about rewards

or take many more steps to get the information we are interested in, and if we have suspicions that something is wrong with the node we run a command to restart a mina service. 

Too many actions need to be performed to get one result.

To be honest, I am a very lazy person, and I was too lazy to do all these actions, and I decided to simplify my life and to create Mina Monitor, that would collect and show me all this information (even on my TV), control node state, while I just sitting on the couch, drinking beer and enjoying my life.

***Show picture with TV***

### We came directly to Mina Monitor

So, Mina Monitor is a client/server application for visually monitoring the Mina node, alerting about node problems, and automatically restarting the node if needed. 

Mina Monitor is written using HTML, CSS, JavaScript, NodeJS and consists of two parts: server-side - this module collects information about node, monitors node health, and restarts node if needed  (must be running on the same server with Mina) and client-side this module are for displaying the state of a node in the browser. 

To develop a client I used my own libraries: Metro 4, ChartJS, DatetimeJS. If you are interested, you can find these libraries in my GitHub profile.

The Monitor server part can work separately, without running a client. But the client doesn't work without a server-side part. The launch of the Monitor is a very simple procedure. I’m going to talk about configuring and starting a client and server in one of the next videos. Also, you can read about launch options and starting variants in the project's README and HOW-TO files on GitHub.

***Show a picture with clients***

Currently, the client has two types of realization: single node, suitable for ordinary users  (included in the repo with server-side), and multiple nodes, suitable for users, who launch more than one node (named Cluster, you can find it in a separate repository).

What are the key features of Mina Monitor?

#### Monitor Client:
- Display of the main indicators of the Mina network (Block height, uptime, epoch, and slot info)
- Displaying the status of the node daemon (SYNCED, CATCHUP, BOOTSTRAP, ...)
- Displaying the health of node (OK, Fork, Hanging)
- Displaying the server resources utilization by the node (CPU, RAM, NETWORK)
- Displaying the balance of the specified address and the value of this balance in different currencies
- Displaying information about delegations to the specified validator address
- Displaying information about won blocks and rewards received in the current epoch
- Displaying the position in the uptime leaderboard
- Displaying the status of several nodes on one page
- Convenient live graphs for displaying utilized resources
- Responsive interface (It is comfortable to look at both PC and phone and tablet)
- Two-color themes: dark and light

#### Monitor Server Side:
- Monitoring node health
- Identification of critical node states (fork, hanging, no peers, ….)
- Determining the synchronization state of a node
- The automatic reboot of the node in case of critical state detection
- Sending messages about the critical state of the node to Telegram and/or to Discord
- Sending the current balance of the specified address to Telegram and/or to Discord
- Sending Mina's cost to Telegram and/or to Discord
- Controlling the snark-worker
- Using WebSocket connection to inform the client about node state

### Time to build Cluster

After launching the first version of the Mina Monitor and the beginning of using it, in my head was born a lot of new ideas on how I can improve an existing code to get new experiences, new useful information from nodes, and visualization it (for example: to show several nodes on one page, to show mina price, to show address position in the uptime leaderboard, and to show information about delegations and rewards).

Some ideas led me to the necessity to create a special client, who is showing several nodes on one page and information which I said above. This is how the Mina Monitor Cluster appeared.

In the process of working on the Cluster, there were also several ideas regarding the server-side of the monitor namely:

- Store node state in the internal cached object. It will stop provoking requests to GraphQL from the server when the client requests data from the server. The use of the cached state of the node allows to increases significantly the performance of the server-side and the speed of displaying information on the client. The ability to run multiple clients without sacrificing server performance.
- Improve the work with parameters that determine the time. Now you can set time values as short strings in human-readable format: for example “1d13h45m” instead of a big number to set a time in milliseconds the number of characters in which you still need to count correctly.
- Transit from HTTP to WebSockets for data exchange between client and server. We get one stable connection instead of a large number of constant small requests on HTTP and inform clients when data is ready.
- Improve the algorithm for recognizing a hanging node state. If the state of the fork is easy to determine, then it is not entirely trivial to determine correctly the hang state of a node.

Many other improvements were made in the work of the server-side.

As a result, the following functionality was added:

#### Cluster Client:
- Anything that a simple client displays, plus
- Displaying the status of several nodes on one page
- Displaying the response rate of a GraphQL node to the main request
- Increase and expand information about node uptime.
- Improvements for displaying Mina price, address balance

#### Extended features for the server-side
- I rewrote code to use WebSocket for exchange data between client and server
- Now client receives a stored state of the node from the cached object
- I added a Snark-worker controller to disabling snark-worker before block production and then resuming its work after block producing
- I added a checker for Monitor memory consumption and reboot node when memory is critical usage
- I added a module to control mina service stops with `journalctl` and wrote this event to the log file

One of the important changes in the extreme version is the use of WebSocket for communication between the client and server parts of the monitor (thanks to Alexander Nikiforov for pushing me to this). The transition from HTTP to WebSocket allowed to reduce the load on the processor both on the client and on the server and to reduce the number of network connections to one full-duplex for each monitored server.

### So which one to use

1) If you are a simple mina user and launched one node, you can use a simple client, included in the main Monitor repository. 
2) If you are a validator and want to stay in the top 120 in the uptime leaderboard, you can use a Cluster Client, because Cluster it's just more convenient for monitoring multiple nodes under one address.


### And lastly...
This is all that I wanted to say today. I have many ideas for further development: 

- the main thing is to make a full-fledged dashboard to control an unlimited number of nodes with support of different nodes types (block producer, snark worker, archive node, and simple node for address control)
- the ability to restart a node remotely via a command in a telegram or in a discord
- interaction between nodes for additional control of the states of nodes
- interaction with the newest API (for example StakeTab Dashboard API when this API will be published)

This was the first video in the series of videos about Mina Monitor. In the next video, I'm going to talk about how to configure and launch the server and the client of Mina Monitor.

Thanks to Mina’s team for the Mina blockchain. I really hope Monitor will be useful not only for me but for all Mina nodes operators. Thank you for your attention. I finished my speech.

THE END
