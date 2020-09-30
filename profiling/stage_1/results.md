# Анализ результатов

## VisualVM
Вкладка Monitor:
![Screen from visual vm monitor](visual-vm-monitor.jpeg?raw=true "Screen from visual vm monitor")
Верхний правый график - график использования Heap-а. Тут можно увидеть как мы дважды нагрузили сервис put-запросами 
(я их обвела крассным) и дважды get-запросами (эти в зеленой области).
Результаты wrk и async profiler-а можно увидеть ниже.

Вкладка Threads:
![Screen from visual vm threads](visual-vm-threads.png?raw=true "Screen from visual vm threads")
Отсюда видим, что у нас есть 8 NIO Selector-ов. Вероятно, по количеству логических процессоров на моем ноуте. 
И поток для NIO Acceptor-а.

## STATUS
Статус просто возвращает Response 200.

Запускаю wrk c такими параметрами:
+ 1 thread (worker) send requests
+ keeping 1 HTTP connections open
+ 30 seconds
+ constant throughput of 2000 requests per second

Результаты с async profiler-а (CPU):
![Flame graph from async profiler](status-flamegraph-cpu.svg?raw=true "Flame graph from async profiler")

Результаты с async profiler-а (ALLOC):
![Flame graph from async profiler](status-flamegraph-alloc.svg?raw=true "Flame graph from async profiler")

На обоих графах видим: 
* поток Selector-a, который обрабатывает наши запросы (98% на CPU, 88% на alloc-е)
* RMI (Remote Method Invocation), который работает с pool-ом thread-ов.

Результаты wrk2:
```
Running 30s test @ http://127.0.0.1:8080/v0/status
     1 threads and 1 connections
     Thread calibration: mean lat.: 1.224ms, rate sampling interval: 10ms
     Thread Stats   Avg      Stdev     Max   +/- Stdev
       Latency     1.21ms  596.47us   4.03ms   61.92%
       Req/Sec     2.11k   188.72     2.67k    62.09%
     Latency Distribution (HdrHistogram - Recorded Latency)
    50.000%    1.21ms
    75.000%    1.65ms
    90.000%    2.05ms
    99.000%    2.40ms
    99.900%    2.57ms
    99.990%    3.27ms
    99.999%    4.03ms
   100.000%    4.03ms
   
     Detailed Percentile spectrum:
          Value   Percentile   TotalCount 1/(1-Percentile)
          0.028     0.000000            1         1.00
          0.373     0.100000         4004         1.11
          0.631     0.200000         7999         1.25
          0.871     0.300000        12018         1.43
          1.062     0.400000        16003         1.67
          1.205     0.500000        20026         2.00
          1.276     0.550000        22014         2.22
          1.348     0.600000        24010         2.50
          1.424     0.650000        25996         2.86
          1.522     0.700000        27996         3.33
          1.651     0.750000        30001         4.00
          1.714     0.775000        31011         4.44
          1.778     0.800000        31995         5.00
          1.842     0.825000        33003         5.71
          1.912     0.850000        33994         6.67
          1.983     0.875000        35010         8.00
          2.018     0.887500        35499         8.89
          2.053     0.900000        36020        10.00
          2.089     0.912500        36493        11.43
          2.127     0.925000        36996        13.33
          2.167     0.937500        37497        16.00
          2.189     0.943750        37756        17.78
          2.213     0.950000        37998        20.00
          2.237     0.956250        38251        22.86
          2.263     0.962500        38512        26.67
          2.291     0.968750        38754        32.00
          2.305     0.971875        38871        35.56
          2.321     0.975000        38994        40.00
          2.335     0.978125        39127        45.71
          2.353     0.981250        39251        53.33
          2.369     0.984375        39373        64.00
          2.377     0.985938        39436        71.11
          2.387     0.987500        39502        80.00
          2.397     0.989062        39556        91.43
          2.407     0.990625        39619       106.67
          2.421     0.992188        39681       128.00
          2.427     0.992969        39715       142.22
          2.435     0.993750        39754       160.00
          2.441     0.994531        39774       182.86
          2.451     0.995313        39806       213.33
          2.465     0.996094        39839       256.00
          2.473     0.996484        39852       284.44
          2.481     0.996875        39869       320.00
          2.489     0.997266        39884       365.71
          2.501     0.997656        39901       426.67
          2.511     0.998047        39916       512.00
          2.521     0.998242        39922       568.89
          2.531     0.998437        39932       640.00
          2.535     0.998633        39938       731.43
          2.553     0.998828        39946       853.33
          2.569     0.999023        39953      1024.00
          2.579     0.999121        39958      1137.78
          2.585     0.999219        39961      1280.00
          2.603     0.999316        39965      1462.86
          2.639     0.999414        39969      1706.67
          2.669     0.999512        39973      2048.00
          2.673     0.999561        39975      2275.56
          2.677     0.999609        39977      2560.00
          2.691     0.999658        39979      2925.71
          2.745     0.999707        39981      3413.33
          2.765     0.999756        39983      4096.00
          2.907     0.999780        39984      4551.11
          2.939     0.999805        39985      5120.00
          3.017     0.999829        39986      5851.43
          3.113     0.999854        39987      6826.67
          3.267     0.999878        39988      8192.00
          3.267     0.999890        39988      9102.22
          3.417     0.999902        39989     10240.00
          3.417     0.999915        39989     11702.86
          3.499     0.999927        39990     13653.33
          3.499     0.999939        39990     16384.00
          3.499     0.999945        39990     18204.44
          3.893     0.999951        39991     20480.00
          3.893     0.999957        39991     23405.71
          3.893     0.999963        39991     27306.67
          3.893     0.999969        39991     32768.00
          3.893     0.999973        39991     36408.89
          4.035     0.999976        39992     40960.00
          4.035     1.000000        39992          inf
   #[Mean    =        1.210, StdDeviation   =        0.596]
   #[Max     =        4.034, Total count    =        39992]
   #[Buckets =           27, SubBuckets     =         2048]
   ----------------------------------------------------------
     59999 requests in 30.00s, 6.01MB read
   Requests/sec:   1999.91
   Transfer/sec:    205.07KB
```
Итоги:
* обработали всего 59999 реквестов
* прочитали 6.01MB каких-то данных (заголовки и прочее)
* За 0.028 миллисекунд обработали 1ый запрос.
* 4.035 миллисекунд обработали 39992 запросов.
* сервер обрабатывал 1999.91 запросов в секунду
* 50% обрабатывается за 1.21ms
* 100.000% за 4.03ms-1.21ms=2.82ms (скорее всего время потратилось на освобождение хипа)
   
