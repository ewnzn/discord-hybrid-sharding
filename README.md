<p align="center"><a href="https://nodei.co/npm/discord-hybrid-sharding/"><img src="https://nodei.co/npm/discord-hybrid-sharding.png"></a></p>
<p align="center"><img src="https://img.shields.io/npm/v/discord-hybrid-sharding"> <img src="https://img.shields.io/npm/dm/discord-hybrid-sharding?label=downloads"> <img src="https://img.shields.io/npm/l/discord-hybrid-sharding"> <img src="https://img.shields.io/github/repo-size/meister03/discord-hybrid-sharding">  <a href="https://discord.gg/YTdNBHh"><img src="https://discordapp.com/api/guilds/697129454761410600/widget.png" alt="Discord server"/></a></p>

# Discord-Hybrid-Sharding
The first package which combines sharding manager & internal sharding to save a lot of resources, which allows clustering!

In other words: "Mixing both: if you need `x` shards for `n` process!"

If you are interested in auto-scaling & cross-hosting on other machines, check out this package `npmjs.com/discord-cross-hosting`

## Why?
The sharding manager is very heavy and  uses more than 300MB per shard during light usage, while internal sharding uses just 20% of it. Internal sharding reaches its' limits at  14000 guilds and becomes slow when your bot gets bigger.

Your only solution becomes converting to the sharding manager. That's why this new package will solve all your problems (tested by many bots with 20-170k guilds), because it spawns shards, which has internal shards. **You can save up to 60% on resources!**

