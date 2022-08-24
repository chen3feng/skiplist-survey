## Survey of Skip List Implementations



Here is a brief summary of skip list packages available in Go that you may consider using after a quick Google/Github search. If you know of any others, please contact me so I can add them here.

Some things most of the packages have in common: 

- Keys are `int` type, which are 32 or 64 bit depending on GOARCH
- Values are generally of type `interface {}`, so they accept any datatype (Go does not have generics).
- The probability of adding new nodes to each linked level is *P*. The values vary from 0.25 to 0.5. This is an important parameter for performance tuning and memory usage.



Here are some brief notes on each implementation:

- [github.com/mtchavez/skiplist](github.com/mtchavez/skiplist)
  - Values are type `[]byte`, which almost always means conversion.
  - Global constant for *P* value = 0.25, cannot be changed at runtime.
- [github.com/huandu/skiplist](github.com/huandu/skiplist)
  - Globally sets *P* to *almost* 0.25 (using bitmasks and shifting) and can be changed at runtime.
  - You must specify a comparator and type for keys when creating the list.
  - Keys are of type `interface{}`
  - Not threadsafe
- [github.com/zhenjl/skiplist](github.com/zhenjl/skiplist)
  - Adjustable *P* value and max level per list.
  - Adjustable insert level probability per list.
  - Allows duplicates stored at a single key and therefore does not have an update operation.
  - When creating a list, you specify a comparator. It has many built-in that are generated by running an external script that writes out a Go source file with the interfaces.
  - Uses separate search and insert fingers to speed up finding highly local keys consecutively.
  - Threadsafe but fingers are shared as well across all lists
- [github.com/golang-collections/go-datastructures/slice/skip](github.com/golang-collections/go-datastructures/)
  - Intelligently sets maximum level based on key's datatype (uint8 up to uint64)
  - More complex interface; you must define an Entry type that implements the interface it specifies with a comparator
  - *P* value is a global constant, 0.5
- [github.com/ryszard/goskiplist](github.com/ryszard/goskiplist)
  - P value is a global constant, 0.25
  - Very straightforward implementation and interface
  - Not threadsafe
- [github.com/sean-public/fast-skiplist](github.com/sean-public/fast-skiplist)
  - Fastest concurrent implementation in all benchmarks; `huandu` is very close in every benchmark but it is **not** threadsafe.
  - See fast-skiplist's README for details on how this is achieved.



### Benchmarks

Running the benchmarks found in this repo locally is easy:

```sh
go get github.com/sean-public/skiplist-survey
go install github.com/sean-public/skiplist-survey
skiplist-survey > output.csv
```

Chen3feng tested on a MacBook Pro with Apple M1 Pro chip, 32 GB Memory.

The following tables are the result (the less the better, unit: us/op), I have no time to make them graphs.

#### Inserts