## PUT
### В 1 соединение
Кладем значения в хранилище. Формируем пакеты с помощью скриптов lua (находятся в /wrk/put.lua)
Запускаю wrk c такими параметрами:
+ 1 threads (worker) send requests
+ keeping 1 HTTP connections open
+ 30 seconds
+ constant throughput of 10000 requests per second

Результаты с async profiler-а (CPU):
![Flame graph from async profiler](1-put-flamegraph-cpu.svg?raw=true "Flame graph from async profiler")
Практически 99 семплов принадлежат NIO Selector-у, который обрабатывает наши запросы. Видно, что башенка селектора 
заканчивается  транзакциями? на Rocks DB.

Результаты с async profiler-а (ALLOC):
![Flame graph from async profiler](1-put-flamegraph-alloc.svg?raw=true "Flame graph from async profiler")
Практически та же картинка, что и на CPU, только башенки заканчиваются выделением памяти для примитивов.

Результаты wrk2:
```
Running 30s test @ http://127.0.0.1:8080
  1 threads and 1 connections
  Thread calibration: mean lat.: 65.697ms, rate sampling interval: 438ms
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   780.96ms  261.23ms   1.27s    61.06%
    Req/Sec     9.54k   621.64    10.90k    66.67%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%  902.14ms
 75.000%  940.54ms
 90.000%    1.07s 
 99.000%    1.25s 
 99.900%    1.27s 
 99.990%    1.27s 
 99.999%    1.27s 
100.000%    1.27s 

  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)
     297.471     0.000000           13         1.00
     351.487     0.100000        19064         1.11
     495.103     0.200000        38273         1.25
     564.223     0.300000        57465         1.43
     837.631     0.400000        76143         1.67
     902.143     0.500000        95609         2.00
     909.823     0.550000       105152         2.22
     916.479     0.600000       114659         2.50
     920.063     0.650000       123832         2.86
     926.207     0.700000       133901         3.33
     940.543     0.750000       142864         4.00
     957.439     0.775000       147469         4.44
     973.311     0.800000       152474         5.00
     988.671     0.825000       156991         5.71
    1004.543     0.850000       162068         6.67
    1034.239     0.875000       166502         8.00
    1050.623     0.887500       169367         8.89
    1065.983     0.900000       171363        10.00
    1106.943     0.912500       173747        11.43
    1123.327     0.925000       176029        13.33
    1137.663     0.937500       178612        16.00
    1139.711     0.943750       179811        17.78
    1142.783     0.950000       180898        20.00
    1148.927     0.956250       182121        22.86
    1162.239     0.962500       183150        26.67
    1191.935     0.968750       184353        32.00
    1196.031     0.971875       185313        35.56
    1197.055     0.975000       185581        40.00
    1203.199     0.978125       186141        45.71
    1214.463     0.981250       186751        53.33
    1227.775     0.984375       187318        64.00
    1234.943     0.985938       187631        71.11
    1241.087     0.987500       187908        80.00
    1247.231     0.989062       188201        91.43
    1255.423     0.990625       188502       106.67
    1264.639     0.992188       188997       128.00
    1264.639     0.992969       188997       142.22
    1265.663     0.993750       189528       160.00
    1265.663     0.994531       189528       182.86
    1265.663     0.995313       189528       213.33
    1266.687     0.996094       189833       256.00
    1266.687     0.996484       189833       284.44
    1266.687     0.996875       189833       320.00
    1266.687     0.997266       189833       365.71
    1266.687     0.997656       189833       426.67
    1267.711     0.998047       190195       512.00
    1267.711     0.998242       190195       568.89
    1267.711     0.998437       190195       640.00
    1267.711     0.998633       190195       731.43
    1267.711     0.998828       190195       853.33
    1267.711     0.999023       190195      1024.00
    1267.711     0.999121       190195      1137.78
    1267.711     0.999219       190195      1280.00
    1267.711     0.999316       190195      1462.86
    1267.711     0.999414       190195      1706.67
    1267.711     0.999512       190195      2048.00
    1267.711     0.999561       190195      2275.56
    1268.735     0.999609       190276      2560.00
    1268.735     1.000000       190276          inf
#[Mean    =      780.961, StdDeviation   =      261.230]
#[Max     =     1267.712, Total count    =       190276]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  287316 requests in 30.00s, 18.36MB read
Requests/sec:   9577.29
Transfer/sec:    626.64KB
```
Итоги:
* обработали всего 287316 реквестов
* прочитали 18.36MB каких-то данных (заголовки и прочее)
* за 297.471 миллисекунд обработали 1ый запрос.
* сервер обрабатывал 9577.29 запросов в секунду
* за 1.27 секунд обработали 190276 запросов.