- **Decentralized ClusterEval function -> Listenerless, less memory leaks & cluster/client doesn't have to be ready**
- **Heartbeat System -> Respawn unresponsive or dead `ClusterClient`s**
- **IPC System -> Client <-> ClusterManager -> `.request()`, `.reply()`, `.send()`**
- Memory efficient -> 60% less memory when clustering
- Debug event -> A good overview of cluster information
- EvalOnManager function & other cool functions you need...
- `Strings` & `Functions with Context` support on `.broadcastEval()`
- Optional timeout feature on `.broadcastEval()` to prevent memory leaks
- **[Supports cross-hosting: `Shard/Cluster` managing and cross-host communication (`.broadcastEval()`, `IPC`)](https://npmjs.com/discord-cross-hosting)**
- **[Supports syncing Discord rate limits globally](https://npmjs.com/discord-cross-ratelimit)**
**Scroll down to check our new functions!**

## How does it work?
There are clusters/master shards, which behave like regular shards in the sharding manager and spawns clusters in addition to internal shards. You do not have to spawn as many regular Shards (master shards), which can be replaced with internal shards.
"for process `n` , `n` internal shards"

Example: `A Discord bot with 4000 guilds`
Normaly we would spawn 4 shards with the Sharding Manager (`~4 x 300MB memory`), but in this case we start with 2 clusters/master shards, which spawns 2 internal shards ==> We just saved 2 shards in comparison to the regular Sharding Manager (`~2 x 300MB memory`).

### See below for the Guide

**If you need help, feel free to join our <a href="https://discord.gg/YTdNBHh">Discord server</a>. ☺**

# Download
```cli
npm i discord-hybrid-sharding
------ or ---------------------
yarn add discord-hybrid-sharding
```

# Discord.js v13
- Full Discord.js v13 support
- `Strings` and `Functions` with `context` are supported in `.broadcastEval()`
- Most public methods accept sole objects, such as `.spawn({ amount: 20, timeout: -1 })`
- Very similar functions to the Discord.js ShardingManager and more for the advanced usage

# Setting up

**[Click to open documentation](https://sharding.js.org)**

First, add the module into your project (into your shard/cluster file).
Filename: `Cluster.js`
```js
const Cluster = require('discord-hybrid-sharding');

const manager = new Cluster.Manager(`${__dirname}/bot.js`, {
    totalShards: 7 , // or 'auto'
    /// Check below for more options
    shardsPerClusters: 2, 
    // totalClusters: 7,
    mode: 'process' ,  // you can also choose "worker"
    token: 'YOUR_TOKEN',
});

manager.on('clusterCreate', cluster => console.log(`Launched Cluster ${cluster.id}`));
manager.spawn({ timeout: -1 });
```

After that, insert the code below into your `bot.js` file
```js
const Cluster = require('discord-hybrid-sharding');
const Discord = require('discord.js');

const client = new Discord.Client({
    shards: Cluster.data.SHARD_LIST, // An array of shards that will get spawned
    shardCount: Cluster.data.TOTAL_SHARDS, // Total number of shards
});

client.cluster = new Cluster.Client(client); // initialize the Client, so we access the .broadcastEval()
client.login('YOUR_TOKEN');
```

# Eval-ing over clusters

*Following examples assume that your `Discord.Client` is called `client`.*

```js
client.cluster.broadcastEval(`this.guilds.cache.size`)
    .then(results => console.log(`${results.reduce((prev, val) => prev + val, 0)} total guilds`));

// or with a callback function
client.cluster.broadcastEval(c => c.guilds.cache.size)
    .then(results => console.log(`${results.reduce((prev, val) => prev + val, 0)} total guilds`));
```

# Cluster.Manager 
| Option | Type | Default | Description |
| ------------- | ------------- | ------------- | ------------- |
| totalShards | number or string | "auto" | Amount of internal shards which will be spawned |
| totalClusters | number or string | "auto" | Amount of processes/clusters which will be spawned |
| shardsPerClusters | number or string | - | Amount of shards which will be in one process/cluster |
| shardList | Array[number] | - | OPTIONAL - On cross-hosting or spawning specific shards you can provide a shardList of internal Shard IDs, which will get spawned |
| mode | "worker" or "process" | "worker" | Cluster.Manager mode for the processes |
| token | string | - | OPTIONAL -Bot token is only required totalShards are set to "auto" |

The Manager.spawn options are the same as for Sharding Manager

# Cluster Events
| Event |  Description |
| ------------- | -------------- |
| clusterCreate | Triggered when a Cluster gets spawned |

# Cluster Client Properties
All properties like for `.broadcastEval()` are available, just replace the `client.shard` with `client.cluster`
Other properties:
| Property |  Description |
| ------------- | -------------- |
| client.cluster.count | Returns the amount of all clusters |
| client.cluster.id | Returns the current cluster ID |
| client.cluster.ids | Returns all internal shards of the cluster |

**Feel free to contribute/suggest or contact me on my Discord server or in DM: Meister#9667**

# Changes | Migrating to Discord-Hybrid-Sharding

Options are now labelled as `cluster` instead of `shard`:
```diff
- client.shard...
+ client.cluster...

- .broadcastEval((c, context) => c.guilds.cache.get(context.guildId), { context: { guildId: '1234' }, shard: 0 })
+ .broadcastEval((c, context) => c.guilds.cache.get(context.guildId), { context: { guildId: '1234' }, cluster: 0 })
```
Small changes in naming conventions:
```diff
- client.shard.respawnAll({ shardDelay = 5000, respawnDelay = 500, timeout = 30000 })
+ client.cluster.respawnAll({ clusterDelay = 5000, respawnDelay = 5500, timeout = 30000 })

- manager.shard.respawnAll({ shardDelay = 5000, respawnDelay = 500, timeout = 30000 })
+ manager.respawnAll({ clusterDelay = 5000, respawnDelay = 5500, timeout = 30000 })

```
Get current cluster ID:
```diff
- client.shard.id
+ client.cluster.id
```
Get current shard ID:
```diff
- client.shard.id
+ message.guild.shardId
```
Get total shards count:
```diff
- client.shard.count
+ client.cluster.info.TOTAL_SHARDS
```
Get all ShardID's in the current cluster:
```diff
- client.shard.id
+ [...client.cluster.ids.keys()]
```

# New functions & events:

## `HeartbeatSystem`
- Checks if Cluster/Client sends a heartbeat on a given interval
- When the Client doesn't send a heartbeat, it will be marked as dead/unresponsive
- Cluster will get respawned after the given amount of missed heartbeats has been reached
```js
const manager = new Cluster.Manager(`${__dirname}/bot.js`, {
    totalShards: 8,
    shardsPerClusters: 2, 
    keepAlive: {
        interval: 2000, // Interval to send a heartbeat
        maxMissedHeartbeats: 5, // Maximum amount of missed Heartbeats until Cluster will get respawned
        maxClusterRestarts: 3 // Maximum Amount of restarts that can be performed in 1 hour in the HeartbeatSystem
    },
});
```

## `EvalOnCluster`
Decentralized ClusterClient eval function taht doesn't open any listeners and minimizes the risk of creating a memory leak during `.broadcastEval()`
- Build-in eval timeout which resolves after a given time
- No additional listeners - less memory leaks, better than `.broadCastEval()`
- Client & all clusters don't need to be ready
```js
client.cluster.evalOnCluster('this.cluster.id', { cluster: 0, timeout: 10000 });
```

## `IPC System`
* The IPC System allows you to listen to your messages
* You can communicate between the cluster and the client
* This allows you to send requests from the client to the cluster and reply to them and vice versa
* You can also send normal messages which do not need to be replied

ClusterManager | `cluster.js`
```js
const Cluster = require('discord-hybrid-sharding');
const manager = new Cluster.Manager(`${__dirname}/testbot.js`, {
    totalShards: 1,
    totalClusters: 1,
});

manager.on('clusterCreate', cluster => {
    cluster.on('message', message => {
    console.log(message);
        if (!message._sRequest) return; // Check if the message needs a reply
        message.reply({ content: 'hello world' });
    });
    setInterval(() => {
        cluster.send({ content: 'I am alive' }); // Send a message to the client
        cluster.request({ content: 'Are you alive?', alive: true }).then(e => console.log(e)); // Send a message to the client
    }, 5000);
});
manager.spawn({timeout: -1})
```

ClusterClient | `client.js`
```js
const Cluster = require('discord-hybrid-sharding');
const Discord = require('discord.js');
const client = new Discord.Client({
    shards: Cluster.data.SHARD_LIST, // An array of shards that will get spawned
    shardCount: Cluster.data.TOTAL_SHARDS, // Total number of shards
});

client.cluster = new Cluster.Client(client); 
client.cluster.on('message', message => {
    console.log(message);
    if(!message._sRequest) return; // Check if the message needs a reply
    if(message.alive) message.reply({ content: 'Yes I am!' }):
});
setInterval(() => {
    client.cluster.send({ content: 'I am alive as well!' });
}, 5000);
client.login('YOUR_TOKEN');
```
Evals a Script on the ClusterManager:
```
client.cluster.evalOnManager('process.memoryUsage().rss / 1024 ** 2');
```
Listen to debug messages and internal stuff:
```
manager.on('debug', console.log);
```
Optional Timeout on broadcastEval (Promise will get rejected after given time):
```
client.cluster.broadcastEval('new Promise((resolve, reject) => {})', { timeout: 10000 });
```

Open a PR/Issue when you need other Functions :)

# Bugs, glitches and sssues
If you encounter any problems feel free to open an issue in our <a href="https://github.com/meister03/discord-hybrid-sharding/issues">GitHub repository or join the Discord server.</a>

# Credits
Credits goes to the discord.js libary for the Base Code (See `changes.md`) and to this helpful [server](https://discord.gg/BpeedKh)