|iterations        |100000|200000|300000|400000|500000|600000|700000|800000|900000|1000000|1100000|1200000|1300000|1400000|1500000|1600000|1700000|1800000|1900000|2000000|2100000|2200000|2300000|2400000|2500000|2600000|2700000|2800000|2900000|3000000|FIELD32|
|------------------|------|------|------|------|------|------|------|------|------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|
|chen3fengInserts  |92    |91    |101   |87    |79    |88    |79    |87    |79    |90     |82     |77     |84     |88     |81     |80     |89     |88     |77     |82     |87     |84     |78     |82     |85     |82     |91     |83     |83     |81     |       |
|colInserts        |207   |214   |240   |227   |248   |223   |236   |247   |243   |227    |239    |231    |226    |242    |240    |227    |238    |240    |227    |250    |244    |233    |251    |230    |236    |232    |243    |248    |244    |252    |       |
|huanduInserts     |213   |214   |203   |207   |201   |231   |222   |229   |215   |215    |208    |228    |221    |214    |209    |223    |220    |217    |228    |222    |214    |226    |217    |208    |213    |217    |224    |221    |207    |206    |       |
|liyue201Inserts   |323   |290   |264   |262   |278   |289   |270   |277   |283   |285    |280    |287    |282    |300    |291    |288    |304    |284    |284    |284    |294    |297    |293    |305    |303    |293    |301    |304    |302    |292    |       |
|mtchavezInserts   |290   |297   |237   |244   |259   |253   |235   |255   |249   |245    |271    |292    |251    |228    |243    |235    |237    |243    |239    |229    |241    |231    |244    |235    |225    |234    |236    |232    |236    |224    |       |
|mtInserts         |68    |91    |127   |85    |99    |110   |85    |97    |97    |91     |82     |97     |81     |76     |87     |84     |108    |73     |84     |76     |97     |90     |83     |75     |96     |71     |86     |76     |88     |79     |       |
|seanInserts       |371   |131   |132   |132   |118   |128   |123   |118   |127   |120    |126    |125    |126    |125    |127    |121    |131    |116    |122    |125    |125    |118    |126    |125    |130    |131    |117    |126    |118    |126    |       |
|ryszardInserts    |426   |345   |384   |349   |317   |383   |333   |307   |385   |323    |351    |325    |335    |350    |369    |345    |377    |328    |354    |345    |346    |352    |316    |343    |331    |331    |335    |357    |386    |325    |       |

#### Worst Insert

|iterations        |100000|200000|300000|400000|500000|600000|700000|800000|900000|1000000|1100000|1200000|1300000|1400000|1500000|1600000|1700000|1800000|1900000|2000000|2100000|2200000|2300000|2400000|2500000|2600000|2700000|2800000|2900000|3000000|FIELD32|
|------------------|------|------|------|------|------|------|------|------|------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|
|chen3fengWorstInserts|121   |125   |127   |119   |120   |114   |126   |117   |119   |125    |124    |129    |118    |115    |122    |120    |120    |123    |123    |125    |122    |131    |122    |119    |118    |119    |121    |115    |127    |124    |       |
|colWorstInserts   |249   |247   |241   |248   |267   |266   |277   |277   |278   |269    |277    |294    |281    |268    |278    |274    |280    |291    |268    |281    |281    |278    |279    |281    |281    |286    |288    |278    |291    |274    |       |
|huanduWorstInserts|223   |232   |244   |247   |249   |255   |258   |252   |252   |259    |256    |261    |256    |243    |272    |254    |256    |258    |262    |256    |257    |262    |254    |255    |282    |267    |252    |254    |266    |269    |       |
|liyue201WorstInserts|304   |289   |280   |280   |281   |283   |302   |336   |300   |310    |306    |321    |301    |328    |342    |317    |314    |318    |317    |315    |322    |316    |334    |317    |334    |332    |325    |345    |325    |328    |       |
|mtchavezWorstInserts|213   |212   |222   |258   |261   |266   |261   |262   |256   |263    |261    |254    |264    |268    |256    |256    |262    |255    |249    |251    |266    |250    |250    |252    |251    |259    |252    |274    |249    |251    |       |
|mtWorstInserts    |83    |88    |153   |81    |88    |77    |94    |76    |94    |82     |96     |78     |74     |86     |93     |76     |88     |77     |90     |81     |85     |81     |93     |87     |78     |75     |97     |75     |100    |76     |       |
|seanWorstInserts  |154   |167   |159   |189   |179   |163   |182   |177   |167   |172    |162    |178    |169    |182    |177    |177    |174    |165    |175    |173    |175    |178    |175    |173    |173    |163    |169    |178    |180    |165    |       |
|ryszardWorstInserts|301   |422   |408   |401   |421   |455   |416   |414   |433   |429    |422    |412    |439    |431    |424    |409    |450    |396    |407    |431    |420    |426    |437    |450    |436    |439    |420    |470    |399    |450    |       |


#### Delete

