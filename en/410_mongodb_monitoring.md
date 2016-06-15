##MongoDB monitoring

MongoDB performance monitor plugin for Open-Falcon

##Function Supported

Tested version: Version MongoDB 2.4, 2.6 3.0, 3.2, and Percona MongoDB3.0 are supported.

Storage engine supported: MMAPv1, wiredTiger, RocksDB, PerconaFT.
 (indexes of some storage engines are not collected, and can be added in the code).
 
 Chondrophone: standlone, replica sets, zone clusters

Node supported: mongo data nodes, configuration nodes, Primary/Secondary, mongos. Arbite nodes are not supported.

Monitoring data acquisition principle:
1.survival monitoring: auth included.

2.serverStatus

3.replSetGetStatus

4.oplog.rs

5.mongos

Collect and report per minute by cron, and collecting makes no difference to MongoDB theory.

##Environmental requirements:
```
Operating system: Linux
Python 2.6
PyYAML > 3.10
python-requests > 0.11
```

##Mongomo deploy:

1.Decompress to directory /path/to/mongomon.

2.Deploy the MongoDB multi-case (mongod, configuration node, mongos) information of present server, /path/to/mongomon/conf/mongomon.conf. Record an instance per line: port, user name, password 
	{port: 27017, user: "",password: ""}
    
3.Deploy crontab, and modify the mongomon installation path of file mongomon/conf/mongomon_cron to cp mongomon_cron /etc/cron.d/.

4.Check the MongoDB metric in the dashboard of open-falcon.

5.The default endpoint is hostname.

##MongoDB falcon screen:

##picture
##MongoDB index collected:




