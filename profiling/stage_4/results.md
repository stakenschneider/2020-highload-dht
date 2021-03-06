# Шардирование
На этом шаге реализовали горизонтальное масштабирование через поддержку кластерных конфигураций, 
состоящих из нескольких узлов, взаимодействующих друг с другом через HTTP API.

## Параметры машины

<b>CPU:</b> 1,2 GHz 4‑ядерный процессор Intel Core i7

<b>System:</b> Mac OS X (10.15.7)

<b>Architecture:</b> x86_64 64bit

## VisualVM
#### Вкладка Threads:
![Screen from visual vm threads](visual-vm-threads-2.png?raw=true "Screen from visual vm threads")
![Screen from visual vm threads](visual-vm-threads-1.png?raw=true "Screen from visual vm threads")
По сравнению с предыдущей версией у нас по набору потоков для каждой из трех нод.

#### Вкладка Monitor:
![Screen from visual vm monitor](visual-vm-monitor.png?raw=true "Screen from visual vm monitor")
<b>Картина на CPU usage: </b>
3 раза нагрузка put-ами, get, put, 2 get-а. Put-ы занимают в районе 50 процентов CPU, get-ы при тех же параметрах
 40 процентов. Подробные результаты представлены ниже.


## PUT
Запускаю wrk c такими параметрами:
+ keeping 64 HTTP connections open
+ 30 seconds
+ constant throughput of 32000 requests per second

Результаты с async profiler-а (CPU):
![Flame graph from async profiler](put-cpu-16-30-32000.svg?raw=true "Flame graph from async profiler")

Результаты с async profiler-а (CPU без -t):
![Flame graph from async profiler](put-cpu-4-16-30-60000.svg?raw=true "Flame graph from async profiler")

Результаты с async profiler-а (ALLOC):
![Flame graph from async profiler](put-alloc-16-30-32000.svg?raw=true "Flame graph from async profiler")

Результаты с async profiler-а (ALLOC без -t):
![Flame graph from async profiler](put-alloc-4-16-30-60000.svg?raw=true "Flame graph from async profiler")

Результаты с async profiler-а (LOCK):
![Flame graph from async profiler](put-lock-16-30-32000.svg?raw=true "Flame graph from async profiler")

Результаты с async profiler-а (LOCK без -t):
![Flame graph from async profiler](put-lock-4-16-30-60000.svg?raw=true "Flame graph from async profiler")

На первый взгляд, по сравнению с предыдущей версией, изменилось только количество "башенок". Всего в три раза больше.

