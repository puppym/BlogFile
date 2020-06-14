

### 并发优化的总体思路

1. 对需要重放的区块数量平均分成cpu逻辑内核的份数，然后每一份启动一个协程开始重放区块数据。
2. 构建一个协程池，协程池内的协程数目为cpu的核数，然后定义一个batchBlockNum，每次向协程池中的空余协程投放batchBlockNum个block。因为每一个协程都在大量读写磁盘，这里使用协程池限制协程数量主要是因为磁盘I/O的限制。

对于思路1主要是在区块数据中，前面区块数据运行较快，后面区块数据运行慢。通过平分区块数量不能达到平分计算量的效果，因此会导致运行前面区块的协程早已运行完毕，运行后面区块的协程还在继续运行，并且此时CPU的逻辑核没有用完，导致CPU利用效率不高。下面是在服务器上运行0-500W的时间：

```golang
[^[[1;32mINFO ^[[0m] 2019-12-27 20:48:49 <ether-query>the time spent of all processBlocks 31m59.450883952s.
[^[[1;32mINFO ^[[0m] 2019-12-27 20:48:49 <ether-query>[100.00 precent]Processing block 312000 finished, total elaspe=31m59.454650389s, lastest 1000 block elaspe=8.374434536s
[^[[1;32mINFO ^[[0m] 2019-12-27 21:2:8 <ether-query>the time spent of all processBlocks 45m18.21689252s.
[^[[1;32mINFO ^[[0m] 2019-12-27 21:2:8 <ether-query>[100.00 precent]Processing block 624000 finished, total elaspe=45m18.221932769s, lastest 1000 block elaspe=6.823886852s
[^[[1;32mINFO ^[[0m] 2019-12-27 21:38:45 <ether-query>the time spent of all processBlocks 1h21m55.570234234s.
[^[[1;32mINFO ^[[0m] 2019-12-27 21:38:45 <ether-query>[100.00 precent]Processing block 936000 finished, total elaspe=1h21m55.601807422s, lastest 1000 block elaspe=12.774735824s
[^[[1;32mINFO ^[[0m] 2019-12-27 21:59:33 <ether-query>the time spent of all processBlocks 1h42m43.013076624s.
[^[[1;32mINFO ^[[0m] 2019-12-27 21:59:33 <ether-query>[100.00 precent]Processing block 1248000 finished, total elaspe=1h42m43.013919542s, lastest 1000 block elaspe=15.897518063s
[^[[1;32mINFO ^[[0m] 2019-12-27 22:24:45 <ether-query>the time spent of all processBlocks 2h7m55.226313093s.
[^[[1;32mINFO ^[[0m] 2019-12-27 22:24:45 <ether-query>[100.00 precent]Processing block 1560000 finished, total elaspe=2h7m55.25238175s, lastest 1000 block elaspe=32.432742563s
[^[[1;32mINFO ^[[0m] 2019-12-27 22:59:59 <ether-query>the time spent of all processBlocks 2h43m8.922163609s.
[^[[1;32mINFO ^[[0m] 2019-12-27 22:59:59 <ether-query>[100.00 precent]Processing block 3120000 finished, total elaspe=2h43m8.923114209s, lastest 1000 block elaspe=31.521875552s
[^[[1;32mINFO ^[[0m] 2019-12-27 23:1:42 <ether-query>the time spent of all processBlocks 2h44m51.846466404s.
[^[[1;32mINFO ^[[0m] 2019-12-27 23:1:42 <ether-query>[100.00 precent]Processing block 2184000 finished, total elaspe=2h44m51.847273449s, lastest 1000 block elaspe=28.426876223s
[^[[1;32mINFO ^[[0m] 2019-12-27 23:5:44 <ether-query>the time spent of all processBlocks 2h48m54.432008799s.
[^[[1;32mINFO ^[[0m] 2019-12-27 23:5:44 <ether-query>[100.00 precent]Processing block 1872000 finished, total elaspe=2h48m54.433244116s, lastest 1000 block elaspe=21.001653109s
[^[[1;32mINFO ^[[0m] 2019-12-27 23:56:20 <ether-query>the time spent of all processBlocks 3h39m29.817370157s.
[^[[1;32mINFO ^[[0m] 2019-12-27 23:56:20 <ether-query>[100.00 precent]Processing block 3432000 finished, total elaspe=3h39m29.818207019s, lastest 1000 block elaspe=50.646041945s
[^[[1;32mINFO ^[[0m] 2019-12-28 2:14:55 <ether-query>the time spent of all processBlocks 5h58m5.531759302s.
[^[[1;32mINFO ^[[0m] 2019-12-28 2:14:55 <ether-query>[100.00 precent]Processing block 3744000 finished, total elaspe=5h58m5.532706523s, lastest 1000 block elaspe=2m3.783968869s
[^[[1;32mINFO ^[[0m] 2019-12-28 7:41:15 <ether-query>the time spent of all processBlocks 11h24m25.464511535s.
[^[[1;32mINFO ^[[0m] 2019-12-28 7:41:15 <ether-query>[100.00 precent]Processing block 2808000 finished, total elaspe=11h24m25.481833152s, lastest 1000 block elaspe=25.471121791s
[^[[1;32mINFO ^[[0m] 2019-12-28 13:5:33 <ether-query>the time spent of all processBlocks 16h48m43.00335445s.
[^[[1;32mINFO ^[[0m] 2019-12-28 13:5:33 <ether-query>[100.00 precent]Processing block 4056000 finished, total elaspe=16h48m43.026951754s, lastest 1000 block elaspe=3m2.863890358s
[^[[1;32mINFO ^[[0m] 2019-12-28 14:8:24 <ether-query>the time spent of all processBlocks 17h51m34.033911073s.
[^[[1;32mINFO ^[[0m] 2019-12-28 14:8:24 <ether-query>[100.00 precent]Processing block 2496000 finished, total elaspe=17h51m34.034728524s, lastest 1000 block elaspe=18.203788033s

```