### Несколько соединений
Хоть и просили проводить нагрузочное тестирование только в одно соединение, на 4-ех вроде все работает, потому что
Rocks DB потокобезопасная сама по себе :).
 
Запускаю wrk c такими параметрами:
+ 4 threads (worker) send requests
+ keeping 4 HTTP connections open
+ 60 seconds
+ constant throughput of 10000 requests per second

Результаты с async profiler-а (CPU):
![Flame graph from async profiler](put-flamegraph-cpu.svg?raw=true "Flame graph from async profiler")
Видим, что теперь запросы обрабатываются на 4-ех потоках.

Результаты с async profiler-а (ALLOC):
![Flame graph from async profiler](put-flamegraph-alloc.svg?raw=true "Flame graph from async profiler")
По неизвестной мне причине (пока что) мы видим три селектора и 2 RMI-потока.

Результаты wrk2:
```
Running 1m test @ http://127.0.0.1:8080
  4 threads and 4 connections
  Thread calibration: mean lat.: 406.364ms, rate sampling interval: 2007ms
  Thread calibration: mean lat.: 406.709ms, rate sampling interval: 2014ms
  Thread calibration: mean lat.: 402.879ms, rate sampling interval: 1993ms
  Thread calibration: mean lat.: 407.414ms, rate sampling interval: 2003ms
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     8.42s     4.73s   16.15s    56.81%
    Req/Sec     1.74k   196.52     2.30k    72.16%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%    8.24s 
 75.000%   12.67s 
 90.000%   15.02s 
 99.000%   16.04s 
 99.900%   16.14s 
 99.990%   16.15s 
 99.999%   16.15s 
100.000%   16.15s 

  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

    1032.191     0.000000            8         1.00
    1669.119     0.100000        34977         1.11
    3397.631     0.200000        69812         1.25
    5230.591     0.300000       104706         1.43
    6733.823     0.400000       139597         1.67
    8237.055     0.500000       174516         2.00
    9215.999     0.550000       191954         2.22
   10027.007     0.600000       209437         2.50
   10977.279     0.650000       226951         2.86
   12025.855     0.700000       244299         3.33
   12673.023     0.750000       261834         4.00
   12984.319     0.775000       270751         4.44
   13443.071     0.800000       279193         5.00
   13860.863     0.825000       288029         5.71
   14311.423     0.850000       296841         6.67
   14655.487     0.875000       305330         8.00
   14876.671     0.887500       309810         8.89
   15024.127     0.900000       314083        10.00
   15179.775     0.912500       318417        11.43
   15458.303     0.925000       323004        13.33
   15523.839     0.937500       327895        16.00
   15572.991     0.943750       329345        17.78
   15679.487     0.950000       331519        20.00
   15728.639     0.956250       333863        22.86
   15761.407     0.962500       336053        26.67
   15835.135     0.968750       338150        32.00
   15867.903     0.971875       339436        35.56
   15900.671     0.975000       340435        40.00
   15941.631     0.978125       341444        45.71
   15974.399     0.981250       342671        53.33
   15998.975     0.984375       343848        64.00
   16007.167     0.985938       344273        71.11
   16023.551     0.987500       345060        80.00
   16031.743     0.989062       345414        91.43
   16048.127     0.990625       345902       106.67
   16064.511     0.992188       346261       128.00
   16080.895     0.992969       346592       142.22
   16089.087     0.993750       346762       160.00
   16105.471     0.994531       347290       182.86
   16113.663     0.995313       347574       213.33
   16113.663     0.996094       347574       256.00
   16121.855     0.996484       347974       284.44
   16121.855     0.996875       347974       320.00
   16130.047     0.997266       348430       365.71
   16130.047     0.997656       348430       426.67
   16130.047     0.998047       348430       512.00
   16130.047     0.998242       348430       568.89
   16130.047     0.998437       348430       640.00
   16138.239     0.998633       348770       731.43
   16138.239     0.998828       348770       853.33
   16138.239     0.999023       348770      1024.00
   16138.239     0.999121       348770      1137.78
   16138.239     0.999219       348770      1280.00
   16138.239     0.999316       348770      1462.86
   16138.239     0.999414       348770      1706.67
   16138.239     0.999512       348770      2048.00
   16146.431     0.999561       348934      2275.56
   16146.431     0.999609       348934      2560.00
   16146.431     0.999658       348934      2925.71
   16146.431     0.999707       348934      3413.33
   16146.431     0.999756       348934      4096.00
   16146.431     0.999780       348934      4551.11
   16146.431     0.999805       348934      5120.00
   16146.431     0.999829       348934      5851.43
   16146.431     0.999854       348934      6826.67
   16146.431     0.999878       348934      8192.00
   16146.431     0.999890       348934      9102.22
   16146.431     0.999902       348934     10240.00
   16146.431     0.999915       348934     11702.86
   16146.431     0.999927       348934     13653.33
   16146.431     0.999939       348934     16384.00
   16146.431     0.999945       348934     18204.44
   16146.431     0.999951       348934     20480.00
   16146.431     0.999957       348934     23405.71
   16146.431     0.999963       348934     27306.67
   16146.431     0.999969       348934     32768.00
   16146.431     0.999973       348934     36408.89
   16146.431     0.999976       348934     40960.00
   16146.431     0.999979       348934     46811.43
   16146.431     0.999982       348934     54613.33
   16146.431     0.999985       348934     65536.00
   16146.431     0.999986       348934     72817.78
   16146.431     0.999988       348934     81920.00
   16146.431     0.999989       348934     93622.86
   16146.431     0.999991       348934    109226.67
   16146.431     0.999992       348934    131072.00
   16146.431     0.999993       348934    145635.56
   16146.431     0.999994       348934    163840.00
   16146.431     0.999995       348934    187245.71
   16146.431     0.999995       348934    218453.33
   16146.431     0.999996       348934    262144.00
   16146.431     0.999997       348934    291271.11
   16146.431     0.999997       348934    327680.00
   16154.623     0.999997       348935    374491.43
   16154.623     1.000000       348935          inf
#[Mean    =     8416.097, StdDeviation   =     4733.633]
#[Max     =    16146.432, Total count    =       348935]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  438624 requests in 1.00m, 28.03MB read
Requests/sec:   7310.51
Transfer/sec:    478.32KB
```
Итоги:
* обработали всего 438624 реквестов
* прочитали 28.03MB каких-то данных (заголовки и прочее)
* за  1032.191 миллисекунд обработали 1ый запрос.
* за 16.154623 секунд обработали 348935 запросов.
* сервер обрабатывал 7310.51 запросов в секунду (а мы нагружали 10000)