| Counters | Type | Notes |
| -- | -- | -- |
| mongo_local_alive | GAUGE | Mongodb survival local monitoring, if unlocked Auth，Successful connection authentication is required. |
| asserts_msg | COUNTER | Message asserts quantity /second |
| asserts_regular | COUNTER | Regular asserts quantity/second |
| asserts_rollovers | COUNTER | The times of counter roll over /second, and the counter will be reset every 2^30 asserts |
| asserts_user | COUNTER | User asserts quantity/second |
| asserts_warning | COUNTER | Warning asserts quantity/second |
| page_faults | COUNTER | Page faults times/second |
| connections_available | GAUGE | Unused available connection quantity |
| connections_current | GAUGE | Connected connection quantity of all current clients |
| connections_used_percent | GAUGE | Used connection quantity percent |
| connections_totalCreated | COUNTER | New created connection quantity/second |
| globalLock_currentQueue_total | GAUGE | Operating quantity waits to be locked in current queue |
| globalLock_currentQueue_readers | GAUGE | Operating quantity waits to be read lock in current queue |
| globalLock_currentQueue_writers | GAUGE | Operating quantity waits to write lock in current queue |
| locks_Global_acquireCount_ISlock | COUNTER | Instance level intent shared lock acquisition times |
| locks_Global_acquireCount_IXlock | COUNTER | Instance level intent exclusive shared lock acquisition times |
| locks_Global_acquireCount_Slock | COUNTER | Instance level shared lock acquisition times |
| locks_Global_acquireCount_Xlock | COUNTER | Instance level exclusive lock acquisition times |
| locks_Global_acquireWaitCount_ISlock | COUNTER | Instance level intent shared lock waiting times |
| locks_Global_acquireWaitCount_IXlock | COUNTER| Instance level intent exclusive lock waiting times |
| locks_Global_timeAcquiringMicros_ISlock | COUNTER | Instance level shared lock acquisition time-consuming Unit: us |
| locks_Global_timeAcquiringMicros_IXlock | COUNTER | Instance level exclusive acquisition time-consuming Unit: us |
| locks_Database_acquireCount_ISlock | COUNTER | Database level intent shared lock acquisition times |
| locks_Database_acquireCount_IXlock | COUNTER | Database level intent exclusive lock acquisition times |
| locks_Database_acquireCount_Slock | COUNTER | Database level shared lock acquisition times |
| locks_Database_acquireCount_Xlock | COUNTER | Database level exclusive lock acquisition times |
| locks_Collection_acquireCount_ISlock | COUNTER | Set level intent shared lock acquisition times |
| locks_Collection_acquireCount_IXlock | COUNTER | Set level intent exclusive lock acquisition times |
| locks_Collection_acquireCount_Xlock | COUNTER | Set level exclusive lock acquisition times |
| opcounters_command | COUNTER | All commands executed by database/second |
| opcounters_insert | COUNTER | Insert operation times executed by database/second |
| opcounters_delete | COUNTER | Delete operation times executed by database/second |
| opcounters_update | COUNTER | Update operation times executed by database/second |
| opcounters_query | COUNTER | Query operation times executed by database/second |
| opcounters_getmore | COUNTER | Getmore operation times executed by database/second |
| opcountersRepl_command | COUNTER | All command times copied and executed by database/second |
| opcountersRepl_insert | COUNTER | Insert command times copied and executed by database/second  |
| opcountersRepl_delete | COUNTER | Delete command times copied and executed by database/second  |
| opcountersRepl_update | COUNTER | Update command times copied and executed by database/second  |
| opcountersRepl_query | COUNTER | Query command times copied and executed by database/second |
| opcountersRepl_getmore | COUNTER | Getmore command times copied and executed by database/second  |
| network_bytesIn | COUNTER | Network transmission bytes received by database/second |
| network_bytesOut | COUNTER | Network transmission bytes sent by database/second |
| network_numRequests | COUNTER | Request times received by database/second |
| mem_virtual | GAUGE | Virtual memory used by database process  |
| mem_resident | GAUGE | Physical memory used by database process  |
| mem_mapped | GAUGE | Mapped memory, only used formmapv1 storage engine |
| mem_bits | GAUGE | 64 or 32bit |
| mem_mappedWithJournal | GAUGE | Map memory consumed by journal, only used formmapv1 storage engine |
| backgroundFlushing_flushes | COUNTER | Refresh writes times to disk by database/second  |
| backgroundFlushing_average_ms | GAUGE | Average time-consuming of refresh writes to disk by database,Unit ms |
| backgroundFlushing_last_ms | COUNTER | Current latest time-consuming of refresh writes to disk by database,Unit ms |
| backgroundFlushing_total_ms | GAUGE | Total time-consuming of refresh writes to disk by database,Unit ms |
| cursor_open_total | GAUGE | Total cursor quantity maintained for clients by current database  |
| cursor_timedOut | COUNTER | Cursor quantity of database timout/second  |
| cursor_open_noTimeout | GAUGE | Cursor quantity of setting dbquery.Option.notimeout |
| cursor_open_pinned | GAUGE | Cursor quantity of opened pinned |
| repl_health | GAUGE | Copied healthy status |
| repl_myState | GAUGE | Duplicate sets status of current node  |
| repl_oplog_window | GAUGE | Size of oplog window |
| repl_optime | GAUGE | Time of last execution |
| replication_lag_percent | GAUGE | Delayed percent(lag/oplog_window) |
| repl_lag | GAUGE | Secondary copy delay,Unit s |
| shards_size | GAUGE | Zone number of database cluster; config.shards.count |
| shards_mongosSize | GAUGE | Mongos node number of database cluster; config.mongos.count |
| shards_chunkSize | GAUGE | Chunksize size setting of database cluster，acquired in config.settings |
| shards_activeWindow | GAUGE | Whether the time window is setup to the data balancer of database cluster，1/0 |
| shards_activeWindow_start | GAUGE | Start time of time window of the data balancer of database cluster ,format 23.30 indicates 23：30 |
| shards_activeWindow_stop | GAUGE | End time of time window of the data balancer of database cluster ,format 23.30 indicates 23：30 |
| shards_BalancerState | GAUGE | The status of data balancer of database cluster,Whether it is open. |
| shards_isBalancerRunning | GAUGE | Whether the data balancer of database cluster is taking block migration |
| wt_cache_used_total_bytes | GAUGE | The total bytes of wiredtiger cache |
| wt_cache_dirty_bytes | GAUGE | The bytes of "dirty" data in wiredtiger cache |
| wt_cache_readinto_bytes | COUNTER | The bytes of database writing Into wiredtiger cache/second |
| wt_cache_writtenfrom_bytes | COUNTER | The bytes of database writing into the disk from wiredtiger cache/second |
| wt_concurrentTransactions_write | GAUGE | Write tickets available to the wiredtiger storage engine |
| wt_concurrentTransactions_read | GAUGE | Read tickets available to the wiredtiger storage engine |
| wt_bm_bytes_read | COUNTER | The bytes of block-manager read/second |
| wt_bm_bytes_written | COUNTER | The bytes of block-manager write/second |
| wt_bm_blocks_read | COUNTER | The number of block-manager read/second |
| wt_bm_blocks_written | COUNTER | The number of block-manager write/second |
| rocksdb_num_immutable_mem_table | 
| rocksdb_mem_table_flush_pending | 
| rocksdb_compaction_pending |
| rocksdb_background_errors | 
| rocksdb_num_entries_active_mem_table | 
| rocksdb_num_entries_imm_mem_tables |
| rocksdb_num_snapshots |
| rocksdb_oldest_snapshot_time | 
| rocksdb_num_live_versions |
| rocksdb_total_live_recovery_units | 
| PerconaFT_cachetable_size_current |
| PerconaFT_cachetable_size_limit |
| PerconaFT_cachetable_size_writing | 
| PerconaFT_checkpoint_count | 
| PerconaFT_checkpoint_time |
| PerconaFT_checkpoint_write_leaf_bytes_compressed |
| PerconaFT_checkpoint_write_leaf_bytes_uncompressed | 
| PerconaFT_checkpoint_write_leaf_count | 
| PerconaFT_checkpoint_write_leaf_time | 
| PerconaFT_checkpoint_write_nonleaf_bytes_compressed | 
| PerconaFT_checkpoint_write_nonleaf_bytes_uncompressed | 
| PerconaFT_checkpoint_write_nonleaf_count | 
| PerconaFT_checkpoint_write_nonleaf_time |
| PerconaFT_compressionRatio_leaf |
| PerconaFT_compressionRatio_nonleaf |
| PerconaFT_compressionRatio_overall | 
| PerconaFT_fsync_count | 
| PerconaFT_fsync_time |
| PerconaFT_log_bytes |
| PerconaFT_log_count | 
| PerconaFT_log_time |
| PerconaFT_serializeTime_leaf_compress | 
| PerconaFT_serializeTime_leaf_decompress | 
| PerconaFT_serializeTime_leaf_deserialize | 
| PerconaFT_serializeTime_leaf_serialize | 
| PerconaFT_serializeTime_nonleaf_compress | 
| PerconaFT_serializeTime_nonleaf_decompress | 
| PerconaFT_serializeTime_nonleaf_deserialize | 
| PerconaFT_serializeTime_nonleaf_serialize | 