|iterations        |100000|200000|300000|400000|500000|600000|700000|800000|900000|1000000|1100000|1200000|1300000|1400000|1500000|1600000|1700000|1800000|1900000|2000000|2100000|2200000|2300000|2400000|2500000|2600000|2700000|2800000|2900000|3000000|FIELD32|
|------------------|------|------|------|------|------|------|------|------|------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|
|chen3fengDelete   |32    |31    |31    |30    |30    |32    |39    |32    |31    |31     |31     |31     |31     |31     |31     |33     |31     |32     |34     |34     |34     |32     |32     |32     |34     |34     |34     |32     |32     |34     |       |
|colDelete         |141   |158   |148   |141   |165   |144   |148   |152   |154   |167    |149    |166    |147    |148    |154    |152    |148    |156    |150    |164    |165    |157    |152    |159    |154    |159    |152    |157    |164    |173    |       |
|huanduDelete      |79    |78    |79    |78    |77    |77    |77    |76    |77    |77     |77     |84     |77     |79     |74     |78     |87     |74     |79     |77     |77     |77     |77     |77     |77     |76     |77     |79     |77     |77     |       |
|liyue201Delete    |171   |170   |178   |173   |187   |188   |188   |175   |187   |188    |186    |188    |175    |198    |194    |190    |187    |191    |184    |195    |193    |190    |194    |201    |199    |187    |195    |201    |192    |196    |       |
|mtchavezDelete    |160   |217   |162   |258   |212   |149   |155   |163   |206   |191    |190    |171    |169    |189    |206    |164    |216    |196    |189    |187    |151    |164    |188    |167    |185    |204    |203    |237    |214    |159    |       |
|mtDelete          |52    |71    |64    |62    |62    |68    |62    |63    |65    |65     |72     |64     |65     |67     |65     |65     |65     |67     |67     |66     |66     |66     |68     |70     |67     |66     |66     |67     |68     |70     |       |
|seanDelete        |40    |37    |37    |36    |37    |35    |36    |35    |35    |35     |37     |35     |36     |35     |35     |36     |36     |35     |34     |35     |36     |34     |35     |35     |35     |35     |35     |35     |35     |35     |       |
|ryszardDelete     |213   |194   |199   |218   |211   |211   |218   |210   |209   |213    |212    |221    |208    |217    |211    |226    |215    |220    |239    |222    |218    |222    |223    |211    |221    |214    |217    |219    |223    |217    |       |

#### Worst Delete

|iterations        |100000|200000|300000|400000|500000|600000|700000|800000|900000|1000000|1100000|1200000|1300000|1400000|1500000|1600000|1700000|1800000|1900000|2000000|2100000|2200000|2300000|2400000|2500000|2600000|2700000|2800000|2900000|3000000|FIELD32|
|------------------|------|------|------|------|------|------|------|------|------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|
|chen3fengWorstDelete|83    |78    |81    |80    |80    |81    |81    |82    |81    |82     |83     |81     |82     |82     |84     |83     |84     |85     |85     |85     |84     |84     |82     |82     |84     |84     |85     |85     |86     |86     |       |
|colWorstDelete    |196   |189   |199   |195   |206   |200   |204   |209   |212   |215    |204    |220    |212    |219    |210    |211    |210    |215    |212    |215    |223    |219    |204    |210    |217    |218    |215    |228    |222    |218    |       |
|huanduWorstDelete |199   |197   |195   |202   |201   |199   |206   |203   |204   |215    |199    |213    |199    |212    |201    |203    |265    |204    |202    |202    |209    |203    |210    |209    |210    |214    |210    |206    |207    |211    |       |
|liyue201WorstDelete|221   |226   |219   |225   |224   |243   |247   |252   |234   |231    |252    |263    |266    |238    |262    |256    |257    |246    |255    |259    |254    |248    |268    |246    |259    |258    |274    |244    |250    |270    |       |
|mtchavezWorstDelete|148   |192   |201   |218   |225   |188   |224   |238   |190   |209    |188    |226    |222    |208    |196    |212    |187    |205    |226    |263    |213    |208    |247    |194    |251    |205    |238    |207    |218    |201    |       |
|mtWorstDelete     |137   |137   |133   |127   |135   |130   |134   |136   |133   |139    |136    |141    |143    |135    |138    |143    |142    |144    |137    |143    |138    |141    |145    |144    |138    |140    |137    |140    |140    |136    |       |
|seanWorstDelete   |87    |90    |89    |90    |92    |91    |92    |91    |95    |93     |92     |94     |92     |92     |93     |92     |95     |92     |96     |93     |102    |97     |95     |94     |98     |96     |97     |96     |102    |99     |       |
|ryszardWorstDelete|276   |276   |292   |283   |281   |296   |306   |289   |291   |291    |301    |292    |294    |313    |300    |295    |282    |306    |306    |304    |306    |289    |304    |298    |307    |307    |320    |312    |285    |305    |       |


