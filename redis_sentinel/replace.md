root@server07-prod:/opt/kafka/kafka_2.12-3.9.0/config/kraft# redis-cli -p 26379 info Sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=10.10.0.175:6379,slaves=5,sentinels=6
root@server07-prod:/opt/kafka/kafka_2.12-3.9.0/config/kraft# redis-cli -p 26379 SENTINEL RESET mymaster
(integer) 1
root@server07-prod:/opt/kafka/kafka_2.12-3.9.0/config/kraft# redis-cli -p 26379 info Sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=10.10.0.175:6379,slaves=2,sentinels=3
