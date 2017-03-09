# NPL Runtime Performance Compare

## High-Performance JIT Compiler
NPL syntax is 100% compatible with Lua, therefore it can be configured to utilize JIT compiler (Luajit). 
In general, luajit is believed to be one of the fastest JIT compiler in the world. It's speed is close to C/C++, and can even out-perform static typed languages like java/C#. It is the fastest dynamic language in the world.
However, special care needs to be taken when writing test cases, since badly written test case can make the same code 100 times slower. 

### Compare Chart
Following is from [Julia](http://julialang.org/). Source code: 
 [C](https://github.com/JuliaLang/julia/blob/master/test/perf/micro/perf.c), [Fortran](https://github.com/JuliaLang/julia/blob/master/test/perf/micro/perf.f90), [Python](https://github.com/JuliaLang/julia/blob/master/test/perf/micro/perf.py), [Matlab/Octave](https://github.com/JuliaLang/julia/blob/master/test/perf/micro/perf.m), [R](https://github.com/JuliaLang/julia/blob/master/test/perf/micro/perf.R), [JavaScript](https://github.com/JuliaLang/julia/tree/master/test/perf/micro/java/src/main/java), [Java](https://github.com/JuliaLang/julia/tree/master/test/perf/micro/java/src/main/java), [Go](https://github.com/JuliaLang/julia/blob/master/test/perf/micro/perf.go), [Lua](https://github.com/JuliaLang/julia/blob/master/test/perf/micro/perf.lua).

The following micro-benchmark results were obtained on a single core (serial execution) on an Intel(R) Xeon(R) CPU E7-8850 2.00GHz CPU with 1TB of 1067MHz DDR3 RAM, running Linux:

|            |C/Fortran |Python |**NPL** |Java|JavaScript|Go	|R	|Matlab |Octave	 
|---------------|-------|-------|--------|----|------|------|-------|-------|--------
|               |gcc5.1 |3.4.3	| 1.0    |1.8 |V8    |go1.5	|3.2.2	|R2015b	|4.0.0	 
|mandel	        |0.81	|15.32	|**0.67**|1.35|0.66	|1.11	|53.16	|7.58	|451.81	 
|fib	        |0.70	|77.76	|**1.71**|1.21|3.36	|1.86	|533.52 |26.89	|9324.35 
|rand_mat_mul	|3.48	|1.14	|**1.16**|2.36|15.07|1.42	|1.57	|1.12	|1.12	 
|rand_mat_stat	|1.45	|17.93	|**3.27**|3.92|2.30	|2.96	|14.56	|14.52	|30.93	 
|parse_int      |5.05	|17.02	|**5.77**|3.35|6.06	|1.20	|45.73	|802.52	|9581.44 
|quicksort      |1.31	|32.89	|**2.03**|2.60|2.70	|1.29	|264.54 |4.92	|1866.01 
|pi_sum	        |1.00	|21.99	|**1.00**|1.00|1.01	|1.00	|9.56	|1.00	|299.31	 

_Figure: benchmark times relative to C (smaller is better, C performance = 1.0)._


> C and Fortran compiled by gcc 5.1.1, taking best timing from all optimization levels (-O0 through -O3). C, Fortran, Go. Python 3 was installed from the Anaconda distribution. The Python implementations of rand_mat_stat and rand_mat_mul use NumPy (v1.9.2) functions; the rest are pure Python implementations.
Benchmarks can also be seen here as a plot created with Gadfly.

These benchmarks, while not comprehensive, do test compiler performance on a range of common code patterns, such as function calls, string parsing, sorting, numerical loops, random number generation, and array operations. It is important to note that these benchmark implementations are not written for absolute maximal performance (the fastest code to compute fib(20) is the constant literal 6765). Rather, all of the benchmarks are written to test the performance of specific algorithms implemented in each language. In particular, all languages use the same algorithm: the Fibonacci benchmarks are all recursive while the pi summation benchmarks are all iterative; the “algorithm” for random matrix multiplication is to call LAPACK, except where that’s not possible, such as in JavaScript. The point of these benchmarks is to compare the performance of specific algorithms across language implementations, not to compare the fastest means of computing a result, which in most high-level languages relies on calling C code. 


## NPL Database Performance
More information, please see [[UsingTableDatabase]]

Following is tested on my Intel-i7-3GHZ CPU. See [test folder](https://github.com/NPLPackages/main/blob/master/script/ide/System/Database/test/test_TableDatabase.lua) for test source code.

### Run With Conservative Mode
Following is averaged value from 100000+ operations in a single thread

* Random non-index insert: `43478 inserts/second`
   * Async API tested with default configuration with 1 million records on a single thread.
* Round trip latency call: 
   * Blocking API: `20000 query/s`
   * Non-blocking API: `11ms` or `85 query/s` (due to NPL time slice)
   * i.e. Round strip means start next operation after previous one is returned. This is latency test.
* Random indexed inserts: `17953 query/s`
   * i.e. start next operation immediately, without waiting for the first one to return.
* Random select with auto-index: `18761 query/s`
   * i.e. same as above, but with findOne operation. 
* Randomly mixing CRUD operations: `965-7518` query/s
   * i.e. same as above, but randomly calling Create/Read/Update/Delete (CRUD) on the same auto-indexed table.
   * Mixing read/write can be slow when database grows bigger. e.g. you can get `18000 CRUD/s` for just 10000 records. 

## NPL Web Server Performance

The following is done using ApacheBench (AB tool) with 5000 requests and 1000 concurrency on a 1 CPU 1GB memory virtual machine. The queried content is [[http://paracraft.wiki/]], which is a fairly standard dynamic web page. 

```
root@iZ62yqw316fZ:/opt/paracraftwiki# ab -n 5000 -c 1000 http://paracraft.wiki/
This is ApacheBench, Version 2.3 <$Revision: 655654 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking paracraft.wiki (be patient).....done

Finished 5000 requests

Server Software:        NPL/1.1
Server Hostname:        paracraft.wiki
Server Port:            80

Document Path:          /
Document Length:        24736 bytes

Concurrency Level:      1000
Time taken for tests:   3.005 seconds
Complete requests:      5000
Failed requests:        0
Write errors:           0
Total transferred:      124730000 bytes
HTML transferred:       123680000 bytes
Requests per second:    1664.05 [#/sec] (mean)
Time per request:       600.944 [ms] (mean)
Time per request:       0.601 [ms] (mean, across all concurrent requests)
Transfer rate:          40538.43 [Kbytes/sec] received
```
## References
There are a few benchmark compare sites: 
- [[https://github.com/attractivechaos/plb]]: [[algorithm compare |http://attractivechaos.github.io/plb/]]
- [TechEmpower](https://github.com/TechEmpower/FrameworkBenchmarks): compare web server framework only
- [Luajit Performance](http://luajit.org/performance.html)
- [Computer benchmark game](http://benchmarksgame.alioth.debian.org/): No luajit.