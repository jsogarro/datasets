Skip to content 

[ ![logo](../../local/img/kx.svg) ](https://code.kx.com/ "code.kx.com")

kdb+ and q documentation 

Market Fragmentation: A kdb+ framework for multiple liquidity sources | kdb+ and q documentation 

Initializing search 




 Ask a question

  * [ Home ](https://code.kx.com/home)
  * [ kdb+ and q ](../..)
  * [ kdb Insights SDK ](https://code.kx.com/insights)
  * [ kdb Insights Enterprise ](https://code.kx.com/insights/enterprise)
  * [ KDB.AI ](https://code.kx.com/kdbai)
  * [ PyKX ](https://code.kx.com/pykx)
  * [ APIs ](https://code.kx.com/insights/api)
  * [ Help ](https://code.kx.com/home/support.html)



[ ![logo](../../local/img/kx.svg) ](../.. "kdb+ and q documentation") kdb+ and q documentation 

  * [ Home ](https://code.kx.com/home)
  * kdb+ and q  kdb+ and q 
    * [ About ](../..)
    * Getting Started  Getting Started 
      * [ Install ](../../learn/install/)
      * [ Licenses ](../../learn/licensing/)
    * Learn  Learn 
      * [ Overview ](../../learn/)
      * Mountain tour  Mountain tour 
        * [ Overview ](../../learn/tour/overview/)
        * [ Begin here ](../../learn/tour/)
        * [ The q session ](../../learn/tour/session/)
        * [ Tables ](../../learn/tour/tables/)
        * [ CSVs ](../../learn/tour/csvs/)
        * [ Datatypes ](../../learn/tour/datatypes/)
        * [ Scripts ](../../learn/tour/scripts/)
        * [ IDE ](../../learn/tour/ide/)
      * [ Q for quants ](../../learn/brief-introduction/)
      * [ Q by Examples ](../../learn/q-by-examples/)
      * [ Q for All (video) ](../../learn/q-for-all/)
      * Examples from Python  Examples from Python 
        * [ Basic ](../../learn/python/examples/)
        * [ Array ](../../learn/python/examples/array/)
        * [ List ](../../learn/python/examples/list/)
        * [ Strings ](../../learn/python/examples/string/)
        * [ Dictionaries ](../../learn/python/examples/dict/)
      * [ Q for Mortals 3 ](https://code.kx.com/q4m3/)
      * Q by Puzzles  Q by Puzzles 
        * [ About ](../../learn/pb/)
        * [ 12 Days of Xmas ](../../learn/pb/xmas-days/)
        * [ ABC problem ](../../learn/pb/abc-problem/)
        * [ Abundant odds ](../../learn/pb/abundant-odds/)
        * [ Four is magic ](../../learn/pb/four-magic/)
        * [ Name Game ](../../learn/pb/name-game/)
        * [ Summarize and Say ](../../learn/pb/sum-say/)
        * [ Word wheel ](../../learn/pb/word-wheel/)
      * Reading room  Reading room 
        * [ Information desk ](../../learn/reading/)
        * [ Boggle ](../../learn/reading/boggle/)
        * [ Cats cradle ](../../learn/reading/strings/)
        * [ Fizz buzz ](../../learn/reading/fizzbuzz/)
        * [ Klondike ](../../learn/reading/klondike/)
        * [ Phrasebook ](https://code.kx.com/phrases/)
        * [ Scrabble ](../../learn/reading/scrabble/)
      * Application examples  Application examples 
        * [ Astronomy ](../astronomy/)
        * [ Detecting card counters ](../card-counters/)
        * [ Corporate actions ](../corporate-actions/)
        * [ Disaster management ](../disaster-management/)
        * [ Exoplanets ](../exoplanets/)
        * [ Market depth ](../market-depth/)
        * Market fragmentation  [ Market fragmentation ](./) On this page 
          * Motivation 
          * The data 
          * The building blocks 
            * Analytic setup 
            * Data filtering 
          * Reference data 
          * Consolidating the data 
            * Extending parameters 
            * Multi-market aggregation 
          * Conclusion 
          * Author 
          * Appendix 
        * [ Option pricing ](../option-pricing/)
        * [ Predicting floods ](../disaster-floods/)
        * [ Signal processing ](../signal-processing/)
        * [ Space weather ](../space-weather/)
        * [ Trading surveillance ](../surveillance/)
        * [ Transaction-cost analysis ](../transaction-cost/)
        * [ Trend indicators ](../trend-indicators/)
      * Advanced q  Advanced q 
        * [ Remarks on Style ](https://github.com/qbists/style)
        * [ Shifts & scans ](../../learn/shifts-scans/)
        * [ Technical articles ](../../learn/blogs/)
        * [ Views ](../../learn/views/)
        * [ Origins ](../../learn/archive/)
        * [ Terminology ](../../about/terminology/)
      * Starting kdb+  Starting kdb+ 
        * [ Overview ](../../learn/startingkdb/)
        * [ The q language ](../../learn/startingkdb/language/)
        * [ IPC ](../../learn/startingkdb/ipc/)
        * [ Tables ](../../learn/startingkdb/tables/)
        * [ Historical database ](../../learn/startingkdb/hdb/)
        * [ Realtime database ](../../learn/startingkdb/tick/)
    * Language  Language 
      * [ Reference card ](../../ref/)
      * [ By topic ](../../basics/by-topic/)
      * Iteration  Iteration 
        * [ Overview ](../../basics/iteration/)
        * [ Implicit iteration ](../../basics/implicit-iteration/)
        * [ Iterators ](../../ref/iterators/)
        * [ Maps ](../../ref/maps/)
        * [ Accumulators ](../../ref/accumulators/)
        * [ Guide to iterators ](../iterators/)
      * Keywords  Keywords 
        * [ abs ](../../ref/abs/)
        * [ aj, aj0, ajf, ajf0 ](../../ref/aj/)
        * [ all, any ](../../ref/all-any/)
        * [ and ](../../ref/and/)
        * [ asc, iasc, xasc ](../../ref/asc/)
        * [ asof ](../../ref/asof/)
        * [ attr ](../../ref/attr/)
        * [ avg, avgs, mavg, wavg ](../../ref/avg/)
        * [ bin, binr ](../../ref/bin/)
        * [ ceiling ](../../ref/ceiling/)
        * [ count, mcount ](../../ref/count/)
        * [ cols, xcol, xcols ](../../ref/cols/)
        * [ cor ](../../ref/cor/)
        * [ cos, acos ](../../ref/cos/)
        * [ cov, scov ](../../ref/cov/)
        * [ cross ](../../ref/cross/)
        * [ csv ](../../ref/csv/)
        * [ cut ](../../ref/cut/)
        * [ delete ](../../ref/delete/)
        * [ deltas ](../../ref/deltas/)
        * [ desc, idesc, xdesc ](../../ref/desc/)
        * [ dev, mdev, sdev ](../../ref/dev/)
        * [ differ ](../../ref/differ/)
        * [ distinct ](../../ref/distinct/)
        * [ div ](../../ref/div/)
        * [ dsave ](../../ref/dsave/)
        * [ each, peach ](../../ref/each/)
        * [ ej ](../../ref/ej/)
        * [ ema ](../../ref/ema/)
        * [ enlist ](../../ref/enlist/)
        * [ eval, reval ](../../ref/eval/)
        * [ except ](../../ref/except/)
        * [ exec ](../../ref/exec/)
        * [ exit ](../../ref/exit/)
        * [ exp, xexp ](../../ref/exp/)
        * [ fby ](../../ref/fby/)
        * [ fills ](../../ref/fill/)
        * [ first, last ](../../ref/first/)
        * [ fkeys ](../../ref/fkeys/)
        * [ flip ](../../ref/flip/)
        * [ floor ](../../ref/floor/)
        * [ get, set ](../../ref/get/)
        * [ getenv, setenv ](../../ref/getenv/)
        * [ group ](../../ref/group/)
        * [ gtime, ltime ](../../ref/gtime/)
        * [ hcount ](../../ref/hcount/)
        * [ hdel ](../../ref/hdel/)
        * [ hopen, hclose ](../../ref/hopen/)
        * [ hsym ](../../ref/hsym/)
        * [ ij, ijf ](../../ref/ij/)
        * [ in ](../../ref/in/)
        * [ insert ](../../ref/insert/)
        * [ inter ](../../ref/inter/)
        * [ inv ](../../ref/inv/)
        * [ key ](../../ref/key/)
        * [ keys, xkey ](../../ref/keys/)
        * [ like ](../../ref/like/)
        * [ lj, ljf ](../../ref/lj/)
        * [ load, rload ](../../ref/load/)
        * [ log, xlog ](../../ref/log/)
        * [ lower, upper ](../../ref/lower/)
        * [ lsq ](../../ref/lsq/)
        * [ max, maxs, mmax ](../../ref/max/)
        * [ md5 ](../../ref/md5/)
        * [ med ](../../ref/med/)
        * [ meta ](../../ref/meta/)
        * [ min, mins, mmin ](../../ref/min/)
        * [ mmu ](../../ref/mmu/)
        * [ mod ](../../ref/mod/)
        * [ neg ](../../ref/neg/)
        * [ next, prev, xprev ](../../ref/next/)
        * [ not ](../../ref/not/)
        * [ null ](../../ref/null/)
        * [ or ](../../ref/or/)
        * [ over, scan ](../../ref/over/)
        * [ parse ](../../ref/parse/)
        * [ pj ](../../ref/pj/)
        * [ prd, prds ](../../ref/prd/)
        * [ prior ](../../ref/prior/)
        * [ rand ](../../ref/rand/)
        * [ rank ](../../ref/rank/)
        * [ ratios ](../../ref/ratios/)
        * [ raze ](../../ref/raze/)
        * [ read0 ](../../ref/read0/)
        * [ read1 ](../../ref/read1/)
        * [ reciprocal ](../../ref/reciprocal/)
        * [ reverse ](../../ref/reverse/)
        * [ rotate ](../../ref/rotate/)
        * [ save, rsave ](../../ref/save/)
        * [ select ](../../ref/select/)
        * [ show ](../../ref/show/)
        * [ signum ](../../ref/signum/)
        * [ sin, asin ](../../ref/sin/)
        * [ sqrt ](../../ref/sqrt/)
        * [ ss, ssr ](../../ref/ss/)
        * [ string ](../../ref/string/)
        * [ sublist ](../../ref/sublist/)
        * [ sum, sums, msum, wsum ](../../ref/sum/)
        * [ sv ](../../ref/sv/)
        * [ system ](../../ref/system/)
        * [ tables ](../../ref/tables/)
        * [ tan, atan ](../../ref/tan/)
        * [ til ](../../ref/til/)
        * [ trim, ltrim, rtrim ](../../ref/trim/)
        * [ type ](../../ref/type/)
        * [ uj, ujf ](../../ref/uj/)
        * [ union ](../../ref/union/)
        * [ ungroup ](../../ref/ungroup/)
        * [ update ](../../ref/update/)
        * [ upsert ](../../ref/upsert/)
        * [ value ](../../ref/value/)
        * [ var, svar ](../../ref/var/)
        * [ view, views ](../../ref/view/)
        * [ vs ](../../ref/vs/)
        * [ where ](../../ref/where/)
        * [ within ](../../ref/within/)
        * [ wj, wj1 ](../../ref/wj/)
        * [ xbar ](../../ref/xbar/)
        * [ xgroup ](../../ref/xgroup/)
        * [ xrank ](../../ref/xrank/)
      * [ Overloaded glyphs ](../../ref/overloads/)
      * Operators  Operators 
        * [ Add ](../../ref/add/)
        * [ Amend ](../../ref/amend/)
        * [ Apply, Index, Trap ](../../ref/apply/)
        * [ Assign ](../../ref/assign/)
        * [ Cast ](../../ref/cast/)
        * [ Coalesce ](../../ref/coalesce/)
        * [ Compose ](../../ref/compose/)
        * [ Cut ](../../ref/cut/)
        * [ Deal, Roll, Permute ](../../ref/deal/)
        * [ Delete ](../../ref/delete/)
        * [ Display ](../../ref/display/)
        * [ Dict ](../../ref/dict/)
        * [ Divide ](../../ref/divide/)
        * [ Dynamic Load ](../../ref/dynamic-load/)
        * [ Drop ](../../ref/drop/)
        * [ Enkey, Unkey ](../../ref/enkey/)
        * [ Enumerate ](../../ref/enumerate/)
        * [ Enumeration ](../../ref/enumeration/)
        * [ Enum Extend ](../../ref/enum-extend/)
        * [ Equal ](../../ref/equal/)
        * [ Exec ](../../ref/exec/)
        * [ File Binary ](../../ref/file-binary/)
        * [ File Text ](../../ref/file-text/)
        * [ Fill ](../../ref/fill/)
        * [ Find ](../../ref/find/)
        * [ Flip Splayed ](../../ref/flip-splayed/)
        * [ Greater ](../../ref/greater/)
        * [ Greater Than ](../../ref/greater-than/)
        * [ Identity, Null ](../../ref/identity/)
        * [ Join ](../../ref/join/)
        * [ Less Than ](../../ref/less-than/)
        * [ Lesser ](../../ref/lesser/)
        * [ Match ](../../ref/match/)
        * [ Matrix Multiply ](../../ref/mmu/)
        * [ Multiply ](../../ref/multiply/)
        * [ Not Equal ](../../ref/not-equal/)
        * [ Pad ](../../ref/pad/)
        * [ Select ](../../ref/select/)
        * [ Set Attribute ](../../ref/set-attribute/)
        * [ Simple Exec ](../../ref/simple-exec/)
        * [ Signal ](../../ref/signal/)
        * [ Subtract ](../../ref/subtract/)
        * [ Take ](../../ref/take/)
        * [ Tok ](../../ref/tok/)
        * [ Update ](../../ref/update/)
        * [ Vector Conditional ](../../ref/vector-conditional/)
      * Control constructs  Control constructs 
        * [ Cond ](../../ref/cond/)
        * [ do ](../../ref/do/)
        * [ if ](../../ref/if/)
        * [ while ](../../ref/while/)
      * Namespaces  Namespaces 
        * [ .h (markup) ](../../ref/doth/)
        * [ .j (JSON) ](../../ref/dotj/)
        * [ .m (modules) ](../../ref/dotm/)
        * [ .Q (utils) ](../../ref/dotq/)
        * [ .z (env, callbacks) ](../../ref/dotz/)
      * [ Application ](../../basics/application/)
      * [ Atomic functions ](../../basics/atomic/)
      * [ Comparison ](../../basics/comparison/)
      * [ Conformability ](../../basics/conformable/)
      * [ Connection handles ](../../basics/handles/)
      * [ Command-line options ](../../basics/cmdline/)
      * [ Datatypes ](../../basics/datatypes/)
      * [ Dictionaries ](../../basics/dictsandtables/)
      * [ Enumerations ](../../basics/enumerations/)
      * [ Evaluation control ](../../basics/control/)
      * [ Exposed infrastructure ](../../basics/exposed-infrastructure/)
      * [ File system ](../../basics/files/)
      * [ Function notation ](../../basics/function-notation/)
      * [ Glossary ](../../basics/glossary/)
      * [ Internal functions ](../../basics/internal/)
      * [ Joins ](../../basics/joins/)
      * [ Mathematics ](../../basics/math/)
      * [ Metadata ](../../basics/metadata/)
      * [ Namespaces ](../../basics/namespaces/)
      * [ Pattern matching ](../../basics/pattern/)
      * [ Parse trees ](../../basics/parsetrees/)
      * qSQL  qSQL 
        * [ qSQL queries ](../../basics/qsql/)
        * [ Functional qSQL ](../../basics/funsql/)
      * [ Regular Expressions ](../../basics/regex/)
      * [ Syntax ](../../basics/syntax/)
      * [ System commands ](../../basics/syscmds/)
      * [ Tables ](../../kb/faq/)
      * [ Variadic syntax ](../../basics/variadic/)
    * Database  Database 
      * [ Tables in the filesystem ](../../database/)
      * Populating tables  Populating tables 
        * [ Loading from large files ](../../kb/loading-from-large-files/)
        * [ Foreign keys ](../foreign-keys/)
        * [ Linking columns ](../../kb/linking-columns/)
        * [ Data loaders ](../data-loaders/)
        * [ From MDB via ODBC ](../../database/mdb-odbc/)
      * Persisting tables  Persisting tables 
        * [ Serializing an object ](../../database/object/)
        * [ Splayed tables ](../../kb/splayed-tables/)
        * [ Partitioned tables ](../../kb/partition/)
        * [ Segmented databases ](../../database/segment/)
        * [ Multiple partitions ](../multi-partitioned-dbs/)
      * Maintenance  Maintenance 
        * [ Data management ](../data-management/)
        * [ Data-At-Rest Encryption ](../../kb/dare/)
        * Compression  Compression 
          * [ File compression ](../../kb/file-compression/)
          * [ Compression examples ](../compress/)
          * [ FSI case study ](../../kb/compression/fsicasestudy/)
        * [ Permissions ](../permissions/)
        * [ Query optimization ](../columnar-database/)
        * [ Query scaling ](../query-scaling/)
        * [ Time-series simplification ](../ts-shrink/)
        * [ Compacting HDB sym ](../../kb/compacting-hdb-sym/)
        * [ Working with sym files ](../symfiles/)
    * Developing  Developing 
      * IPC  IPC 
        * [ Overview ](../../basics/ipc/)
        * [ Listening port ](../../basics/listening-port/)
        * [ Deferred response ](../../kb/deferred-response/)
        * [ Async callbacks ](../../kb/callbacks/)
        * [ Named pipes ](../../kb/named-pipes/)
        * [ Serialization examples ](../../kb/serialization/)
        * [ Socket sharding ](../socket-sharding/)
        * [ SSL/TLS ](../../kb/ssl/)
        * [ HTTP ](../../kb/http/)
        * [ WebSockets ](../../kb/websockets/)
      * Tools  Tools 
        * [ Code profiler ](../../kb/profiler/)
        * [ Debugging ](../../basics/debug/)
        * [ Errors ](../../basics/errors/)
        * [ man.q ](../../about/man/)
        * [ Unit tests ](../../kb/unit-tests/)
        * [ Monitor & control execution ](../../kb/using-dotz/)
      * Coding  Coding 
        * [ Geospatial indexing ](../../kb/geospatial/)
        * [ Linear programming ](../../kb/lp/)
        * [ Multithreaded primitives ](../../kb/mt-primitives/)
        * [ Pivoting tables ](../../kb/pivoting-tables/)
        * [ Precision ](../../basics/precision/)
        * [ Programming examples ](../../kb/programming-examples/)
        * [ Programming idioms ](../../kb/programming-idioms/)
        * [ Temporal data ](../../kb/temporal-data/)
        * [ Timezones ](../../kb/timezones/)
        * [ Unicode ](../../kb/unicode/)
      * DevOps  DevOps 
        * [ CPU affinity ](../../kb/cpu-affinity/)
        * [ Daemon ](../../kb/daemon/)
        * [ Firewalling ](../../kb/firewalling/)
        * [ inetd, xinetd ](../../kb/inetd/)
        * [ Linux production notes ](../../kb/linux-production/)
        * [ File system comparison ](../../kb/filesystemTestByNano/)
        * [ Log Files ](../../kb/logging/)
        * [ Multi-threading ](../multi-thread/)
        * [ Multiple versions ](../../kb/versions/)
        * [ Parallel processing ](../../basics/peach/)
        * [ Performance tips ](../../kb/performance-tips/)
        * [ Shebang script ](../../develop/shebang/)
        * [ Surveillance latency ](../surveillance-latency/)
        * [ Windows service ](../../kb/windows-service/)
      * Release notes  Release notes 
        * [ History ](../../releases/)
        * [ Changes in 4.1 ](../../releases/ChangesIn4.1/)
        * [ Changes in 4.0 ](../../releases/ChangesIn4.0/)
        * [ Changes in 3.6 ](../../releases/ChangesIn3.6/)
        * [ Changes in 3.5 ](../../releases/ChangesIn3.5/)
        * [ Changes in 3.4 ](../../releases/ChangesIn3.4/)
        * [ Changes in 3.3 ](../../releases/ChangesIn3.3/)
        * [ Changes in 3.2 ](../../releases/ChangesIn3.2/)
        * [ Changes in 3.1 ](../../releases/ChangesIn3.1/)
        * [ Changes in 3.0 ](../../releases/ChangesIn3.0/)
        * [ Changes in 2.8 ](../../releases/ChangesIn2.8/)
        * [ Changes in 2.7 ](../../releases/ChangesIn2.7/)
        * [ Changes in 2.6 ](../../releases/ChangesIn2.6/)
        * [ Changes in 2.5 ](../../releases/ChangesIn2.5/)
        * [ Changes in 2.4 ](../../releases/ChangesIn2.4/)
        * [ Withdrawn ](../../releases/withdrawn/)
      * [ Developer tools ](../../devtools/)
      * [ FAQ ](../../kb/faq-listbox/)
    * Streaming  Streaming 
      * General architecture  General architecture 
        * [ Overview ](../../architecture/)
        * kdb+tick  kdb+tick 
          * [ Tickerplant (tick.q) ](../../architecture/tickq/)
          * [ Tickerplant pub/sub (u.q) ](../../architecture/uq/)
          * [ RDB (r.q) ](../../architecture/rq/)
      * [ Alternative architecture ](../../kb/kdb-tick/)
      * [ TP Log (data recovery) ](../data-recovery/)
      * [ RTEs (real-time engines) ](../rt-tick/)
      * [ Gateway design ](../gateway-design/)
      * [ Query routing ](../query-routing/)
      * [ Load balancing ](../../kb/load-balancing/)
      * [ Profiling ](../tick-profiling/)
      * [ Disaster recovery ](../disaster-recovery/)
      * [ Kubernetes ](https://youtu.be/jqtkkCqBvr4)
      * [ Order Book ](../order-book/)
      * [ Alternative in-memory layouts ](../../kb/alternative-in-memory-layouts/)
      * [ Corporate actions ](../../kb/corporate-actions/)
      * Advanced  Advanced 
        * [ Distributed systems ](../query-interface/)
        * [ RDB intraday writedown ](../intraday-writedown/)
    * Interfaces  Interfaces 
      * Languages  Languages 
        * C/C++  C/C++ 
          * [ Quick guide ](../../interfaces/c-client-for-q/)
          * [ API reference ](../../interfaces/capiref/)
          * [ C API for kdb+ ](../capi/)
          * [ Extending q with C/C++ ](../../interfaces/using-c-functions/)
          * [ Async callbacks (C client) ](../../kb/server-calling-client/)
        * [ C# ](../../interfaces/csharp/)
        * [ Foreign Function Interface (FFI) ](../../interfaces/ffi/)
        * [ Java ](../../interfaces/java/)
        * [ Python ](../../interfaces/python/)
        * [ R ](../../interfaces/r/)
        * [ Rust ](../../interfaces/rust/)
        * [ Scala ](../../interfaces/scala-client-for-q/)
      * [ KX libraries ](../../interfaces/)
      * [ Bloomberg ](../../interfaces/q-client-for-bloomberg/)
      * [ Excel ](../../interfaces/excel-client-for-q/)
      * [ FIX messaging ](../fix-messaging/)
      * [ GPUs ](../../interfaces/gpus/)
      * [ Matlab ](../../interfaces/matlab-client-for-q/)
      * ODBC  ODBC 
        * [ ODBC client ](../../interfaces/q-client-for-odbc/)
        * [ ODBC3 server ](../../interfaces/q-server-for-odbc3/)
        * [ ODBC3 and Tableau ](../data-visualization/)
      * [ Solace pub/sub ](../solace/)
      * [ Open source ](../../github/)
      * [ Machine learning ](../../ml/)
    * Using kdb+ in the cloud  Using kdb+ in the cloud 
      * [ About ](../../cloud/)
      * Amazon Web Services  Amazon Web Services 
        * [ Reference architecture ](../../cloud/aws/)
        * Amazon EC2 & Storage Services  Amazon EC2 & Storage Services 
          * [ Migrating a kdb+ HDB to Amazon EC2 ](../../cloud/aws/migration/)
          * [ Elastic Block Store (EBS) ](../../cloud/aws/app-a-ebs/)
          * [ EFS (NFS) ](../../cloud/aws/app-b-efs-nfs/)
          * [ Amazon Storage Gateway ](../../cloud/aws/app-c-asg/)
          * [ FSx for Lustre ](../../cloud/aws/lustre/)
        * [ AWS Lambda ](../../cloud/aws-lambda/)
      * Microsoft Azure  Microsoft Azure 
        * [ Reference architecture ](../../cloud/azure/architecture/)
      * Google Cloud  Google Cloud 
        * [ Reference architecture ](../../cloud/gcpm/architecture/)
      * Auto Scaling  Auto Scaling 
        * [ About ](../../cloud/autoscale/)
        * [ Amazon Web Services ](../../cloud/autoscale/aws/)
        * [ Realtime data cluster ](../../cloud/autoscale/rdc/)
        * [ Costs and risks ](../../cloud/autoscale/cost-risk/)
      * Other file systems  Other file systems 
        * [ MapR-FS ](../../cloud/otherfs/mapr/)
        * [ Goofys ](../../cloud/otherfs/goofys/)
        * [ S3FS ](../../cloud/otherfs/s3fs/)
        * [ S3QL ](../../cloud/otherfs/s3ql/)
        * [ ObjectiveFS ](../../cloud/otherfs/objectivefs/)
        * [ WekaIO Matrix ](../../cloud/otherfs/wekaio-matrix/)
        * [ Quobyte ](../../cloud/otherfs/quobyte/)
    * [ Academy ](https://learninghub.kx.com)
    * [ Discussion Forum ](https://learninghub.kx.com/forums/forum/kdb)
    * [ White papers ](../)
    * [ About this site ](../../about/thissite/)
  * [ kdb Insights SDK ](https://code.kx.com/insights)
  * [ kdb Insights Enterprise ](https://code.kx.com/insights/enterprise)
  * [ KDB.AI ](https://code.kx.com/kdbai)
  * [ PyKX ](https://code.kx.com/pykx)
  * [ APIs ](https://code.kx.com/insights/api)
  * [ Help ](https://code.kx.com/home/support.html)



On this page 

  * Motivation 
  * The data 
  * The building blocks 
    * Analytic setup 
    * Data filtering 
  * Reference data 
  * Consolidating the data 
    * Extending parameters 
    * Multi-market aggregation 
  * Conclusion 
  * Author 
  * Appendix 



# Market Fragmentation:  
A kdb+ framework for multiple liquidity sources¶

by James Corcoran

kdb+ plays a large part in the trading and risk management activities of many financial institutions around the world. For a large-scale kdb+ system to be effective, it must be designed efficiently so that it can capture and store colossal amounts of data. However, it is equally important that the system provides intelligent and useful functionality to end-users. In the financial world, increasing participation and advances in technology are resulting in a progressively more fragmented market with liquidity being spread across many trading venues. It is therefore crucial that a kdb+ system can store and analyze information for a financial security from all available sources. This paper presents an approach to the challenge of consolidating share price information for equities that trade on multiple venues.

### Motivation¶

Since the inception of the Markets in Financial Instruments Directive (MiFID), Multilateral Trading Facilities (MTFs) have sprung up across Europe. Alternative Trading Systems are the US equivalent. Prior to the MiFID, trading typically took place on national exchanges. Other types of trading venues in existence today include crossing networks and dark pools. All of these venues compete with each other for trading activity.

For market participants, the increased fragmentation forces more sophisticated trading strategies in the form of smart order routing. A number of additional factors add to the argument that the ability to consolidate real-time market data in-house can add real value to the trading strategies of an organization, including technical glitches at exchanges which can lead to suboptimal pricing and order matching.

![Breakdown of traded volume](img/figure1.png)  
_Breakdown of the traded volume that occurred on the main trading venues for all EMEA equities in December 2012_

Bearing in mind that the data output by a kdb+ system can often be the input into a trading decision or algorithm, the timely and accurate provision of consolidated real-time information for a security is vital.

### The data¶

The goal for kdb+ financial engineers is to analyze various aspects of a stockâs trading activity at each of the venues where it trades. In the equities world, real-time market data vendors provide trade, level-1 and level-2 quote feeds for these venues. Securities trading on different venues will use a different suffix, enabling data consumers to differentiate between venues â for example, Vodafone shares traded on the LSE are reported by Reuters on the Reuters Instrument Code (RIC) VOD.L, whereas shares of the same company traded on Chi-X are recorded on VODl.CHI. In the FX world, we might have feed handlers connecting directly to each ECN. The symbol column in our table would generally be a currency pair and we might use a venue column to differentiate between venues. Regardless of the asset class, in order to get a complete picture of a securityâs trading activity the kdb+ system must collect and store data for all venues for a given security.

For the purposes of this paper, we will assume standard equity trade and quote tables. We will use Reuters cash equities market data in the examples and assume that our feed handler subscribes to the RICs we require and that it publishes the data to our tickerplant.

## The building blocks¶

In order to be able to effectively analyze and consolidate data for securities from multiple venues, we must first have in place an efficient mechanism for retrieving data for securities from a single venue. Here we introduce the concept of a gateway: a kdb+ process that acts as a connection point for end- users. The gatewayâs purpose is to:

  * accept client queries and/or calls made to analytic functions;

  * to dispatch appropriate requests for data to the RDB or HDB, or both; and

  * to return the data to the client.




Data retrieval, data filtering, computation and aggregation, as a general rule, should all be done on the database. The gateway, once it has retrieved this data, can enrich it. Examples of enrichment are time zone conversion, currency conversion and adjustment for corporate actions. We make the case below that consolidation at the security level is also best done in the gateway.

### Analytic setup¶

A typical analytic that end-users might wish kdb+ to provide is an interval function, which gives a range of different analytic aggregations for a stock or list of stocks based on data between a given start time and end time on a given date, or for a range of dates. For simplicity, we will focus on analytics which span just one date. Let us call this function `getIntervalData`. Examples of the analytics the function could provide are `open`, `high`, `low`, `close`, `volume`, `avgprice`, `vwap`, `twap`, `meanspread`, `spreadvolatility`, `range`, and `lastmidprice`. We would like the ability to pass many different parameters into this function, and potentially more than the maximum number allowed in q, which is 8. We may also decide that some parameters are optional and need not be specified by the user. For these reasons we will use a dictionary as the single parameter, rather than defining a function signature with a specific number of arguments. A typical parameter dictionary looks like the following:
    
    
    q)params
    symList  | `VOD.L
    date     | 2013.01.15
    startTime| 08:30
    endTime  | 09:30
    columns  | `vwap`volume

### Data filtering¶

It is important at this stage that we introduce the notion of data filtering. Trading venues have a regulatory requirement to report all trades whether they have been executed electronically or over the counter. Not all trades, particularly in the equities world, should be included in all calculations. For example, one user may want only lit order book trades to be included in his/her VWAP calculation, but another user may prefer that all order book trades appear in the VWAP figure.

![Monthly breakdown of traded volume](img/figure2.png)  
_Monthly breakdown of on-and-off order book traded volume across all EMEA equity markets for the year to December 2012_

Market data vendors break trade data into different categories including auction, lit, hidden and dark order book, off order book and OTC trades, and use data qualifier flags to indicate which category each trade falls into. Data should be filtered based on these qualifiers, and according to the end-userâs requirements, prior to aggregation. This specification should be configurable within the parameter dictionary passed to the `getIntervalData` function.

The various qualifier flags used to filter the raw data for each category can be held in configuration files on a per-venue basis and loaded into memory in the database so that filtering can be done during execution of the query by the use of a simple utility function. An approach to storing this configuration is outlined as follows:

Define a dictionary, `.cfg.filterrules`, keyed by filtering rule, e.g. Order Book, Total Market, Dark Trades, where the corresponding values are tables holding the valid qualifier flags for each venue for that rule.
    
    
    q).cfg.filterrules
    TM | (+(,`venue)!,`LSE`BAT`CHI`TOR)!+(,`qualifier)!,(`A`Auc`B`C`X`DARKTRADE`m..
    OB | (+(,`venue)!,`LSE`BAT`CHI`TOR)!+(,`qualifier)!,(`A`Auc`B`C`m;`A`AUC`OB`C..
    DRK| (+(,`venue)!,`LSE`BAT`CHI`TOR)!+(,`qualifier)!,(,`DARKTRADE;,`DARK;,`DRK..
    
    q).cfg.filterrules[`OB]
    venue| qualifier
    ---- | --------------
    LSE  | `A`Auc`B`C`m
    BAT  | `A`AUC`OB`C
    CHI  | `a`b`auc`ob
    TOR  | `A`Auc`X`Y`OB

Assuming we have access to a params dictionary, we could then construct our query as follows.
    
    
    select vwap:wavg[size;price], volume:sum[size] by sym from trade 
           where date=params[`date],
                 sym in params[`symList],
                 time within (params`startTime;params`endTime), 
                 .util.validTrade[sym;qualifier;params`filterRule]

`.util.validTrade` makes use of the above config data, returning a Boolean indicating whether the qualifier flag for a given record is valid for that recordâs sym according to the given filter rule. Due to the fact that we have defined valid qualifiers on a per-rule per-venue basis, we will of course require a method for retrieving a venue for a given sym. This is best done through the use of a simple dictionary lookup.
    
    
    q).cfg.symVenue
    BARCl.BS | BAT
    BARCl.CHI| CHI
    BARC.L   | LSE
    BARC.TQ  | TOR
    VODl.BS  | BAT
    VODl.CHI | CHI
    VOD.L    | LSE
    VODl.TQ  | TOR
    
    q).util.getVenue[`VOD.L`BARC.BS] 
    `LSE`BAT

This data processing takes place on the RDB and/or HDB. The query will have been dispatched, with parameters, by the gateway. We will now demonstrate a mechanism for consolidating data from multiple venues by passing an additional `multiMarketRule` parameter in our call to the gateway.

## Reference data¶

Having already briefly touched on it, we introduce the role of reference data more formally here. With the above analytic setup, we retrieve data only for the given symbol/s passed to the function. When a multimarket parameter is included however, we need to retrieve and aggregate data for all instrument codes associated with the entity mapped to the sym parameter.

With that in mind, the first thing we need to have is the ability to look up the venues on which a given stock trades. We also need to know the instrument codes used for the stock in question on each venue. As described above (_The data_), these will differ. This information is usually located in a reference data system. The reference data system could be a component of the kdb+ system or it could be an external application within the bank. Regardless of where the reference data is sourced, it should be processed and loaded into memory at least once per day. The databases (RDB and HDB) require access to the reference data, as does the gateway. In terms of size, reference data should be a very small fraction of the size of market data, so memory overhead will be minimal.

The most effective layout is to have a table keyed on sym, mapping each sym in our stock universe to its primary sym. By primary sym, we mean the instrument code of the company for the primary venue on which it trades. For example, VOD.Lâs is simply VOD.L since Vodafoneâs primary venue is the LSE whereas VODl.CHIâs is also VOD.L.
    
    
    q).cfg.multiMarketMap
    sym      | primarysym venue
    ---------| ----------------
    BARCl.BS | BARC.L     BAT
    BARCl.CHI| BARC.L     CHI
    BARC.L   | BARC.L     LSE
    BARC.TQ  | BARC.L     TOR
    VODl.BS  | VOD.L      BAT
    VODl.CHI | VOD.L      CHI
    VOD.L    | VOD.L      LSE
    VODl.TQ  | VOD.L      TOR

## Consolidating the data¶

### Extending parameters¶

Providing a consolidated analytic for a stock requires that we query the database for all syms associated with an entity. With the above reference data at our disposal, we can now write a utility function,

`.util.extendSymsForMultiMarket`, which will expand the list of syms passed into the params dictionary. This function should be called if and only if a `multiMarketRule` parameter is passed. We should also be careful to preserve the original sym list passed to us, as we will aggregate back up to it during the final consolidation step. The following is an implementation of such a utility function:
    
    
    .util.extendSymsForMultiMarket:{[symList] 
        distinct raze {update origSymList:x from
                       select symList:sym from .cfg.multiMarketMap
                       where primarysym in .cfg.multiMarketMap[x]`primarysym 
                       } each (),symList
        }
    
    
    q).util.extendSymsForMultiMarket[`BARC.L`VOD.L] 
    symList   origSymList
    ---------------------
    BARCl.BS  BARC.L
    BARCl.CHI BARC.L
    BARC.L    BARC.L
    BARC.TQ   BARC.L
    VODl.BS   VOD.L
    VODl.CHI  VOD.L
    VOD.L     VOD.L
    VODl.TQ   VOD.L

We can now use this utility in the `getIntervalData` function defined on the gateway so that we dispatch our query to the database with an extended `symList`, as follows:
    
    
    if[params[`multiMarketRule]~`multi; 
        extended_syms:.util.extendSymsForMultiMarket[params`symList]; 
        params:@[params;`symList;:;extended_syms`symList];
      ];

Once we have adjusted the parameters appropriately, we can dispatch the query to the database/s in the exact same manner as before. The only thing that happens differently is that we are querying for additional syms in our `symList`. This will naturally result in a slightly more expensive query.

### Multi-market aggregation¶

The final, and arguably the most critical step in consolidating the data is to aggregate our analytics at the entity level as opposed to the sym level.

Having dispatched the query to the database/s, we now have our data held in memory in the gateway, aggregated on a per-sym basis. Assuming that all venues are trading in the same currency, all that remains is for us to aggregate this data further up to primary sym level. We will use configuration data to define multi-market aggregation rules.

Multiple currencies

If the various venues trade in different currencies, we would invoke a function in the gateway to convert all data to a common currency prior to aggregation. This method assumes that FX risk is hedged during the lifetime of the trade.

The method of aggregation is dependent on the analytic in question. Therefore it is necessary for us to define these rules in a q file. For a volume analytic, the consolidated volume is simply the sum of the volume on all venues. The consolidated figure for a maximum or minimum analytic will be the maximum or minimum of all data points. We need to do a little more work however for a weighted-average analytic such as a VWAP. Given a set of VWAPs and volumes for a stock from different venues, the consolidated VWAP is given by the formula:

( Σ _vwap_ Ã _volume_ ) ÷ ( Σ _volume_ )

It is evident that we need access to the venue volumes as well as the individually-calculated VWAPs in order to weight the consolidated analytic correctly. This means that when a consolidated VWAP is requested, we need the query to return a `volume` column as well as a `vwap` column to our gateway.

Similarly, for a consolidated real-time snapshot of the midprice (letâs call it `lastmidprice`), rather than working out the mid price for a stock on each venue, we need to return the last bid price and the last ask price on each venue. We then take the maximum of the bid prices and the minimum of the ask prices. This represents the tightest spread available and from there we can work out a meaningful consolidated mid price.

The knowledge required for additional column retrieval could be implemented in a utility function, `.util.extendExtraColParams`, prior to the query dispatch.

Here we present a list of consolidation rules for a few common analytics.
    
    
    .cfg.multiMarketAgg:()!();
    .cfg.multiMarketAgg[`volume]:"sum volume" 
    .cfg.multiMarketAgg[`vwap]:"wavg[volume;vwap]" 
    .cfg.multiMarketAgg[`range]: "(max maxprice)-(min minprice)" 
    .cfg.multiMarketAgg[`tickcount]:"sum tickcount" 
    .cfg.multiMarketAgg[`maxbid]:"max maxbid" 
    .cfg.multiMarketAgg[`minask]:"min minask" 
    .cfg.multiMarketAgg[`lastmidprice]:"((max lastbid)+(min lastask))%2"

With the consolidation rules defined, we can use them in the final aggregation before presenting the data. The un-aggregated data is presented as follows.
    
    
    q)res
    sym       volume    vwap    maxprice minprice lastbid lastask 
    --------------------------------------------------------------
    BARCl.BS  5202383   244.05  244.25   243.85   244      244.1
    BARCl.CHI 5847878   244.1   244.3    243.9    244.05   244.15
    BARC.L    30283638  244.1   244.3    243.9    244.05   244.15
    BARC.TQ   3928294   244.15  244.35   243.95   244.1    244.2
    VODl.BS   10342910  160.9   161.245  159.85   160.895  160.9
    VODl.CHI  10383645  160.9   161.5    159.85   160.89   160.895
    VOD.L     108378262 160.895 161.245  159.9    160.895  160.9
    VODl.TQ   10252838  160.895 161.245  159.89   160.895  160.9

We can now aggregate it through the use of a clever functional select, utilizing qâs parse feature to bring our configured aggregation rules into play. First, left-join the original user-passed `symList` back to the results table. This is the entity that we want to roll up to.
    
    
    res:lj[res;`sym xkey select sym:symList, origSymList from extended_syms]

The result of this gives us the following table.
    
    
    sym       volume    vwap    maxprice minprice lastbid  lastask origSymList 
    --------------------------------------------------------------------------
    BARCl.BS  5202383   244.05  244.25   243.85   244      244.15  BARC.L
    BARCl.CHI 5847878   244.1   244.3    243.9    244.05   244.15  BARC.L
    BARC.L    30283638  244.1   244.3    243.9    244.05   244.15  BARC.L
    BARC.TQ   3928294   244.15  244.35   243.95   244.1    244.2   BARC.L
    VODl.BS   10342910  161.195 161.245  159.85   161.195  161.205 VOD.L
    VODl.CHI  10383645  161.19  161.25   159.85   161.195  161.21  VOD.L
    VOD.L     108378262 161.195 161.245  159.9    161.195  161.205 VOD.L
    VODl.TQ   10252838  161.195 161.245  159.9    161.195  161.205 VOD.L

The final step is to aggregate by the originally supplied user `symList`.
    
    
    / aggregate by origSymList and rename this column to sym
    byClause:(enlist`sym)!enlist`origSymList;

We then look up the multimarket rules for the columns we are interested in, use [`parse`](../../ref/parse/) to parse each string, and create a dictionary mapping each column name to its corresponding aggregation. This dictionary is required for the final parameter into the functional select.
    
    
    aggClause:columns!parse each .cfg.multiMarketAgg[columns:params`columns]

`aggClause` is thus defined as:
    
    
    volume      | (sum;`volume)
    vwap        | (wavg;`volume;`vwap)
    range       | (-;(max;`maxprice);(min;`minprice))
    lastmidprice| (%;(+;(max;`lastbid);(min;`lastask));2)

And our functional select is constructed as follows:
    
    
    res:0!?[res;();byClause;aggClause];

Giving the final result-set, the consolidated analytics, ready to be passed back to the user:
    
    
    sym    volume    vwap     range  lastmidprice 
    ---------------------------------------------
    BARC.L 45262193  244.0986 0.5.   244.125 
    VOD.L  139357655 161.1946 1.4    161.2

## Conclusion¶

This paper has described a methodology for analyzing data across multiple liquidity sources in kdb+. The goal was to show how we could aggregate tick data for a financial security from multiple data sources or trading venues. The main features of a simple analytics system were briefly outlined and we described how to dispatch vanilla analytic requests from the gateway. The concept of data filtering was then introduced and its importance when aggregating time series data was outlined. After this, we explained the role of reference data in a kdb+ system and how it fits into the analytic framework. Armed with the appropriate reference data and consolidation rules, we were then able to dispatch relevant requests to our databases and aggregate the results in the gateway in order to return consolidated analytics to the user.

The framework was provided in the context of an equities analytics system, but is extendable to other major asset classes as well as electronically traded derivatives. In FX, the ECNs provided by brokerages and banks act as the trading venues, and instead of using the symbol suffix to differentiate between venues, one can use a combination of the currency pair and the venue. Similarly in commodities, provided there is enough liquidity in the instrument, the same rules and framework can be applied. Other use cases, such as aggregating positions, risk and P&L from the desk or regional level to the department level, could be implemented using the same principles described in this paper.

A script is provided in the Appendix below so that users can work through the implementation described in this paper.

All tests were performed with kdb+ 3.0 (2012.09.26)

## Author¶

![James Corcoran](../../img/faces/jamescorcoran.jpg)

**James Corcoran** has worked as a kdb+ consultant in some of the worldâs largest financial institutions and has experience in implementing global software and data solutions in all major asset classes. He has delivered talks and presentations on various aspects of kdb+ and most recently spoke at the annual KX user conference in London. As a qualified professional risk manager he is also involved in various ongoing risk-management projects at KX.

## Appendix¶

The code in this appendix can be found on Github at  [kxcontrib/market-fragmentation](https://github.com/kxcontrib/market-fragmentation).
    
    
    ////////////////////////////////
    // Set up configuration data
    ////////////////////////////////
    
    .cfg.filterrules:()!();
     .cfg.filterrules[`TM]:([venue:`LSE`BAT`CHI`TOR]
                           qualifier:(
                              `A`Auc`B`C`X`DARKTRADE`m;
                              `A`AUC`B`c`x`D ARK;
                              `a`auc`b`c`x`DRK;
                              `A`Auc`B`C`X`DARKTRADE`m)
                            );
    .cfg.filterrules[`OB]:([venue:`LSE`BAT`CHI`TOR]
                           qualifier:(`A`Auc`B`C`m;`A`AUC`B`c;`a`auc`b`c;`A`A uc`B`C`m));
    .cfg.filterrules[`DRK]:([venue:`LSE`BAT`CHI`TOR]
                             qualifier:`DARKTRADE`DARK`DRK`DARKTRADE);
    
    .cfg.symVenue:()!();
    .cfg.symVenue[`BARCl.BS]:`BAT;
    .cfg.symVenue[`BARCl.CHI]:`CHI;
    .cfg.symVenue[`BARC.L]:`LSE;
    .cfg.symVenue[`BARC.TQ]:`TOR;
    .cfg.symVenue[`VODl.BS]:`BAT;
    .cfg.symVenue[`VODl.CHI]:`CHI;
    .cfg.symVenue[`VOD.L]:`LSE;
    .cfg.symVenue[`VODl.TQ]:`TOR;
    
    .cfg.multiMarketMap:(
      [sym:`BARCl.BS`BARCl.CHI`BARC.L`BARC.TQ`VODl.BS`VODl.CHI`VOD.L`VODl.TQ] 
      primarysym:`BARC.L`BARC.L`BARC.L`BARC.L`VOD.L`VOD.L`VOD.L`VOD.L;
      venue:`BAT`CHI`LSE`TOR`BAT`CHI`LSE`TOR);
    
    .cfg.multiMarketAgg:()!();
    .cfg.multiMarketAgg[`volume]:"sum volume"
    .cfg.multiMarketAgg[`vwap]:"wavg[volume;vwap]"
    .cfg.multiMarketAgg[`range]: "(max maxprice)-(min minprice)"
    .cfg.multiMarketAgg[`tickcount]:"sum tickcount"
    .cfg.multiMarketAgg[`maxbid]:"max maxbid"
    .cfg.multiMarketAgg[`minask]:"min minask"
    .cfg.multiMarketAgg[`lastmidprice]:"((max lastbid)+(min lastask))%2"
    
    .cfg.defaultParams:`startTime`endTime`filterRule`multiMarketRule!
                       (08:30;16:30;`OB;`none);
    
    
    ////////////////////////////////
    // Analytic functions
    ////////////////////////////////
    
    getIntervalData:{[params]
        -1"Running getIntervalData for params: ",-3!params;
        params:.util.applyDefaultParams[params]; 
        if[params[`multiMarketRule]~`multi;
            extended_syms:.util.extendSymsForMultiMarket[params`symList]; 
            params:@[params;`symList;:;extended\_syms`symList];
        ];
    
    res:select volume:sum[size], vwap:wavg[size;price], range:max[price]-min[price], 
               maxprice:max price, minprice:min price,
               maxbid:max bid, minask:min ask,
               lastbid:last bid, lastask:last ask, lastmidprice:(last[bid]+last[ask])%2 
        by sym from trade
        where date=params[`date],
              sym in params[`symList],
              time within (params`startTime;params`endTime),
              .util.validTrade[sym;qualifier;params`filterRule];
    
    if[params[`multiMarketRule]~`multi;
        res:lj[res;`sym xkey select sym:symList, origSymList from extended_syms]; 
        byClause:(enlist`sym)!enlist`origSymList;
        aggClause:columns!parse each .cfg.multiMarketAgg[columns:params`columns]; 
        res:0!?[res;();byClause;aggClause];
      ];
      :(`sym,params[`columns])\#0!res
    };
    
    
    ////////////////////////////////
    // Utilities
    ////////////////////////////////
    
    .util.applyDefaultParams:{[params]
        .cfg.defaultParams,params
        };
    
    .util.validTrade:{[sym;qualifier;rule] 
        venue:.cfg.symVenue[sym];
        validqualifiers:(.cfg.filterrules[rule]each venue)`qualifier; 
        first each qualifier in' validqualifiers
        };
    
    .util.extendSymsForMultiMarket:{[symList] 
        distinct raze {update origSymList:x from
                       select symList:sym from .cfg.multiMarketMap
                       where primarysym in .cfg.multiMarketMap[x]`primarysym
                       } each (),symList
        }
    
    
    ////////////////////////////////
    // Generate trade data
    ////////////////////////////////
    \P 6
    
    trade:([]date:`date$();sym:`$();time:`time$();price:`float$();size:`int$());
    
    pi:acos -1;
    / Box-muller from kx.com/q/stat.q
    nor:{$[x=2*n:x div 2;raze sqrt[-2*log n?1f]*/:(sin;cos)@\:(2*pi)*n?1f;-1_.z.s 1+x]} 
    
    generateRandomPrices:{[s0;n] 
        dt:1%365*1000;
        timesteps:n; 
        vol:.2; 
        mu:.01;
        randomnumbers:sums(timesteps;1)#(nor timesteps); 
        s:s0*exp[(dt*mu-xexp[vol;2]%2) + randomnumbers*vol*sqrt[dt]]; 
        raze s}
    
    n:1000;
    `trade insert (n#2013.01.15;
                   n?`BARCl.BS`BARCl.CHI`BARC.L`BARC.TQ; 
                   08:00:00.000+28800*til n;
                   generateRandomPrices[244;n]; 10*n?100000);
    
    `trade insert (n#2013.01.15;
                   n?`VODl.BS`VODl.CHI`VOD.L`VODl.TQ; 
                   08:00:00.000+28800*til n;
                   generateRandomPrices[161;n]; 
                   10*n?100000);
    
    / add dummy qualifiers
    trade:{
      update qualifier:1?.cfg.filterrules[`TM;.cfg.symVenue[sym]]`qualifier from x
      } each trade;
    
    / add dummy prevailing quotes 
    spread:0.01;
    update bid:price-0.5*spread, ask:price+0.5*spread from `trade;
    
    `time xasc `trade;
    
    
    ////////////////////////////////
    // Usage
    ////////////////////////////////
    
    params:`symList`date`startTime`endTime`columns!(
        `VOD.L`BARC.L; 
        2013.01.15;
        08:30;
        09:30;
        `volume`vwap`range`maxbid`minask`lastmidprice);
    
    / default, filterRule=orderbook & multiMarketRule=none 
    a:getIntervalData params;
    
    / change filterRule from 'orderbook' to 'total market' 
    b:getIntervalData @[params;`filterRule;:;`TM];
    
    / change multiMarketRule from 'none' to 'multi' to get consolidated analytics 
    c:getIntervalData @[params;`multiMarketRule;:;`multi];

Back to top 

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).  
Kx and kdb+ are registered trademarks of [Kx Systems, Inc.](https://kx.com), a subsidiary of [FD Technologies plc](https://www.fdtechnologies.com/). 

Made with [ Material for MkDocs ](https://squidfunk.github.io/mkdocs-material/)