Результаты wrk2:
```
Running 30s test @ http://127.0.0.1:8080
  2 threads and 64 connections
  Thread calibration: mean lat.: 4.698ms, rate sampling interval: 20ms
  Thread calibration: mean lat.: 4.706ms, rate sampling interval: 23ms
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     2.51ms    4.02ms  45.92ms   96.03%
    Req/Sec    16.39k     1.35k   25.95k    88.61%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%    1.70ms
 75.000%    2.43ms
 90.000%    3.33ms
 99.000%   26.88ms
 99.900%   40.74ms
 99.990%   44.32ms
 99.999%   45.34ms
100.000%   45.95ms
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)
       0.073     0.000000            1         1.00
       0.807     0.100000        63647         1.11
       1.076     0.200000       126982         1.25
       1.300     0.300000       190597         1.43
       1.498     0.400000       254189         1.67
       1.702     0.500000       317465         2.00
       1.815     0.550000       349365         2.22
       1.940     0.600000       380944         2.50
       2.089     0.650000       412692         2.86
       2.251     0.700000       444623         3.33
       2.429     0.750000       476126         4.00
       2.531     0.775000       492042         4.44
       2.645     0.800000       507902         5.00
       2.775     0.825000       523950         5.71
       2.923     0.850000       539750         6.67
       3.107     0.875000       555569         8.00
       3.211     0.887500       563420         8.89
       3.327     0.900000       571450        10.00
       3.457     0.912500       579295        11.43
       3.625     0.925000       587290        13.33
       3.879     0.937500       595185        16.00
       4.103     0.943750       599197        17.78
       4.507     0.950000       603104        20.00
       5.507     0.956250       607062        22.86
       7.199     0.962500       611033        26.67
       9.615     0.968750       614995        32.00
      11.783     0.971875       616985        35.56
      13.967     0.975000       618965        40.00
      15.871     0.978125       620957        45.71
      17.775     0.981250       622931        53.33
      19.951     0.984375       624917        64.00
      21.631     0.985938       625908        71.11
      23.663     0.987500       626906        80.00
      25.615     0.989062       627890        91.43
      27.663     0.990625       628883       106.67
      29.551     0.992188       629874       128.00
      30.447     0.992969       630374       142.22
      31.359     0.993750       630867       160.00
      32.191     0.994531       631374       182.86
      32.991     0.995313       631875       213.33
      33.727     0.996094       632362       256.00
      34.175     0.996484       632612       284.44
      34.687     0.996875       632864       320.00
      35.263     0.997266       633098       365.71
      36.031     0.997656       633349       426.67
      37.087     0.998047       633599       512.00
      37.791     0.998242       633719       568.89
      38.527     0.998437       633847       640.00
      39.167     0.998633       633969       731.43
      39.999     0.998828       634093       853.33
      40.831     0.999023       634217      1024.00
      41.183     0.999121       634281      1137.78
      41.503     0.999219       634338      1280.00
      41.823     0.999316       634401      1462.86
      42.175     0.999414       634464      1706.67
      42.591     0.999512       634530      2048.00
      42.719     0.999561       634555      2275.56
      42.911     0.999609       634591      2560.00
      43.007     0.999658       634617      2925.71
      43.231     0.999707       634652      3413.33
      43.487     0.999756       634681      4096.00
      43.583     0.999780       634695      4551.11
      43.743     0.999805       634712      5120.00
      43.871     0.999829       634728      5851.43
      44.031     0.999854       634743      6826.67
      44.127     0.999878       634756      8192.00
      44.223     0.999890       634764      9102.22
      44.319     0.999902       634772     10240.00
      44.383     0.999915       634781     11702.86
      44.479     0.999927       634787     13653.33
      44.671     0.999939       634796     16384.00
      44.799     0.999945       634799     18204.44
      44.831     0.999951       634803     20480.00
      44.863     0.999957       634806     23405.71
      44.895     0.999963       634810     27306.67
      44.959     0.999969       634814     32768.00
      45.023     0.999973       634816     36408.89
      45.055     0.999976       634818     40960.00
      45.087     0.999979       634820     46811.43
      45.119     0.999982       634822     54613.33
      45.183     0.999985       634825     65536.00
      45.183     0.999986       634825     72817.78
      45.247     0.999988       634826     81920.00
      45.343     0.999989       634828     93622.86
      45.343     0.999991       634828    109226.67
      45.439     0.999992       634830    131072.00
      45.439     0.999993       634830    145635.56
      45.439     0.999994       634830    163840.00
      45.439     0.999995       634830    187245.71
      45.503     0.999995       634831    218453.33
      45.503     0.999996       634831    262144.00
      45.503     0.999997       634831    291271.11
      45.727     0.999997       634832    327680.00
      45.727     0.999997       634832    374491.43
      45.727     0.999998       634832    436906.67
      45.727     0.999998       634832    524288.00
      45.727     0.999998       634832    582542.22
      45.951     0.999998       634833    655360.00
      45.951     1.000000       634833          inf
#[Mean    =        2.506, StdDeviation   =        4.021]
#[Max     =       45.920, Total count    =       634833]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  957479 requests in 30.00s, 75.79MB read
Requests/sec:  31917.33
Transfer/sec:      2.53MB
```

Итоги:
* обработали всего 957479 запросов
* прочитали 75.79MB данных 
* сервер обрабатывал 31917.33 запросов в секунду 

Для нашего сервера уже тяжеловато обрабатывать 32000 запросов в секунду, еле-еле с этим справляется. 

## GET
Запускаю wrk c такими параметрами:
+ keeping 16 HTTP connections open
+ 30 seconds
+ constant throughput of 32000 requests per second

Результаты с async profiler-а (CPU):
![Flame graph from async profiler](get-cpu-16-30-32000.svg?raw=true "Flame graph from async profiler")

Результаты с async profiler-а (CPU без -t):
![Flame graph from async profiler](get-lock-4-16-30-60000.svg?raw=true "Flame graph from async profiler")

Результаты с async profiler-а (ALLOC):
![Flame graph from async profiler](get-alloc-16-30-32000.svg?raw=true "Flame graph from async profiler")

Результаты с async profiler-а (ALLOC без -t):
![Flame graph from async profiler](get-lock-4-16-30-60000.svg?raw=true "Flame graph from async profiler")

Результаты с async profiler-а (LOCK):
![Flame graph from async profiler](get-lock-16-30-32000.svg?raw=true "Flame graph from async profiler")
Картина та же самая, всего по три комплекта, но сам стек одинаковый для всех.

Результаты с async profiler-а (LOCK без -t):
![Flame graph from async profiler](get-lock-4-16-30-60000.svg?raw=true "Flame graph from async profiler")