## GET
### В 1 соединение
Читаем из хранилища. Формируем пакеты с помощью скриптов lua (находятся в /wrk/get.lua)

Запускаю wrk c такими параметрами:
+ 1 thread (worker) send requests
+ keeping 1 HTTP connections open
+ 30 seconds
+ constant throughput of 10000 requests per second

Результаты с async profiler-а (CPU):
![Flame graph from async profiler](1-get-flamegraph-cpu.svg?raw=true "Flame graph from async profiler")
Картина особо не изменилась, просто мы видим, что вызываем get, а не put.

Результаты с async profiler-а (ALLOC):
![Flame graph from async profiler](1-get-flamegraph-alloc.svg?raw=true "Flame graph from async profiler")
То же самое.

Результаты wrk2:
```
Running 30s test @ http://127.0.0.1:8080
  1 threads and 1 connections
  Thread calibration: mean lat.: 0.809ms, rate sampling interval: 10ms
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   799.71us  431.16us   6.44ms   63.56%
    Req/Sec    10.57k     0.87k   17.33k    63.09%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%  813.00us
 75.000%    1.13ms
 90.000%    1.33ms
 99.000%    1.51ms
 99.900%    3.45ms
 99.990%    6.22ms
 99.999%    6.38ms
100.000%    6.45ms

  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)
       0.045     0.000000            2         1.00
       0.223     0.100000        20026         1.11
       0.379     0.200000        40069         1.25
       0.527     0.300000        60007         1.43
       0.672     0.400000        80062         1.67
       0.813     0.500000       100019         2.00
       0.882     0.550000       110105         2.22
       0.948     0.600000       120002         2.50
       1.013     0.650000       130093         2.86
       1.074     0.700000       140019         3.33
       1.134     0.750000       149961         4.00
       1.165     0.775000       154991         4.44
       1.196     0.800000       160097         5.00
       1.227     0.825000       165008         5.71
       1.260     0.850000       170034         6.67
       1.292     0.875000       174966         8.00
       1.309     0.887500       177555         8.89
       1.326     0.900000       180077        10.00
       1.342     0.912500       182454        11.43
       1.360     0.925000       185063        13.33
       1.376     0.937500       187442        16.00
       1.385     0.943750       188718        17.78
       1.394     0.950000       189970        20.00
       1.403     0.956250       191221        22.86
       1.412     0.962500       192470        26.67
       1.423     0.968750       193774        32.00
       1.428     0.971875       194364        35.56
       1.434     0.975000       194983        40.00
       1.441     0.978125       195567        45.71
       1.450     0.981250       196217        53.33
       1.462     0.984375       196820        64.00
       1.471     0.985938       197163        71.11
       1.482     0.987500       197456        80.00
       1.497     0.989062       197750        91.43
       1.520     0.990625       198071       106.67
       1.552     0.992188       198380       128.00
       1.576     0.992969       198534       142.22
       1.616     0.993750       198687       160.00
       1.676     0.994531       198843       182.86
       1.745     0.995313       199001       213.33
       1.846     0.996094       199155       256.00
       1.918     0.996484       199234       284.44
       2.035     0.996875       199312       320.00
       2.131     0.997266       199390       365.71
       2.257     0.997656       199468       426.67
       2.433     0.998047       199546       512.00
       2.517     0.998242       199585       568.89
       2.637     0.998437       199625       640.00
       2.839     0.998633       199663       731.43
       3.141     0.998828       199702       853.33
       3.493     0.999023       199741      1024.00
       3.825     0.999121       199761      1137.78
       4.013     0.999219       199780      1280.00
       4.487     0.999316       199800      1462.86
       5.011     0.999414       199819      1706.67
       5.435     0.999512       199839      2048.00
       5.559     0.999561       199849      2275.56
       5.663     0.999609       199858      2560.00
       5.827     0.999658       199868      2925.71
       5.943     0.999707       199878      3413.33
       6.015     0.999756       199888      4096.00
       6.043     0.999780       199893      4551.11
       6.063     0.999805       199897      5120.00
       6.119     0.999829       199902      5851.43
       6.131     0.999854       199907      6826.67
       6.175     0.999878       199912      8192.00
       6.203     0.999890       199915      9102.22
       6.235     0.999902       199917     10240.00
       6.243     0.999915       199919     11702.86
       6.283     0.999927       199924     13653.33
       6.283     0.999939       199924     16384.00
       6.315     0.999945       199926     18204.44
       6.323     0.999951       199927     20480.00
       6.327     0.999957       199928     23405.71
       6.335     0.999963       199929     27306.67
       6.359     0.999969       199930     32768.00
       6.371     0.999973       199931     36408.89
       6.375     0.999976       199933     40960.00
       6.375     0.999979       199933     46811.43
       6.375     0.999982       199933     54613.33
       6.375     0.999985       199933     65536.00
       6.379     0.999986       199934     72817.78
       6.379     0.999988       199934     81920.00
       6.379     0.999989       199934     93622.86
       6.411     0.999991       199935    109226.67
       6.411     0.999992       199935    131072.00
       6.411     0.999993       199935    145635.56
       6.411     0.999994       199935    163840.00
       6.411     0.999995       199935    187245.71
       6.447     0.999995       199936    218453.33
       6.447     1.000000       199936          inf
#[Mean    =        0.800, StdDeviation   =        0.431]
#[Max     =        6.444, Total count    =       199936]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  299975 requests in 30.00s, 19.74MB read
  Non-2xx or 3xx responses: 299975
Requests/sec:   9999.30
Transfer/sec:    673.78KB
```
Итоги:
* обработали всего 299975 реквестов
* прочитали19.74MB каких-то данных (заголовки и прочее)
* за 0.045 миллисекунд обработали 1ый запрос.
* за 6.447 миллисекунд  обработали 199936 запросов.
* сервер обрабатывал 9999.30 запросов в секунду 


