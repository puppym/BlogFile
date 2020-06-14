### goMaxProces和batchBlockNumber对geth-query速度的影响

### 逻辑核数P对geth-query速度影响

这里的协程数目为16个

1. 600W-610W  batchNumber=1000  goMaxProces=8

```bash
[^[[1;32mINFO ^[[0m] 2020-01-08 20:24:23 <ether-query>the time spent of all processBlocks 2h45m23.027042234s. startBlock 6000000. endBlock: 6100000
```

2. 600W-610W  batchNumber=1000  goMaxProces=10

```bash
[^[[1;32mINFO ^[[0m] 2020-01-08 22:45:9 <ether-query>the time spent of all processBlocks 2h20m37.571436315s. startBlock 6000000. endBlock: 6100000
```

3. 600W-610W  batchNumber=1000  goMaxProces=12

```bash
[^[[1;32mINFO ^[[0m] 2020-01-09 0:50:25 <ether-query>the time spent of all processBlocks 2h5m7.005384979s. startBlock 6000000. endBlock: 6100000

```

4. 600W-610W  batchNumber=1000  goMaxProces=14

```bash
[^[[1;32mINFO ^[[0m] 2020-01-09 2:55:57 <ether-query>the time spent of all processBlocks 2h5m23.496165041s. startBlock 6000000. endBlock: 6100000

```

5. 600W-610W  batchNumber=1000  goMaxProces=16

```bash
[^[[1;32mINFO ^[[0m] 2020-01-09 5:1:26 <ether-query>the time spent of all processBlocks 2h5m19.891477419s. startBlock 6000000. endBlock: 6100000

```

6. 600W-610W  batchNumber=2000  goMaxProces=10

```bash
[^[[1;32mINFO ^[[0m] 2020-01-08 22:45:9 <ether-query>the time spent of all processBlocks 2h20m37.571436315s. startBlock 6000000. endBlock: 6100000
```



7. 600W-610W  batchNumber=3000  goMaxProces=10

```bash
[^[[1;32mINFO ^[[0m] 2020-01-09 10:14:11 <ether-query>the time spent of all processBlocks 2h41m16.422652064s. startBlock 6000000. endBlock: 6100000

```



### 协程数目对geth-query速度影响

协程数目为8个



1. 600W-610W  batchNumber=1000  goMaxProces=8



2. 600W-610W  batchNumber=1000  goMaxProces=10



3. 600W-610W  batchNumber=1000  goMaxProces=12



4. 600W-610W  batchNumber=1000  goMaxProces=14



5. 600W-610W  batchNumber=1000  goMaxProces=16



6. 600W-610W  batchNumber=2000  goMaxProces=10



7. 600W-610W  batchNumber=3000  goMaxProces=10



协程数目为32个



1. 600W-610W  batchNumber=1000  goMaxProces=8



2. 600W-610W  batchNumber=1000  goMaxProces=10



3. 600W-610W  batchNumber=1000  goMaxProces=12



4. 600W-610W  batchNumber=1000  goMaxProces=14



5. 600W-610W  batchNumber=1000  goMaxProces=16



6. 600W-610W  batchNumber=2000  goMaxProces=10



7. 600W-610W  batchNumber=3000  goMaxProces=10









1. 600W-620W  batchNumber=2000  goMaxProces=8



2. 600W-620W  batchNumber=2000  goMaxProces=10



3. 600W-620W  batchNumber=2000  goMaxProces=12



4. 600W-620W  batchNumber=2000  goMaxProces=14



5. 600W-620W  batchNumber=2000  goMaxProces=16



6. 600W-620W  batchNumber=1000  goMaxProces=10



7. 600W-620W  batchNumber=5000  goMaxProces=10