上面数据显示16个协程中已经跑完了12个，还有4个协程正在运行，但是此时CPU多核没有被充分利用，因此导致后面四个线程执行很慢。



#### 优化结果

通过协程池，将计算任务切分细化，反复投入协程池中计算，充分利用CPU多核资源。不会出现上述协程数量少，没有充分利用多核CPU资源的现象。下面是是在本地小米笔记本上运行0-150W个块的结果。能够在27分钟内跑完。

```golang
[^[[1;32mINFO ^[[0m] 2019-12-28 11:22:5 <ether-query>[100.00 precent]Processing block 50000 finished, total elaspe=1m52.7260788s, lastest 1000 block elaspe=4.8261719s
[^[[1;32mINFO ^[[0m] 2019-12-28 11:24:1 <ether-query>the time spent of all processBlocks 3m48.8266052s.
[^[[1;32mINFO ^[[0m] 2019-12-28 11:24:1 <ether-query>[100.00 precent]Processing block 100000 finished, total elaspe=3m48.8273116s, lastest 1000 block elaspe=5.3085744s
[^[[1;32mINFO ^[[0m] 2019-12-28 11:24:48 <ether-query>the time spent of all processBlocks 4m36.0259237s.
[^[[1;32mINFO ^[[0m] 2019-12-28 11:24:48 <ether-query>[100.00 precent]Processing block 150000 finished, total elaspe=4m36.0259784s, lastest 1000 block elaspe=6.807896s
[^[[1;32mINFO ^[[0m] 2019-12-28 11:26:27 <ether-query>the time spent of all processBlocks 6m15.0691613s.
[^[[1;32mINFO ^[[0m] 2019-12-28 11:26:27 <ether-query>[100.00 precent]Processing block 200000 finished, total elaspe=6m15.0692211s, lastest 1000 block elaspe=7.74742s
[^[[1;32mINFO ^[[0m] 2019-12-28 11:26:48 <ether-query>the time spent of all processBlocks 6m36.493572s.
[^[[1;32mINFO ^[[0m] 2019-12-28 11:26:48 <ether-query>[100.00 precent]Processing block 250000 finished, total elaspe=6m36.4947711s, lastest 1000 block elaspe=11.2663397s
[^[[1;32mINFO ^[[0m] 2019-12-28 11:27:10 <ether-query>the time spent of all processBlocks 6m57.9905741s.
[^[[1;32mINFO ^[[0m] 2019-12-28 11:27:10 <ether-query>[100.00 precent]Processing block 300000 finished, total elaspe=6m57.9906727s, lastest 1000 block elaspe=9.6412666s
[^[[1;32mINFO ^[[0m] 2019-12-28 11:27:25 <ether-query>the time spent of all processBlocks 7m12.6182014s.
[^[[1;32mINFO ^[[0m] 2019-12-28 11:27:25 <ether-query>[100.00 precent]Processing block 350000 finished, total elaspe=7m12.6183005s, lastest 1000 block elaspe=11.7795113s
[^[[1;32mINFO ^[[0m] 2019-12-28 11:27:46 <ether-query>the time spent of all processBlocks 7m34.4092998s.
[^[[1;32mINFO ^[[0m] 2019-12-28 11:27:46 <ether-query>[100.00 precent]Processing block 400000 finished, total elaspe=7m34.4093556s, lastest 1000 block elaspe=9.8950875s
[^[[1;32mINFO ^[[0m] 2019-12-28 11:29:29 <ether-query>the time spent of all processBlocks 7m24.7914709s.
[^[[1;32mINFO ^[[0m] 2019-12-28 11:29:30 <ether-query>[100.00 precent]Processing block 450000 finished, total elaspe=7m24.7939269s, lastest 1000 block elaspe=11.0956565s
[^[[1;32mINFO ^[[0m] 2019-12-28 11:32:6 <ether-query>the time spent of all processBlocks 8m4.771902s.
[^[[1;32mINFO ^[[0m] 2019-12-28 11:32:6 <ether-query>[100.00 precent]Processing block 500000 finished, total elaspe=8m4.7719688s, lastest 1000 block elaspe=8.8180755s
[^[[1;32mINFO ^[[0m] 2019-12-28 11:34:30 <ether-query>the time spent of all processBlocks 8m3.0453104s.
[^[[1;32mINFO ^[[0m] 2019-12-28 11:34:30 <ether-query>[100.00 precent]Processing block 600000 finished, total elaspe=8m3.0454871s, lastest 1000 block elaspe=7.8598099s
[^[[1;32mINFO ^[[0m] 2019-12-28 11:34:31 <ether-query>the time spent of all processBlocks 9m42.79327s.
[^[[1;32mINFO ^[[0m] 2019-12-28 11:34:31 <ether-query>[100.00 precent]Processing block 550000 finished, total elaspe=9m42.7934115s, lastest 1000 block elaspe=7.3706034s
[^[[1;32mINFO ^[[0m] 2019-12-28 11:34:46 <ether-query>the time spent of all processBlocks 7m57.0929712s.
[^[[1;32mINFO ^[[0m] 2019-12-28 11:34:46 <ether-query>[100.00 precent]Processing block 650000 finished, total elaspe=7m57.0930377s, lastest 1000 block elaspe=11.7731954s
[^[[1;32mINFO ^[[0m] 2019-12-28 11:38:31 <ether-query>the time spent of all processBlocks 11m21.1237646s.
[^[[1;32mINFO ^[[0m] 2019-12-28 11:38:31 <ether-query>[100.00 precent]Processing block 700000 finished, total elaspe=11m21.1238307s, lastest 1000 block elaspe=25.1700576s
[^[[1;32mINFO ^[[0m] 2019-12-28 11:43:5 <ether-query>the time spent of all processBlocks 15m40.2572939s.
[^[[1;32mINFO ^[[0m] 2019-12-28 11:43:5 <ether-query>[100.00 precent]Processing block 750000 finished, total elaspe=15m40.2580111s, lastest 1000 block elaspe=8.7452299s
[^[[1;32mINFO ^[[0m] 2019-12-28 11:43:22 <ether-query>the time spent of all processBlocks 15m35.8469396s.
[^[[1;32mINFO ^[[0m] 2019-12-28 11:43:22 <ether-query>[100.00 precent]Processing block 800000 finished, total elaspe=15m35.8469963s, lastest 1000 block elaspe=9.5661573s
[^[[1;32mINFO ^[[0m] 2019-12-28 11:44:57 <ether-query>the time spent of all processBlocks 12m51.1912258s.
[^[[1;32mINFO ^[[0m] 2019-12-28 11:44:57 <ether-query>[100.00 precent]Processing block 900000 finished, total elaspe=12m51.1913453s, lastest 1000 block elaspe=16.7695408s
[^[[1;32mINFO ^[[0m] 2019-12-28 11:47:7 <ether-query>the time spent of all processBlocks 17m37.141099s.
[^[[1;32mINFO ^[[0m] 2019-12-28 11:47:7 <ether-query>[100.00 precent]Processing block 850000 finished, total elaspe=17m37.1411707s, lastest 1000 block elaspe=12.9488595s
[^[[1;32mINFO ^[[0m] 2019-12-28 11:52:31 <ether-query>the time spent of all processBlocks 18m1.2477838s.
[^[[1;32mINFO ^[[0m] 2019-12-28 11:52:31 <ether-query>[100.00 precent]Processing block 950000 finished, total elaspe=18m1.247891s, lastest 1000 block elaspe=22.7486088s
[^[[1;32mINFO ^[[0m] 2019-12-28 11:53:19 <ether-query>the time spent of all processBlocks 18m32.9316895s.
[^[[1;32mINFO ^[[0m] 2019-12-28 11:53:19 <ether-query>[100.00 precent]Processing block 1050000 finished, total elaspe=18m32.9322085s, lastest 1000 block elaspe=27.3946238s
[^[[1;32mINFO ^[[0m] 2019-12-28 11:53:34 <ether-query>the time spent of all processBlocks 19m3.2116665s.
[^[[1;32mINFO ^[[0m] 2019-12-28 11:53:34 <ether-query>[100.00 precent]Processing block 1000000 finished, total elaspe=19m3.2118003s, lastest 1000 block elaspe=27.9216245s
[^[[1;32mINFO ^[[0m] 2019-12-28 12:0:25 <ether-query>the time spent of all processBlocks 21m53.9529791s.
[^[[1;32mINFO ^[[0m] 2019-12-28 12:0:25 <ether-query>[100.00 precent]Processing block 1100000 finished, total elaspe=21m53.9530519s, lastest 1000 block elaspe=36.246237s
[^[[1;32mINFO ^[[0m] 2019-12-28 12:9:45 <ether-query>the time spent of all processBlocks 26m40.5610668s.
[^[[1;32mINFO ^[[0m] 2019-12-28 12:9:45 <ether-query>[100.00 precent]Processing block 1150000 finished, total elaspe=26m40.5641365s, lastest 1000 block elaspe=37.3508872s
[^[[1;32mINFO ^[[0m] 2019-12-28 12:11:58 <ether-query>the time spent of all processBlocks 28m35.6333611s.
[^[[1;32mINFO ^[[0m] 2019-12-28 12:11:58 <ether-query>[100.00 precent]Processing block 1200000 finished, total elaspe=28m35.6334354s, lastest 1000 block elaspe=27.2829537s
[^[[1;32mINFO ^[[0m] 2019-12-28 12:14:44 <ether-query>the time spent of all processBlocks 29m47.1777318s.
[^[[1;32mINFO ^[[0m] 2019-12-28 12:14:44 <ether-query>[100.00 precent]Processing block 1250000 finished, total elaspe=29m47.1777815s, lastest 1000 block elaspe=25.78786s
[^[[1;32mINFO ^[[0m] 2019-12-28 12:14:51 <ether-query>the time spent of all processBlocks 27m44.0462302s.
[^[[1;32mINFO ^[[0m] 2019-12-28 12:14:51 <ether-query>[100.00 precent]Processing block 1300000 finished, total elaspe=27m44.0463309s, lastest 1000 block elaspe=22.1827759s
[^[[1;32mINFO ^[[0m] 2019-12-28 12:17:33 <ether-query>the time spent of all processBlocks 25m2.0519017s.
[^[[1;32mINFO ^[[0m] 2019-12-28 12:17:33 <ether-query>[100.00 precent]Processing block 1350000 finished, total elaspe=25m2.0519585s, lastest 1000 block elaspe=15.6108034s
[^[[1;32mINFO ^[[0m] 2019-12-28 12:17:52 <ether-query>the time spent of all processBlocks 24m32.9954046s.
[^[[1;32mINFO ^[[0m] 2019-12-28 12:17:52 <ether-query>[100.00 precent]Processing block 1400000 finished, total elaspe=24m32.9954672s, lastest 1000 block elaspe=10.1273727s
[^[[1;32mINFO ^[[0m] 2019-12-28 12:19:15 <ether-query>the time spent of all processBlocks 25m41.3141391s.
[^[[1;32mINFO ^[[0m] 2019-12-28 12:19:15 <ether-query>[100.00 precent]Processing block 1450000 finished, total elaspe=25m41.314202s, lastest 1000 block elaspe=12.9553585s
[^[[1;32mINFO ^[[0m] 2019-12-28 12:21:26 <ether-query>the time spent of all processBlocks 21m1.3153966s.
[^[[1;32mINFO ^[[0m] 2019-12-28 12:21:26 <ether-query>[100.00 precent]Processing block 1500001 finished, total elaspe=21m1.3154361s, lastest 1000 block elaspe=70.8µs
```