### Несколько соединений
Запускаю wrk c такими параметрами:
+ 4 threads (worker) send requests
+ keeping 4 HTTP connections open
+ 30 seconds
+ constant throughput of 10000 requests per second

Результаты с async profiler-а (CPU):
![Flame graph from async profiler](get-flamegraph-cpu.svg?raw=true "Flame graph from async profiler")
В целом, картина та же самая, что и в PUT в несколько соединений, только стектрейс в Rocks DB чуть поменьше.

Результаты с async profiler-а (ALLOC):
![Flame graph from async profiler](get-flamegraph-alloc.svg?raw=true "Flame graph from async profiler")
А тут та же самая.

Результаты wrk2:
```
wrk2 -t4 -c4 -d30s -R10000 -s ./wrk/get.lua  --latency  http://127.0.0.1:8080
Running 30s test @ http://127.0.0.1:8080
  4 threads and 4 connections
  Thread calibration: mean lat.: 5.809ms, rate sampling interval: 31ms
  Thread calibration: mean lat.: 4.210ms, rate sampling interval: 19ms
  Thread calibration: mean lat.: 4.736ms, rate sampling interval: 25ms
  Thread calibration: mean lat.: 4.077ms, rate sampling interval: 22ms
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   778.55ms  907.76ms   2.84s    76.55%
    Req/Sec     2.22k   451.47     3.40k    67.64%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%  267.52ms
 75.000%    1.52s 
 90.000%    2.37s 
 99.000%    2.69s 
 99.900%    2.80s 
 99.990%    2.84s 
 99.999%    2.84s 
100.000%    2.84s 

  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

       0.066     0.000000            1         1.00
       2.013     0.100000        17347         1.11
      13.567     0.200000        34675         1.25
      77.823     0.300000        52011         1.43
     146.687     0.400000        69362         1.67
     267.519     0.500000        86679         2.00
     407.295     0.550000        95356         2.22
     554.495     0.600000       104050         2.50
     808.447     0.650000       112744         2.86
    1062.911     0.700000       121366         3.33
    1519.615     0.750000       130032         4.00
    1778.687     0.775000       134374         4.44
    1937.407     0.800000       138706         5.00
    2022.399     0.825000       143032         5.71
    2158.591     0.850000       147379         6.67
    2289.663     0.875000       151732         8.00
    2340.863     0.887500       153975         8.89
    2371.583     0.900000       156066        10.00
    2402.303     0.912500       158257        11.43
    2422.783     0.925000       160389        13.33
    2449.407     0.937500       162646        16.00
    2469.887     0.943750       163607        17.78
    2488.319     0.950000       164709        20.00
    2500.607     0.956250       165884        22.86
    2521.087     0.962500       166866        26.67
    2557.951     0.968750       167977        32.00
    2580.479     0.971875       168500        35.56
    2596.863     0.975000       169036        40.00
    2619.391     0.978125       169628        45.71
    2639.871     0.981250       170291        53.33
    2648.063     0.984375       170694        64.00
    2654.207     0.985938       171037        71.11
    2664.447     0.987500       171201        80.00
    2684.927     0.989062       171489        91.43
    2695.167     0.990625       171829       106.67
    2707.455     0.992188       172052       128.00
    2711.551     0.992969       172176       142.22
    2713.599     0.993750       172293       160.00
    2717.695     0.994531       172430       182.86
    2723.839     0.995313       172632       213.33
    2727.935     0.996094       172752       256.00
    2727.935     0.996484       172752       284.44
    2729.983     0.996875       172828       320.00
    2738.175     0.997266       172887       365.71
    2764.799     0.997656       172953       426.67
    2783.231     0.998047       173039       512.00
    2785.279     0.998242       173077       568.89
    2787.327     0.998437       173095       640.00
    2791.423     0.998633       173145       731.43
    2793.471     0.998828       173177       853.33
    2795.519     0.999023       173190      1024.00
    2799.615     0.999121       173206      1137.78
    2805.759     0.999219       173226      1280.00
    2811.903     0.999316       173246      1462.86
    2820.095     0.999414       173259      1706.67
    2826.239     0.999512       173274      2048.00
    2832.383     0.999561       173289      2275.56
    2834.431     0.999609       173303      2560.00
    2834.431     0.999658       173303      2925.71
    2836.479     0.999707       173316      3413.33
    2836.479     0.999756       173316      4096.00
    2838.527     0.999780       173339      4551.11
    2838.527     0.999805       173339      5120.00
    2838.527     0.999829       173339      5851.43
    2838.527     0.999854       173339      6826.67
    2838.527     0.999878       173339      8192.00
    2838.527     0.999890       173339      9102.22
    2840.575     0.999902       173347     10240.00
    2840.575     0.999915       173347     11702.86
    2840.575     0.999927       173347     13653.33
    2842.623     0.999939       173358     16384.00
    2842.623     1.000000       173358          inf
#[Mean    =      778.545, StdDeviation   =      907.762]
#[Max     =     2840.576, Total count    =       173358]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  273376 requests in 30.00s, 17.99MB read
  Non-2xx or 3xx responses: 273376
Requests/sec:   9112.82
Transfer/sec:    614.05KB
```

Итоги:
* обработали всего 273376 реквестов
* прочитали 17.99MB каких-то данных (заголовки и прочее)
* за 0.045 миллисекунд обработали 1ый запрос
* за 2.842 секунды  обработали 173358 запросов
* сервер обрабатывал 9112.82 запросов в секунду 