# aw04报告

## 1. 构造webpos的docker镜像，运行负载测试

```webpos```使用了redis作为会话缓存，并将从京东获取的product信息缓存到redis中。
通过本地构造redis集群，将```webpos```镜像在docker容器上运行后，使用gatling脚本进行测试，结果如下：

```shell
使用cpus=0.5 的参数进行负载测试
================================================================================
---- Global Information --------------------------------------------------------
> request count                                        100 (OK=100    KO=0     )
> min response time                                   6905 (OK=6905   KO=-     )
> max response time                                   9025 (OK=9025   KO=-     )
> mean response time                                  8321 (OK=8321   KO=-     )
> std deviation                                        532 (OK=532    KO=-     )
> response time 50th percentile                       8451 (OK=8451   KO=-     )
> response time 75th percentile                       8560 (OK=8560   KO=-     )
> response time 95th percentile                       8924 (OK=8924   KO=-     )
> response time 99th percentile                       8948 (OK=8948   KO=-     )
> mean requests/sec                                     10 (OK=10     KO=-     )
---- Response Time Distribution ------------------------------------------------
> t < 800 ms                                             0 (  0%)
> 800 ms < t < 1200 ms                                   0 (  0%)
> t > 1200 ms                                          100 (100%)
> failed                                                 0 (  0%)
================================================================================
```
在不限制cpu使用率的情况下，响应时间比较慢。
## 2. 使用haproxy进行webpos的水平拓展，并进行负载测试
```shell
使用cpus=0.5 的3个容器进行水平拓展
================================================================================
---- Global Information --------------------------------------------------------
> request count                                        100 (OK=100    KO=0     )
> min response time                                    721 (OK=721    KO=-     )
> max response time                                   2593 (OK=2593   KO=-     )
> mean response time                                  2049 (OK=2049   KO=-     )
> std deviation                                        435 (OK=435    KO=-     )
> response time 50th percentile                       2191 (OK=2191   KO=-     )
> response time 75th percentile                       2383 (OK=2383   KO=-     )
> response time 95th percentile                       2511 (OK=2511   KO=-     )
> response time 99th percentile                       2582 (OK=2582   KO=-     )
> mean requests/sec                                 33.333 (OK=33.333 KO=-     )
---- Response Time Distribution ------------------------------------------------
> t < 800 ms                                             1 (  1%)
> 800 ms < t < 1200 ms                                   6 (  6%)
> t > 1200 ms                                           93 ( 93%)
> failed                                                 0 (  0%)
================================================================================
```
在进行了水平拓展后，响应时间相应下降了，性能有所提升。
## 3. 使用redis集群作为webpos缓存，并进行负载测试

```shell
* 限制每个容器cpus=0.1.

首次进行100个用户的并发访问
================================================================================
---- Global Information --------------------------------------------------------
> request count                                        100 (OK=100    KO=0     )
> min response time                                   2441 (OK=2441   KO=-     )
> max response time                                  10719 (OK=10719  KO=-     )
> mean response time                                  7128 (OK=7128   KO=-     )
> std deviation                                       1800 (OK=1800   KO=-     )
> response time 50th percentile                       7123 (OK=7123   KO=-     )
> response time 75th percentile                       8726 (OK=8726   KO=-     )
> response time 95th percentile                       9631 (OK=9631   KO=-     )
> response time 99th percentile                      10407 (OK=10407  KO=-     )
> mean requests/sec                                  9.091 (OK=9.091  KO=-     )
---- Response Time Distribution ------------------------------------------------
> t < 800 ms                                             0 (  0%)
> 800 ms < t < 1200 ms                                   0 (  0%)
> t > 1200 ms                                          100 (100%)
> failed                                                 0 (  0%)
================================================================================

多次进行100个用户的并发访问
================================================================================
---- Global Information --------------------------------------------------------
> request count                                        100 (OK=100    KO=0     )
> min response time                                   2177 (OK=2177   KO=-     )
> max response time                                   7872 (OK=7872   KO=-     )
> mean response time                                  5346 (OK=5346   KO=-     )
> std deviation                                       1161 (OK=1161   KO=-     )
> response time 50th percentile                       5367 (OK=5367   KO=-     )
> response time 75th percentile                       6104 (OK=6104   KO=-     )
> response time 95th percentile                       7077 (OK=7077   KO=-     )
> response time 99th percentile                       7755 (OK=7755   KO=-     )
> mean requests/sec                                   12.5 (OK=12.5   KO=-     )
---- Response Time Distribution ------------------------------------------------
> t < 800 ms                                             0 (  0%)
> 800 ms < t < 1200 ms                                   0 (  0%)
> t > 1200 ms                                          100 (100%)
> failed                                                 0 (  0%)
================================================================================
```
分析可见，在采用了cache技术和会话缓存技术之后，在初次获取product数据后，webpos已经能够从缓存中找到对应的
数据，不需要再次进行获取，因而得到的访问时间减少。
并且，在相同用户多次访问时，webpos总是能从redis缓存中找到cart，避免了数据缺失。