#### Average Search

|iterations        |100000|200000|300000|400000|500000|600000|700000|800000|900000|1000000|1100000|1200000|1300000|1400000|1500000|1600000|1700000|1800000|1900000|2000000|2100000|2200000|2300000|2400000|2500000|2600000|2700000|2800000|2900000|3000000|FIELD32|
|------------------|------|------|------|------|------|------|------|------|------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|
|chen3fengAvgSearch|60    |57    |57    |58    |58    |59    |58    |65    |65    |65     |64     |65     |65     |64     |64     |65     |64     |64     |64     |64     |64     |64     |68     |67     |66     |66     |67     |67     |63     |63     |       |
|colAvgSearch      |199   |188   |196   |197   |201   |203   |201   |217   |217   |215    |210    |216    |221    |213    |218    |221    |219    |226    |215    |216    |210    |224    |224    |213    |225    |222    |216    |213    |218    |215    |       |
|huanduAvgSearch   |168   |167   |170   |175   |181   |167   |186   |173   |186   |172    |174    |179    |178    |177    |178    |183    |175    |184    |179    |184    |178    |182    |180    |178    |183    |184    |182    |184    |188    |182    |       |
|liyue201AvgSearch |287   |310   |309   |316   |320   |335   |330   |344   |341   |344    |352    |355    |353    |356    |343    |341    |341    |358    |349    |352    |365    |356    |346    |362    |354    |363    |354    |387    |378    |360    |       |
|mtchavezAvgSearch |65    |62    |65    |68    |66    |71    |69    |68    |67    |66     |71     |71     |67     |71     |69     |73     |69     |68     |69     |69     |68     |73     |73     |70     |72     |71     |77     |76     |71     |69     |       |
|mtAvgSearch       |84    |98    |89    |98    |87    |90    |89    |89    |90    |92     |93     |93     |91     |94     |93     |90     |94     |95     |98     |92     |94     |93     |95     |95     |94     |93     |94     |97     |95     |95     |       |
|seanAvgSearch     |77    |76    |75    |74    |75    |77    |83    |79    |79    |85     |79     |83     |75     |81     |77     |81     |77     |82     |83     |82     |84     |80     |80     |85     |77     |80     |86     |82     |80     |81     |       |
|ryszardAvgSearch  |169   |168   |177   |171   |166   |172   |178   |191   |174   |183    |182    |184    |182    |178    |182    |179    |186    |180    |178    |184    |183    |197    |221    |196    |205    |195    |194    |189    |188    |185    |       |


#### Search End