下面是在服务器上运行0-500W个块所需要的时间，batchBlockNum为50000，将500W个块一共分成了100份然后丢到16个协程的协程池中运行，运行结果如下：

```golang
[^[[1;32mINFO ^[[0m] 2019-12-28 16:33:11 <ether-query>the time spent of all processBlocks 17m39.844907368s. startBlock 1100000. endBlock: 1149999
[^[[1;32mINFO ^[[0m] 2019-12-28 16:34:46 <ether-query>the time spent of all processBlocks 18m40.895034229s. startBlock 1300000. endBlock: 1349999
[^[[1;32mINFO ^[[0m] 2019-12-28 16:35:5 <ether-query>the time spent of all processBlocks 18m52.978243646s. startBlock 1350000. endBlock: 1399999
[^[[1;32mINFO ^[[0m] 2019-12-28 16:35:7 <ether-query>the time spent of all processBlocks 19m10.495102548s. startBlock 1250000. endBlock: 1299999
[^[[1;32mINFO ^[[0m] 2019-12-28 16:35:39 <ether-query>the time spent of all processBlocks 20m1.163593074s. startBlock 1150000. endBlock: 1199999
[^[[1;32mINFO ^[[0m] 2019-12-28 16:37:2 <ether-query>the time spent of all processBlocks 21m10.1677007s. startBlock 1200000. endBlock: 1249999
[^[[1;32mINFO ^[[0m] 2019-12-28 16:39:24 <ether-query>the time spent of all processBlocks 22m0.496030923s. startBlock 1400000. endBlock: 1449999
[^[[1;32mINFO ^[[0m] 2019-12-28 16:43:33 <ether-query>the time spent of all processBlocks 24m1.844751902s. startBlock 1450000. endBlock: 1499999
[^[[1;32mINFO ^[[0m] 2019-12-28 16:52:35 <ether-query>the time spent of all processBlocks 27m50.866173765s. startBlock 1600000. endBlock: 1649999
[^[[1;32mINFO ^[[0m] 2019-12-28 16:52:38 <ether-query>the time spent of all processBlocks 28m56.846380841s. startBlock 1500000. endBlock: 1549999
[^[[1;32mINFO ^[[0m] 2019-12-28 16:54:0 <ether-query>the time spent of all processBlocks 29m57.328821266s. startBlock 1550000. endBlock: 1599999
[^[[1;32mINFO ^[[0m] 2019-12-28 16:56:42 <ether-query>the time spent of all processBlocks 26m58.393073374s. startBlock 1800000. endBlock: 1849999
[^[[1;32mINFO ^[[0m] 2019-12-28 16:56:49 <ether-query>the time spent of all processBlocks 28m31.958346062s. startBlock 1650000. endBlock: 1699999
[^[[1;32mINFO ^[[0m] 2019-12-28 16:57:44 <ether-query>the time spent of all processBlocks 26m48.994741596s. startBlock 1850000. endBlock: 1899999
[^[[1;32mINFO ^[[0m] 2019-12-28 17:1:52 <ether-query>the time spent of all processBlocks 32m15.740769365s. startBlock 1750000. endBlock: 1799999
[^[[1;32mINFO ^[[0m] 2019-12-28 17:4:1 <ether-query>the time spent of all processBlocks 28m53.77529577s. startBlock 2050000. endBlock: 2099999
[^[[1;32mINFO ^[[0m] 2019-12-28 17:4:2 <ether-query>the time spent of all processBlocks 28m57.189375669s. startBlock 2000000. endBlock: 2049999
[^[[1;32mINFO ^[[0m] 2019-12-28 17:4:45 <ether-query>the time spent of all processBlocks 31m33.68621197s. startBlock 1900000. endBlock: 1949999
[^[[1;32mINFO ^[[0m] 2019-12-28 17:7:9 <ether-query>the time spent of all processBlocks 32m23.039259976s. startBlock 1950000. endBlock: 1999999
[^[[1;32mINFO ^[[0m] 2019-12-28 17:7:11 <ether-query>the time spent of all processBlocks 31m31.845826881s. startBlock 2100000. endBlock: 2149999
[^[[1;32mINFO ^[[0m] 2019-12-28 17:7:40 <ether-query>the time spent of all processBlocks 30m37.660729544s. startBlock 2150000. endBlock: 2199999
[^[[1;32mINFO ^[[0m] 2019-12-28 17:9:19 <ether-query>the time spent of all processBlocks 40m23.975274627s. startBlock 1700000. endBlock: 1749999
[^[[1;32mINFO ^[[0m] 2019-12-28 17:9:53 <ether-query>the time spent of all processBlocks 30m28.154668565s. startBlock 2200000. endBlock: 2249999
[^[[1;32mINFO ^[[0m] 2019-12-28 17:17:29 <ether-query>the time spent of all processBlocks 33m55.991357548s. startBlock 2250000. endBlock: 2299999
[^[[1;32mINFO ^[[0m] 2019-12-28 17:28:44 <ether-query>the time spent of all processBlocks 31m0.539798776s. startBlock 2550000. endBlock: 2599999
[^[[1;32mINFO ^[[0m] 2019-12-28 17:30:3 <ether-query>the time spent of all processBlocks 33m13.90860743s. startBlock 2500000. endBlock: 2549999
[^[[1;32mINFO ^[[0m] 2019-12-28 17:32:9 <ether-query>the time spent of all processBlocks 27m23.960983964s. startBlock 2750000. endBlock: 2799999
[^[[1;32mINFO ^[[0m] 2019-12-28 17:33:25 <ether-query>the time spent of all processBlocks 31m32.777456336s. startBlock 2600000. endBlock: 2649999
[^[[1;32mINFO ^[[0m] 2019-12-28 17:35:9 <ether-query>the time spent of all processBlocks 27m58.237018338s. startBlock 2850000. endBlock: 2899999
[^[[1;32mINFO ^[[0m] 2019-12-28 17:37:2 <ether-query>the time spent of all processBlocks 29m52.583293682s. startBlock 2800000. endBlock: 2849999
[^[[1;32mINFO ^[[0m] 2019-12-28 17:37:45 <ether-query>the time spent of all processBlocks 30m5.376221735s. startBlock 2900000. endBlock: 2949999
[^[[1;32mINFO ^[[0m] 2019-12-28 17:39:34 <ether-query>the time spent of all processBlocks 30m15.525385673s. startBlock 2950000. endBlock: 2999999
[^[[1;32mINFO ^[[0m] 2019-12-28 17:41:19 <ether-query>the time spent of all processBlocks 31m26.300424654s. startBlock 3000000. endBlock: 3049999
[^[[1;32mINFO ^[[0m] 2019-12-28 17:49:42 <ether-query>the time spent of all processBlocks 32m12.684130295s. startBlock 3050000. endBlock: 3099999
[^[[1;32mINFO ^[[0m] 2019-12-28 18:0:59 <ether-query>the time spent of all processBlocks 32m14.79154853s. startBlock 3100000. endBlock: 3149999
[^[[1;32mINFO ^[[0m] 2019-12-28 18:5:56 <ether-query>the time spent of all processBlocks 35m52.909127494s. startBlock 3150000. endBlock: 3199999
[^[[1;32mINFO ^[[0m] 2019-12-28 18:6:59 <ether-query>the time spent of all processBlocks 34m50.024639492s. startBlock 3200000. endBlock: 3249999
[^[[1;32mINFO ^[[0m] 2019-12-28 18:12:48 <ether-query>the time spent of all processBlocks 39m22.930757335s. startBlock 3250000. endBlock: 3299999
[^[[1;32mINFO ^[[0m] 2019-12-28 18:18:5 <ether-query>the time spent of all processBlocks 42m56.104578819s. startBlock 3300000. endBlock: 3349999
[^[[1;32mINFO ^[[0m] 2019-12-28 18:32:37 <ether-query>the time spent of all processBlocks 55m35.332545365s. startBlock 3350000. endBlock: 3399999
[^[[1;32mINFO ^[[0m] 2019-12-28 18:34:8 <ether-query>the time spent of all processBlocks 56m23.056534463s. startBlock 3400000. endBlock: 3449999
[^[[1;32mINFO ^[[0m] 2019-12-28 18:35:44 <ether-query>the time spent of all processBlocks 54m25.533930264s. startBlock 3500000. endBlock: 3549999
[^[[1;32mINFO ^[[0m] 2019-12-28 18:39:5 <ether-query>the time spent of all processBlocks 59m31.047006576s. startBlock 3450000. endBlock: 3499999
[^[[1;32mINFO ^[[0m] 2019-12-28 18:52:21 <ether-query>the time spent of all processBlocks 1h2m38.523126657s. startBlock 3550000. endBlock: 3599999
[^[[1;32mINFO ^[[0m] 2019-12-28 19:17:4 <ether-query>the time spent of all processBlocks 1h16m4.508916646s. startBlock 3600000. endBlock: 3649999
[^[[1;32mINFO ^[[0m] 2019-12-28 19:30:33 <ether-query>the time spent of all processBlocks 1h24m36.561378898s. startBlock 3650000. endBlock: 3699999
[^[[1;32mINFO ^[[0m] 2019-12-28 20:3:16 <ether-query>the time spent of all processBlocks 1h56m17.566896705s. startBlock 3700000. endBlock: 3749999
[^[[1;32mINFO ^[[0m] 2019-12-28 20:43:33 <ether-query>the time spent of all processBlocks 2h30m45.292721417s. startBlock 3750000. endBlock: 3799999
[^[[1;32mINFO ^[[0m] 2019-12-28 21:12:7 <ether-query>the time spent of all processBlocks 2h54m2.261516243s. startBlock 3800000. endBlock: 3849999
[^[[1;32mINFO ^[[0m] 2019-12-28 22:14:40 <ether-query>the time spent of all processBlocks 3h42m2.204951585s. startBlock 3850000. endBlock: 3899999
[^[[1;32mINFO ^[[0m] 2019-12-28 22:45:31 <ether-query>the time spent of all processBlocks 5h41m29.698284467s. startBlock 2650000. endBlock: 2699999
[^[[1;32mINFO ^[[0m] 2019-12-28 22:50:7 <ether-query>the time spent of all processBlocks 4h15m59.371082516s. startBlock 3900000. endBlock: 3949999
[^[[1;32mINFO ^[[0m] 2019-12-28 23:1:18 <ether-query>the time spent of all processBlocks 4h25m33.567506116s. startBlock 3950000. endBlock: 3999999
[^[[1;32mINFO ^[[0m] 2019-12-28 23:15:7 <ether-query>the time spent of all processBlocks 4h22m45.91636253s. startBlock 4050000. endBlock: 4099999
[^[[1;32mINFO ^[[0m] 2019-12-28 23:25:59 <ether-query>the time spent of all processBlocks 6h29m16.181898489s. startBlock 2450000. endBlock: 2499999
[^[[1;32mINFO ^[[0m] 2019-12-28 23:36:14 <ether-query>the time spent of all processBlocks 4h57m8.578754486s. startBlock 4000000. endBlock: 4049999
[^[[1;32mINFO ^[[0m] 2019-12-28 23:44:29 <ether-query>the time spent of all processBlocks 6h51m53.170169699s. startBlock 2300000. endBlock: 2349999
[^[[1;32mINFO ^[[0m] 2019-12-29 1:4:23 <ether-query>the time spent of all processBlocks 8h0m21.326823147s. startBlock 2700000. endBlock: 2749999
[^[[1;32mINFO ^[[0m] 2019-12-29 1:17:58 <ether-query>the time spent of all processBlocks 6h0m54.136068723s. startBlock 4100000. endBlock: 4149999
[^[[1;32mINFO ^[[0m] 2019-12-29 3:10:9 <ether-query>the time spent of all processBlocks 7h39m36.635267922s. startBlock 4150000. endBlock: 4199999
[^[[1;32mINFO ^[[0m] 2019-12-29 4:59:8 <ether-query>the time spent of all processBlocks 8h55m51.558678367s. startBlock 4200000. endBlock: 4249999
[^[[1;32mINFO ^[[0m] 2019-12-29 5:6:49 <ether-query>the time spent of all processBlocks 6h21m18.750221112s. startBlock 4400000. endBlock: 4449999
[^[[1;32mINFO ^[[0m] 2019-12-29 5:53:35 <ether-query>the time spent of all processBlocks 9h10m2.082220984s. startBlock 4250000. endBlock: 4299999
[^[[1;32mINFO ^[[0m] 2019-12-29 5:54:56 <ether-query>the time spent of all processBlocks 6h53m37.684756692s. startBlock 4500000. endBlock: 4549999
[^[[1;32mINFO ^[[0m] 2019-12-29 6:5:14 <ether-query>the time spent of all processBlocks 7h15m6.9648574s. startBlock 4450000. endBlock: 4499999
[^[[1;32mINFO ^[[0m] 2019-12-29 6:19:55 <ether-query>the time spent of all processBlocks 8h5m15.204863149s. startBlock 4350000. endBlock: 4399999
[^[[1;32mINFO ^[[0m] 2019-12-29 6:52:2 <ether-query>the time spent of all processBlocks 7h36m55.352961238s. startBlock 4550000. endBlock: 4599999
[^[[1;32mINFO ^[[0m] 2019-12-29 6:53:4 <ether-query>the time spent of all processBlocks 9h40m56.315233652s. startBlock 4300000. endBlock: 4349999
[^[[1;32mINFO ^[[0m] 2019-12-29 7:37:0 <ether-query>the time spent of all processBlocks 14h42m59.664844128s. startBlock 2400000. endBlock: 2449999
[^[[1;32mINFO ^[[0m] 2019-12-29 7:42:27 <ether-query>the time spent of all processBlocks 14h49m48.861297854s. startBlock 2350000. endBlock: 2399999
[^[[1;32mINFO ^[[0m] 2019-12-29 7:47:29 <ether-query>the time spent of all processBlocks 8h21m30.57061173s. startBlock 4600000. endBlock: 4649999
[^[[1;32mINFO ^[[0m] 2019-12-29 10:6:28 <ether-query>the time spent of all processBlocks 10h30m14.643420266s. startBlock 4650000. endBlock: 4699999
[^[[1;32mINFO ^[[0m] 2019-12-29 12:51:10 <ether-query>the time spent of all processBlocks 13h6m41.831405616s. startBlock 4700000. endBlock: 4749999
[^[[1;32mINFO ^[[0m] 2019-12-29 13:31:39 <ether-query>the time spent of all processBlocks 12h27m15.468728874s. startBlock 4750000. endBlock: 4799999
[^[[1;32mINFO ^[[0m] 2019-12-29 14:23:47 <ether-query>the time spent of all processBlocks 13h5m48.994441747s. startBlock 4800000. endBlock: 4849999
[^[[1;32mINFO ^[[0m] 2019-12-29 16:19:43 <ether-query>the time spent of all processBlocks 11h12m53.816343208s. startBlock 4950000. endBlock: 5000000
[^[[1;32mINFO ^[[0m] 2019-12-29 17:20:59 <ether-query>the time spent of all processBlocks 14h10m49.236902746s. startBlock 4850000. endBlock: 4899999
[^[[1;32mINFO ^[[0m] 2019-12-29 17:59:19 <ether-query>the time spent of all processBlocks 13h0m10.683565187s. startBlock 4900000. endBlock: 4949999
```