Результаты wrk2:
```
Running 30s test @ http://127.0.0.1:8080
  2 threads and 64 connections
  Thread calibration: mean lat.: 28.613ms, rate sampling interval: 210ms
  Thread calibration: mean lat.: 36.553ms, rate sampling interval: 262ms
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    31.06ms   59.56ms 208.00ms   81.86%
    Req/Sec    15.97k     0.95k   18.11k    88.76%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%    2.49ms
 75.000%    6.85ms
 90.000%  144.26ms
 99.000%  196.22ms
 99.900%  202.62ms
 99.990%  206.72ms
 99.999%  207.74ms
100.000%  208.13ms
  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)
       0.118     0.000000            1         1.00
       1.190     0.100000        63417         1.11
       1.523     0.200000       126723         1.25
       1.787     0.300000       189891         1.43
       2.109     0.400000       253403         1.67
       2.487     0.500000       316647         2.00
       2.721     0.550000       348205         2.22
       3.049     0.600000       379708         2.50
       3.517     0.650000       411351         2.86
       4.231     0.700000       442972         3.33
       6.851     0.750000       474604         4.00
       9.255     0.775000       490445         4.44
      16.431     0.800000       506249         5.00
      98.623     0.825000       522070         5.71
     124.735     0.850000       537915         6.67
     135.039     0.875000       553743         8.00
     138.879     0.887500       561750         8.89
     144.255     0.900000       569610        10.00
     154.879     0.912500       577521        11.43
     164.735     0.925000       585446        13.33
     171.135     0.937500       593344        16.00
     174.207     0.943750       597226        17.78
     178.047     0.950000       601166        20.00
     182.783     0.956250       605242        22.86
     185.855     0.962500       609083        26.67
     188.543     0.968750       613124        32.00
     189.695     0.971875       615072        35.56
     190.847     0.975000       617019        40.00
     191.999     0.978125       619056        45.71
     193.023     0.981250       621086        53.33
     194.047     0.984375       622975        64.00
     194.687     0.985938       624078        71.11
     195.199     0.987500       624912        80.00
     195.839     0.989062       625934        91.43
     196.607     0.990625       627004       106.67
     197.375     0.992188       628022       128.00
     197.759     0.992969       628454       142.22
     198.143     0.993750       628941       160.00
     198.527     0.994531       629398       182.86
     198.911     0.995313       629846       213.33
     199.423     0.996094       630381       256.00
     199.679     0.996484       630642       284.44
     199.935     0.996875       630882       320.00
     200.191     0.997266       631078       365.71
     200.575     0.997656       631321       426.67
     201.087     0.998047       631605       512.00
     201.343     0.998242       631736       568.89
     201.599     0.998437       631830       640.00
     201.983     0.998633       631976       731.43
     202.239     0.998828       632090       853.33
     202.751     0.999023       632204      1024.00
     203.135     0.999121       632251      1137.78
     203.519     0.999219       632308      1280.00
     204.159     0.999316       632394      1462.86
     204.543     0.999414       632449      1706.67
     204.927     0.999512       632501      2048.00
     205.183     0.999561       632534      2275.56
     205.439     0.999609       632567      2560.00
     205.567     0.999658       632596      2925.71
     205.823     0.999707       632642      3413.33
     205.951     0.999756       632659      4096.00
     206.079     0.999780       632687      4551.11
     206.079     0.999805       632687      5120.00
     206.207     0.999829       632704      5851.43
     206.335     0.999854       632718      6826.67
     206.463     0.999878       632730      8192.00
     206.591     0.999890       632737      9102.22
     206.719     0.999902       632747     10240.00
     206.847     0.999915       632753     11702.86
     206.975     0.999927       632758     13653.33
     207.103     0.999939       632772     16384.00
     207.103     0.999945       632772     18204.44
     207.103     0.999951       632772     20480.00
     207.231     0.999957       632778     23405.71
     207.359     0.999963       632784     27306.67
     207.359     0.999969       632784     32768.00
     207.487     0.999973       632791     36408.89
     207.487     0.999976       632791     40960.00
     207.487     0.999979       632791     46811.43
     207.487     0.999982       632791     54613.33
     207.615     0.999985       632794     65536.00
     207.615     0.999986       632794     72817.78
     207.743     0.999988       632796     81920.00
     207.743     0.999989       632796     93622.86
     207.871     0.999991       632797    109226.67
     207.999     0.999992       632800    131072.00
     207.999     0.999993       632800    145635.56
     207.999     0.999994       632800    163840.00
     207.999     0.999995       632800    187245.71
     207.999     0.999995       632800    218453.33
     207.999     0.999996       632800    262144.00
     207.999     0.999997       632800    291271.11
     208.127     0.999997       632802    327680.00
     208.127     1.000000       632802          inf
#[Mean    =       31.058, StdDeviation   =       59.560]
#[Max     =      208.000, Total count    =       632802]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  955407 requests in 30.00s, 76.32MB read
Requests/sec:  31847.82
Transfer/sec:      2.54MB
```

Итоги:
* обработали всего 955407 запросов
* прочитали 76.32MB данных 
* сервер обрабатывал 31847.82 запросов в секунду 

## Вывод.
Результаты особо не изменились с прошлой реализации,возможно потому что, по сути, мы хоть 
и шарим, но работаем на одной машине.