|iterations        |100000|200000|300000|400000|500000|600000|700000|800000|900000|1000000|1100000|1200000|1300000|1400000|1500000|1600000|1700000|1800000|1900000|2000000|2100000|2200000|2300000|2400000|2500000|2600000|2700000|2800000|2900000|3000000|FIELD32|
|------------------|------|------|------|------|------|------|------|------|------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|
|chen3fengSearchEnd|32    |35    |31    |32    |48    |48    |62    |46    |37    |31     |45     |66     |44     |43     |43     |38     |37     |49     |55     |39     |40     |34     |34     |49     |45     |32     |40     |43     |55     |48     |       |
|colSearchEnd      |100   |83    |95    |125   |127   |95    |113   |119   |112   |163    |96     |125    |145    |136    |132    |144    |80     |95     |123    |114    |104    |156    |87     |138    |166    |108    |143    |152    |132    |102    |       |
|huanduSearchEnd   |19    |18    |18    |17    |17    |19    |17    |18    |19    |16     |17     |17     |17     |17     |18     |18     |17     |18     |17     |18     |17     |17     |26     |17     |17     |17     |18     |17     |17     |18     |       |
|liyue201SearchEnd |231   |152   |178   |164   |146   |141   |242   |263   |256   |279    |117    |184    |226    |124    |165    |212    |172    |226    |288    |180    |203    |192    |203    |286    |414    |300    |262    |203    |163    |263    |       |
|mtchavezSearchEnd |47    |25    |38    |46    |43    |37    |63    |59    |35    |57     |36     |47     |58     |39     |40     |46     |46     |51     |57     |46     |45     |45     |34     |53     |29     |39     |59     |85     |33     |39     |       |
|mtSearchEnd       |63    |54    |47    |56    |46    |57    |54    |53    |50    |48     |48     |53     |55     |57     |55     |54     |66     |52     |68     |68     |53     |46     |46     |65     |50     |54     |74     |56     |50     |51     |       |
|seanSearchEnd     |60    |67    |65    |49    |48    |47    |44    |48    |79    |63     |38     |33     |54     |42     |42     |61     |57     |50     |55     |63     |74     |53     |73     |46     |34     |76     |64     |62     |42     |65     |       |
|ryszardSearchEnd  |107   |120   |128   |101   |124   |153   |159   |118   |93    |101    |157    |63     |116    |94     |147    |131    |115    |156    |145    |131    |129    |160    |106    |158    |161    |110    |97     |108    |308    |141    |       |

#### chen3feng golang impl

|iterations           |100000|200000|300000|400000|500000|600000|700000|800000|900000|1000000|1100000|1200000|1300000|1400000|1500000|1600000|1700000|1800000|1900000|2000000|2100000|2200000|2300000|2400000|2500000|2600000|2700000|2800000|2900000|3000000|FIELD32|
|---------------------|------|------|------|------|------|------|------|------|------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|
|chen3fengAvgSearch   |53    |60    |61    |60    |60    |62    |62    |63    |63    |63     |64     |63     |64     |64     |61     |62     |62     |64     |63     |65     |67     |66     |67     |65     |65     |65     |65     |67     |67     |67     |       |
|chen3fengDelete      |30    |30    |30    |29    |30    |31    |31    |33    |33    |32     |31     |32     |31     |31     |31     |32     |33     |32     |33     |33     |34     |34     |33     |33     |32     |32     |33     |33     |33     |32     |       |
|chen3fengInserts     |101   |88    |94    |82    |81    |80    |76    |79    |93    |78     |72     |87     |82     |74     |81     |81     |82     |85     |89     |79     |74     |81     |77     |86     |78     |83     |84     |80     |77     |79     |       |
|chen3fengSearchEnd   |46    |39    |33    |58    |56    |28    |29    |25    |51    |30     |37     |28     |40     |26     |57     |44     |47     |45     |36     |35     |41     |45     |44     |66     |40     |38     |23     |29     |42     |58     |       |
|chen3fengWorstDelete |74    |77    |77    |79    |80    |86    |81    |81    |83    |83     |82     |82     |82     |82     |82     |82     |82     |82     |82     |84     |85     |83     |84     |83     |83     |85     |85     |82     |82     |84     |       |
|chen3fengWorstInserts|107   |109   |116   |119   |110   |122   |120   |118   |122   |115    |116    |122    |123    |115    |119    |121    |119    |117    |124    |116    |127    |119    |129    |122    |120    |124    |122    |114    |118    |125    |       |