发现当batchBlockNum为50000的时候，后面的块由于每5W个块执行的时间过长，导致最后16个协程结束时间相差很大，因此得出结论以每5W的块来分配计算量还是不能充分利用计算资源。



670W-680W

```bash

KiB Mem : 32586856 total,   392584 free, 29956712 used,  2237560 buff/cache
KiB Swap: 53687091+total, 48100377+free, 55867140 used.  1946912 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 5369 czm       20   0  0.098t 0.027t   1036 D 146.4 88.8   1376:38 main
  133 root      20   0       0      0      0 R  42.2  0.0   1849:21 kswapd0
20095 even      20   0 4429180  28288     92 S   7.5  0.1   5:08.74 gnome-shell
 4078 root      20   0   67844   9028   2100 S   4.6  0.0  12:54.79 iotop
19937 even      20   0  721728   3396   2700 S   3.6  0.0   1:19.98 Xorg
  113 root      20   0       0      0      0 S   2.9  0.0 100:50.93 kcompactd0

```

如果cpu的逻辑核数设置成8，然后协程池的个数限制成8，磁盘io状况

```bash
Total DISK READ :     496.22 M/s | Total DISK WRITE :       5.91 M/s
Actual DISK READ:     497.54 M/s | Actual DISK WRITE:      81.81 K/s
```

