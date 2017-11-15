`Monitoring`
------------

Each module internally generates it's own list of counters using `ThreadData`
library (which has supports for stats like SUM/RATE/COUNT over various
durations). These counters are periodically submitted to `ZmqMonitor` store
(singleton) which in turn publishes them on it's `PUB` channel. Monitoring agent
on the node or remote note can connect to `ZmqMonitor` and pull off the counters
to perform monitoring on the top of these counters or build interactive
dashboards.

Further each module logs important events like `IFACE_UP` or `NEIGHBOR_DOWN`
in a structured fashion via ZmqMonitor which can be logged to data stores
like `Druid` to do realtime monitoring of log events across the fleet.


### Understanding Counters
---

Counters are exported as key-value pairs where key is string and value is 64 bit
integers. Keys represent either a plain counter like number of neighbors or
statistic like number of SPF computation in last one minute. Stats are encoded
with following method

`<module-name>.<counter-name>.<stat-type>.<seconds>`

For e.g.
- `kvstore.received_key_vals.sum.60` represents number of key-value updates
  happened in past one minute
- `decision.spf_runs.count.3600` represents number of SPF runs in last one hour
- `decision.spf.multipath_ms.avg.0` represent average execution time across all
  SPF calculations


### Important Counters to Monitor
---

Here are some important counters to monitor for production system and alert
engineers quickly if something goes wrong. `breeze monitor counters` will list
you lot of other counters as well, most names are self explanatory.

#### KvStore Counters
- `kvstore.num_keys` => This counter shouldn't exceed a certain threshold and
  must have some max limit for a given network. If number of keys keep
  increasing in KvStore that means system will soon run out of memory
- `kvstore.peers` => Usually every node in a network must have atleast one peer
- `kvstore.pending_full_sync` => Pending full sync request to neighbor, this
  counter should be 0 most of time

#### Spark Counters
- `spark.num_tracked_interfaces` => Indicates number of interfaces learned by
  OpenR
- `spark.num_adjacent_neighbors` must match with `spark.num_tracked_neighbors`
  as eventually adjacency must form between all connected nodes in domain

#### Decision Counters

- `decision.adj_db_update.count.60` shouldn't exceed certain threshold. Higher
  number indicates instable network
- `decision.prefix_db_update.count.60` shouldn't exceed certain threshold.
  Higher number indicates lot of churn in route advertisement
- `decision.spf_runs.count.60` higher number indicates lot of network churn
  (corresponds to adj_db_update).

#### Fib Counters

- `fib.convergence_time_ms.avg.60` indicates average convergece time for all
  events in last one minute.
- `fib.num_routes` should correspond to number of unique advertised prefixes
  across all nodes

#### Link Monitor Counters

- `link_monitor.advertise_adjacencies.sum.60` => higher number indicates lot of
  adjacency flapping
- `link_monitor.advertise_links.sum.60` => higher number indicates lot of link
  flapping on system

### Log Events
---

Along with counters OpenR also publishes certain log events. Each log event is a
json sample described as dictionary of

`<value-type> => map<key, value>`

Each sample has following keys along with more information
- `int` key named `time` indicating timestamp since epoch in number of seconds.
- `string` key named `domain` indicating network name
- `string` key named `name` indicating name of the node
- `string` key named `event` indicating event name

Some important evens names are
- `ROUTE_CALC`
- `ROUTE_UPDATE`
- `NB_RTT_CHANGE`
- `KEY_EXPIRE`
- `IFACE_UPDATE`
- `IFACE_DOWN`
- `IFACE_UP`
- `DECISION_DEBOUNCE`
- `ROUTE_CONVERGENCE`
- `NB_UP`
- `NB_DOWN`
- `NB_RESTART`
- `ADD_PEER`
- `DEL_PEER`