#### chen3feng C++ impl

|iterations           |100000|200000|300000|400000|500000|600000|700000|800000|900000|1000000|1100000|1200000|1300000|1400000|1500000|1600000|1700000|1800000|1900000|2000000|2100000|2200000|2300000|2400000|2500000|2600000|2700000|2800000|2900000|3000000|
|---------------------|------|------|------|------|------|------|------|------|------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|
|chen3fengAvgSearch   |91    |65    |57    |58    |60    |61    |61    |62    |62    |63     |61     |62     |62     |61     |61     |61     |61     |62     |65     |66     |63     |63     |65     |63     |65     |63     |64     |65     |65     |65     |
|chen3fengDelete      |79    |79    |75    |75    |74    |76    |80    |77    |82    |82     |86     |89     |84     |86     |85     |81     |82     |84     |81     |83     |82     |84     |83     |83     |84     |81     |84     |88     |92     |89     |
|chen3fengInserts     |51    |56    |49    |49    |49    |48    |48    |48    |47    |46     |48     |48     |49     |48     |47     |47     |48     |48     |48     |49     |50     |50     |49     |48     |49     |48     |48     |49     |50     |48     |
|chen3fengSearchEnd   |27    |30    |26    |26    |23    |48    |33    |30    |35    |27     |34     |35     |35     |24     |34     |28     |28     |42     |32     |34     |40     |29     |28     |33     |34     |33     |34     |35     |31     |41     |
|chen3fengWorstDelete |102   |101   |107   |106   |106   |106   |105   |106   |105   |104    |104    |106    |105    |107    |105    |105    |104    |106    |114    |107    |111    |109    |108    |108    |109    |108    |109    |108    |108    |109    |
|chen3fengWorstInserts|89    |93    |92    |93    |94    |97    |97    |96    |96    |97     |97     |97     |98     |98     |99     |98     |97     |97     |97     |100    |100    |99     |100    |101    |100    |101    |101    |101    |100    |99     |


### Result Usage


The results are in CSV format for easy charting and analysis.

Here is a summary of results I recorded on a Macbook Pro 15 with a 2.7 GHz Intel Core i7 and 16GB RAM. It takes over an hour to run all the benchmarks.

![best inserts chart](http://i.imgur.com/Vo5etzd.png)

The chart above shows the **best-case insert** speeds. The vertical axis is nanoseconds per operation and the horizontal is the number of items in the list. These are the "best" inserts because they happen at the front of the list, which shouldn't require any searching. The difference in speed here demonstrates the overhead each package introduces in even the most basic operations.



![worst inserts chart](http://i.imgur.com/Z47mCm1.png)

**Worst-case inserts.** These inserts are at the end of the list, requiring searching all the way to the end and then adding the new node. As you can see, `mtchavez` does not scale as well as the other implementations, which only show a small variance even after millions of nodes are added.



![average search chart](http://i.imgur.com/OFgOZQu.png)

**Average search speed**. You can see `mtchavez` again is not searching in *O(log n)* or even *O(n)*. `zhenjl` also appears to have a lot of overhead in its search compared to the other implementations.



![worst case delete chart](http://i.imgur.com/LxSov5E.png)

**Worst case deletions**. In this benchmark, a skip list of a given length is created and then every item is removed, starting from the last one and moving to the front.



![zoom worse cases deletions](http://i.imgur.com/LQYoXuO.png)

If we zoom in, we can see the speed differences between the fastest implementations a bit better. The fastest overall is `sean`, which averages 10-50ns faster in all operations than the next fastest implementation.



### Todo

- (in progress now) Add multithreaded benchmarks with varying concurrency and read/write load proportions.

- Use very tall (max height) skiplists to demonstrate stress caused by multiple calls to random functions on insert.

- Record total and high-water mark memory use.

- Benchmark concurrent inserts on multiple lists to stress globally-locked PRNG in most implementations.

  ​