678W-686W

```bash

KiB Mem : 32586856 total,   390224 free, 29577420 used,  2619212 buff/cache
KiB Swap: 53687091+total, 46930304+free, 67567872 used.  2594540 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 8299 czm       20   0 92.804g 0.027t   4368 S 141.1 87.7   3:13.07 main
  133 root      20   0       0      0      0 S  29.9  0.0   1852:38 kswapd0

```



678W-682W

```bash
KiB Mem : 32586856 total,   417196 free, 29667208 used,  2502452 buff/cache
KiB Swap: 53687091+total, 53531059+free,  1560324 used.  2384660 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 8570 czm       20   0 30.086g 0.027t   6776 S  89.8 88.7   1:29.89 main
  133 root      20   0       0      0      0 S   8.6  0.0   1852:45 kswapd0
```

682W-686W

```bash
KiB Mem : 32586856 total,   274868 free, 30681772 used,  1630216 buff/cache
KiB Swap: 53687091+total, 51654630+free, 20324612 used.  1386608 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 8604 czm       20   0 49.094g 0.028t   3940 S  96.1 91.6   1:20.41 main
  133 root      20   0       0      0      0 S  19.3  0.0   1853:05 kswapd0

```





680W-682W

```bash

KiB Mem : 32586856 total,   253900 free, 27965504 used,  4367452 buff/cache
KiB Swap: 53687091+total, 53564160+free,  1229316 used.  4305436 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 8903 czm       20   0 28.005g 0.025t  19216 S  63.7 83.7   0:57.87 main
 8332 root      20   0   67416  11076   4236 S   5.3  0.0   0:26.55 iotop
  133 root      20   0       0      0      0 S   4.3  0.0   1853:13 kswapd0

```

680W-681W

```bash

KiB Swap: 53687091+total, 53564441+free,  1226500 used.  5304780 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 9072 czm       20   0 27.493g 0.025t  18432 S  79.1 81.7   0:33.99 main
  133 root      20   0       0      0      0 S   4.0  0.0   1853:16 kswapd0
 8332 root      20   0   67416  11164   4300 S   3.6  0.0   0:34.03 iotop
 1562 gdm       20   0  693184  20892   4100 S   1.0  0.1 235:11.11 gsd-color

```

问题总结：

上述问题是由于captureState函数中新建了一些对象，然后这些对象的指针被保存在全局变量的结构体中，导致go gc工具在释放内存时由于captureState中的对象一直有全局对象指针指向其值，导致其内存一直得不到释放。stack和memory都是一个1024字节的数组，因此其内存一直就是成M的增加，当碰到6810086中的`0x68b71d202dc52ad80812b563f3f6b0aaf1f19c04c1260d13055daad5b88a36a8`交易时，其trace数量为100W，导致其内存分配巨大而一直得不到释放。只需将保存stack和memory的部分注释掉即可，因为该部分只要debug打开，该部分会默认加入到structlog的全局变量中。