+++
title = 'Tigerbeetle'
date = 2023-12-21T15:43:34+08:00
draft = true
tag = ["Unboxing"]
+++

# Tigerbeetle 是什麼?
> TigerBeetle is a **distributed financial accounting database** designed for mission critical safety and performance

* domain-specific database
* 4.3 MB!?

# Quickstart
1. Create TigerBeetle data file
``` bash
./tigerbeetle format --cluster=0 --replica=0 --replica-count=1 0_0.tigerbeetle
```

2. Start TigerBeetle server
``` bash
./tigerbeetle start --addresses=3000 0_0.tigerbeetle
```

3. Connect Client
``` bash
./tigerbeetle client --cluster=0 --addresses=3000
```

4. Create 2 accounts
``` bash
create_accounts id=1 code=10 ledger=700,
                id=2 code=10 ledger=700;
```

5. Transfer $10 between 2 accounts
``` bash
create_transfers id=1 debit_account_id=1 credit_account_id=2 amount=10 ledger=700 code=10;
```

6. Check transfer
``` bash
lookup_accounts id=1, id=2;
```

# Tigerbeetle Design
## Operations
* create_accounts
* create_transfers
* lookup_accounts
* lookup_transfers
* get_account_transfers 

## Batching **Operations** (a.k.a Coalesce events)
* Default batch size: 8190
* API Layer (like Connection pool)

## Client Session
* Client ID: 128-bit id
* Max client(hard limit): 32
* Lifecycle
    * send -> "register"
    * cluster commit -> "registered"
    * End Conditions: client terminates, evicted

## Consistency
* Isolation Level: **Strict Serializability**
* Sessions：客戶端會話的特性，例如只能有一個進行中的請求，讀取自己的寫入，觀察按順序發生的寫入等
    * At most one in-flight request
    * Read own writes
    * ```debits_posted```, ```credits_posted``` monotonically increasing
* Requests：請求的處理方式，比如每個請求在集群中最多執行一次，請求不會超時等
* Events：事件一旦提交將永遠保持提交，集群內對象時間戳的唯一性和嚴格遞增性等
    * A.timestamp ≠ B.timestamp
    * A.timestamp < B.timestamp
* Accounts：帳戶的不變性，特定 id 的唯一性，以及借記和貸記的平衡
* Transfers：轉賬的不變性，特定 id 的唯一性，以及轉賬超時的確定性
* Replay Order

# Data Model

# Benchmark 
``` bash
scripts/benchmark.sh
Formatting replica 0...
Starting replica 0...

Benchmarking...
steps [2/4] zig build-exe benchmark ReleaseSafe native-macos... LLVM Emit Object... info: Benchmark running against { 127.0.0.1:50899 }
info(message_bus): connected to replica 0
1234 batches in 36.58 s
load offered = 1000000 tx/s
load accepted = 273348 tx/s
batch latency p00 = 0 ms
batch latency p10 = 4 ms
batch latency p20 = 4 ms
batch latency p30 = 4 ms
batch latency p40 = 4 ms
batch latency p50 = 4 ms
batch latency p60 = 4 ms
batch latency p70 = 4 ms
batch latency p80 = 5 ms
batch latency p90 = 9 ms
batch latency p100 = 527 ms
transfer latency p00 = 0 ms
transfer latency p10 = 1153 ms
transfer latency p20 = 2922 ms
transfer latency p30 = 4927 ms
transfer latency p40 = 7144 ms
transfer latency p50 = 9875 ms
transfer latency p60 = 12617 ms
transfer latency p70 = 15974 ms
transfer latency p80 = 19401 ms
transfer latency p90 = 22649 ms
transfer latency p100 = 26607 ms

100 queries in 20.60 s
query latency p00 = 134 ms
query latency p10 = 155 ms
query latency p20 = 164 ms
query latency p30 = 172 ms
query latency p40 = 177 ms
query latency p50 = 183 ms
query latency p60 = 189 ms
query latency p70 = 195 ms
query latency p80 = 206 ms
query latency p90 = 214 ms
query latency p100 = 
```

# Reference