##Monitoring alarm items recommended to set:

##Instruction: system level monitoring items are provided by falcon agent; monitoring triggering conditions are self-adjusted by the scenes.

| alarm command |
| -- |
| load.1min>10 |
| cpu.idle<10 |
| df.bytes.free.percent<30|
| df.bytes.free.percent<10 |
| mem.memfree.percent<20 |
| mem.memfree.percent<10 |
| mem.memfree.percent<5 |
| mem.swapfree.percent<50 |
| mem.memused.percent>=50 |
| mem.memused.percent>=10 |
| net.if.out.bytes>94371840 |
| net.if.in.bytes>94371840 |
| disk.io.util>90 |
| mongo_local_alive=0 |
| page_faults>100 |
| connections_current>5000 |
| connections_used_percent>60 |
| connections_used_percent>80 |
| connections_totalCreated>1000 |
| globalLock_currentQueue_total>10 |
| globalLock_currentQueue_readers>10 |
| globalLock_currentQueue_writers>10 |
| opcounters_command |
| opcounters_insert |
| opcounters_delete |
| opcounters_update |
| opcounters_query |
| opcounters_getmore |
| opcountersRepl_command |
| opcountersRepl_insert |
| opcountersRepl_delete |
| opcountersRepl_update |
| opcountersRepl_query |
| opcountersRepl_getmore |
| network_bytesIn |
| network_bytesOut |
| network_numRequests |
| repl_health=0 |
| repl_myState not 1/2/7 |
| repl_oplog_window<168 |
| repl_oplog_window<48 |
| replication_lag_percent>50 |
| repl_lag>60 |
| repl_lag>300 |
| shards_mongosSize |

##Contributors

* Zhuo Rulin: mail:zhuorulintec@163.com; weibo: http://weibo.com/u/2540962412
