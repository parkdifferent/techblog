# techblog

## Scalability
* [Microservices and Orchestration](https://martinfowler.com/microservices/)
    * [Domain-Oriented Microservice Architecture at Uber](https://eng.uber.com/microservice-architecture/)


## 接口设计
* [简化跨微服务重用，API 标准化过程中的左移法](https://www.infoq.cn/article/swp38OvkFIhIJO2K4wUT)

* [微软REST API指南](https://github.com/Microsoft/api-guidelines)

* [谷歌API设计指南](https://cloud.google.com/apis/design/)

> 在业余项目里，为了开发出一致的 API，并遵循 API 开发的行业最佳实践，我经常参考这本[风格手册]
(http://apistylebook.com/design/guidelines/)

## system design link
* [design example](http://highscalability.com/blog/category/example)

## 分布式系统

* [分布式系统学习资料汇总](https://www.qtmuniao.com/2021/05/16/distributed-system-material/)
* [awesome-distributed-systems](https://github.com/theanalyst/awesome-distributed-systems)  
* [papers-we-love](https://github.com/papers-we-love/papers-we-love/tree/master/distributed_systems)

* [6.824 Schedule: Spring 2022](https://pdos.csail.mit.edu/6.824/schedule.html)
* [Department of Computer Science and Technology](https://www.cl.cam.ac.uk/teaching/2021/ConcDisSys/materials.html)
* [15-440: Distributed Systems Syllabus](https://www.cs.cmu.edu/~dga/15-440/S14/syllabus.html)
* [CS244b: Distributed Systems](http://www.scs.stanford.edu/20sp-cs244b/)

* [Notes on Distributed Systems for Young Bloods](https://www.somethingsimilar.com/2013/01/14/notes-on-distributed-systems-for-young-bloods/)
* [Distributed systems theory for the distributed systems engineer](https://www.the-paper-trail.org/post/2014-08-09-distributed-systems-theory-for-the-distributed-systems-engineer/)

* [Distributed Systems Reading Group](http://dsrg.pdos.csail.mit.edu/)
* [Readings in distributed systems](https://christophermeiklejohn.com/distributed/systems/2013/07/12/readings-in-distributed-systems.html)

* [Learning about distributed systems: where to start?](http://muratbuffalo.blogspot.com/2020/06/learning-about-distributed-systems.html)

* [Learning a technical subject](http://muratbuffalo.blogspot.com/2021/12/learning-technical-subject.html)

* [Read papers, Not too much, Mostly foundational ones](http://muratbuffalo.blogspot.com/2021/02/read-papers-not-too-much-mostly.html)

* [Foundational distributed systems papers](http://muratbuffalo.blogspot.com/2021/02/foundational-distributed-systems-papers.html)

* [An Introduction to Distributed Systems](https://github.com/aphyr/distsys-class)
## GO
* [Go语言](https://www.infoq.cn/minibook/kHgvXNRNW124to0AJFeG)

* [godropbox](https://github.com/dropbox/godropbox)


## DDD & 微服务

* [去哪儿旅行微服务架构实践](https://www.infoq.cn/article/0OOSDdvcwhu7DYCVukCR)

* [亚马逊云基础架构 16 年创新史](https://www.infoq.cn/article/qo7WWdh7xPzDm76MDh9j)

* [Domain-Oriented Microservice Architecture at Uber](https://eng.uber.com/microservice-architecture/)

* [Why We Leverage Multi-tenancy in Uber’s Microservice Architecture](https://www.uber.com/blog/multitenancy-microservice-architecture/)

* [Troubleshooting Kafka for 2000 Microservices at Wix](https://medium.com/wix-engineering/troubleshooting-kafka-for-2000-microservices-at-wix-986ee382fd1e)
  * [greyhound](https://github.com/wix/greyhound)
* [Optimizing Pinterest’s Data Ingestion Stack: Findings and Learnings](https://medium.com/@Pinterest_Engineering/optimizing-pinterests-data-ingestion-stack-findings-and-learnings-994fddb063bf)
* [Building Scalable Real Time Event Processing with Kafka and Flink￼](https://doordash.engineering/2022/08/02/building-scalable-real-time-event-processing-with-kafka-and-flink/)

## 网关
* [The Architecture of Uber’s API gateway](https://www.uber.com/blog/architecture-api-gateway/)
* [Scaling of Uber’s API gateway](https://www.uber.com/blog/scaling-api-gateway/)  
* [Designing Edge Gateway, Uber’s API Lifecycle Management Platform](https://www.uber.com/blog/gatewayuberapi/)
* [zuul](https://github.com/Netflix/zuul/wiki/Push-Messaging)
* [Apache ShenYu]
## 数据集成

* [Solving the data integration variety problem at scale, with Gobblin](https://engineering.linkedin.com/blog/2021/data-integration-library)
* [gobblin](https://github.com/apache/gobblin)
* [深度解析字节跳动开源数据集成引擎 BitSail](https://github.com/bytedance/)

* [Data Mesh — A Data Movement and Processing Platform](https://netflixtechblog.com/data-mesh-a-data-movement-and-processing-platform-netflix-1288bcab2873)
* [Flink CDC 如何加速海量数据的实时集成？](https://zhuanlan.zhihu.com/p/578930446)

## event-driven-architecture

* [Event-Driven Architecture for Java Developers](https://university.cockroachlabs.com/courses/course-v1:crl+event-driven-architecture-for-java-devs+self-paced/about?_ga=2.91804563.82748203.1668174594-2140806030.1668174594#)
* [CDC](https://www.cockroachlabs.com/docs/stable/change-data-capture-overview.html)
* [Message Queuing and the Database: Solving the Dual Write Problem](https://www.cockroachlabs.com/blog/message-queuing-database-kafka/#the-transactional-outbox-pattern)
* [DBLog: A Generic Change-Data-Capture Framework](https://netflixtechblog.com/dblog-a-generic-change-data-capture-framework-69351fb9099b)
* [How Airbnb, Uber, Netflix, Shopify, and Reddit Use Change Data Capture to Drive Breakthrough Advantages](https://www.arcion.io/blog/brands-using-change-data-capture)

* [Leveraging CockroachDB’s Change Feed for Real-Time Inventory Data Processing](https://doordash.engineering/2022/11/21/leveraging-cockroachdbs-change-feed-for-real-time-inventory-data-processing/)
## 注册中心

* [ShopeePay 自研云原生高可用服务注册中心实践](https://www.toutiao.com/article/7162341615380496903/?wid=1668867319739)
* 得物中间件注册中心演进-黄洪杨， 分享PPT

* [Architectural Principles for High Availability on Hyperforce](https://engineering.salesforce.com/architectural-principles-for-high-availability-on-hyperforce/)

## Databases

* [Awesome Database Learning](https://github.com/pingcap/awesome-database-learning)

* [The history of databases at Netflix & how they use CockroachDB](https://www.cockroachlabs.com/blog/netflix-at-cockroachdb/)

* [eBay’s Global Secondary Indexes](https://tech.ebayinc.com/engineering/ebays-global-secondary-indexes/)

* [https://github.com/Netflix/dynomite](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf)
* [Scaling Datastores at Slack with Vitess](https://slack.engineering/scaling-datastores-at-slack-with-vitess/)
  * [vitess](https://github.com/vitessio/vitess)

* [SingleStore’s Patented Universal Storage - Part 1](https://www.singlestore.com/blog/memsql-singlestore-then-there-was-one/)
* [SingleStore’s Patented Universal Storage - Part 2](https://www.singlestore.com/blog/memsql-singlestore-memsql-7-1-episode-2/)

* [Evolving Schemaless into a Distributed SQL Database](https://www.uber.com/blog/oakland/search/schemaless%20sql%20database/)

* 面向大规模商业系统的数据库设计和实践

* [MyRocks: A space- and write-optimized MySQL database](https://engineering.fb.com/2016/08/31/core-data/myrocks-a-space-and-write-optimized-mysql-database/)

* [深度干货！一篇Paper带您读懂HTAP | StoneDB学术分享会第①期](https://zhuanlan.zhihu.com/p/559365164)
* [爆肝整理5000字！HTAP的关键技术有哪些？| StoneDB学术分享会第三期](https://zhuanlan.zhihu.com/p/567185977)

* [B站万亿级数据库选型与架构设计实践]
* [[数据库技术]PolarDB-超火的云原生](https://zhuanlan.zhihu.com/p/477801513)

## Workflow
* [Orchestrating Data/ML Workflows at Scale With Netflix Maestro](https://netflixtechblog.com/orchestrating-data-ml-workflows-at-scale-with-netflix-maestro-aaa2b41b800c)

* [Conductor](https://conductor.netflix.com/)
* [Conducting Better Business with Uber’s Open Source Orchestration Tool, Cadence](https://www.uber.com/blog/open-source-orchestration-tool-cadence-overview/)  
* [Cadence Multi-Tenant Task Processing](https://www.uber.com/blog/cadence-multi-tenant-task-processing/)  
* [Orchestrating Data/ML Workflows at Scale With Netflix Maestro](https://netflixtechblog.com/orchestrating-data-ml-workflows-at-scale-with-netflix-maestro-aaa2b41b800c)
* [Workflow Comparison: Uber Cadence vs Netflix Conductor](https://www.instaclustr.com/blog/workflow-comparison-uber-cadence-vs-netflix-conductor/)
* [Enabling Faster Financial Partnership Integrations Using Cadence](https://doordash.engineering/2022/05/18/enabling-faster-financial-partnership-integrations-using-cadence/)


## Observability

* [5 Design Patterns for Building Observable Services](https://engineering.salesforce.com/5-design-patterns-for-building-observable-services-d56e7a330419/)

* [Tag: Observability](https://engineering.salesforce.com/tagged/observability/)
* [Introduction to Netflix Servo](https://www.baeldung.com/netflix-servo)
* [Monitoring server applications with Vortex](https://dropbox.tech/infrastructure/monitoring-server-applications-with-vortex) 


## Data Migration
* [What is Data Migration? Guide to Data Migration Solutions and Planning](https://www.arcion.io/blog/data-migration-solutions)
* [Creating High-Quality Staging Data with a NoSQL Data Migration System](https://tech.ebayinc.com/engineering/creating-high-quality-staging-data-with-a-nosql-data-migration-system/)
* [Making our user storage more scalable and secure](https://www.intercom.com/blog/updating-our-user-storage/)

* [Delta: A highly available, strongly consistent storage service using chain replication](https://engineering.fb.com/2022/05/04/data-infrastructure/delta/)
* [Delta: A Data Synchronization and Enrichment Platform](https://netflixtechblog.com/delta-a-data-synchronization-and-enrichment-platform-e82c36a79aee)

* [Online Data Migration from HBase to TiDB with Zero Downtime](https://medium.com/pinterest-engineering/online-data-migration-from-hbase-to-tidb-with-zero-downtime-43f0fb474b84)

* [New Recruiter & Jobs: The largest enterprise data migration at LinkedIn](https://engineering.linkedin.com/blog/2021/new-recruiter---jobs--the-largest-enterprise-data-migration-at-l)

* [How Uber Migrated Financial Data from DynamoDB to Docstore](https://www.uber.com/blog/dynamodb-to-docstore-migration/)

* [Building Uber’s Fulfillment Platform for Planet-Scale using Google Cloud Spanner](https://www.uber.com/blog/building-ubers-fulfillment-platform/)
  * [Uber’s Fulfillment Platform: Ground-up Re-architecture to Accelerate Uber’s Go/Get Strategy](https://www.uber.com/blog/fulfillment-platform-rearchitecture/)
  * [The third article ]()
  * [the fourth article]() 

* [Revolutionizing Money Movements at Scale with Strong Data Consistency](https://www.uber.com/blog/money-scale-strong-data/)

* [Bulldozer: Batch Data Moving from Data Warehouse to Online Key-Value Stores](https://netflixtechblog.com/bulldozer-batch-data-moving-from-data-warehouse-to-online-key-value-stores-41bac13863f8)
  * [Machine Learning meets Databases](https://conf.slac.stanford.edu/xldb2018/sites/xldb2018.conf.slac.stanford.edu/files/Tues_11.15_IoannisPapa-Netflix-2018.pdf)

* [Migrating Facebook to MySQL 8.0](https://engineering.fb.com/2021/07/22/data-infrastructure/mysql/)

## Big Data
* [flink](https://tech.ebayinc.com/engineering/an-introduction-to-apache-flink/)
* [Unified Flink Source at Pinterest: Streaming Data Processing](https://medium.com/pinterest-engineering/unified-flink-source-at-pinterest-streaming-data-processing-c9d4e89f2ed6)


* [Open Sourcing Venice – LinkedIn’s Derived Data Platform](https://engineering.linkedin.com/blog/2022/open-sourcing-venice--linkedin-s-derived-data-platform)
* [Real-time analytics on network flow data with Apache Pinot](https://engineering.linkedin.com/blog/2022/real-time-analytics-on-network-flow-data-with-apache-pinot)
  * [pinot](https://github.com/apache/pinot)
  * [Pinot Real-Time Ingestion with Cloud Segment Storage](https://www.uber.com/blog/pinot-real-time-ingestion/)
  * [Real-Time Exactly-Once Ad Event Processing with Apache Flink, Kafka, and Pinot](https://www.uber.com/blog/real-time-exactly-once-ad-event-processing/)

* [Nebula as a Storage Platform to Build Airbnb’s Search Backends](https://medium.com/airbnb-engineering/nebula-as-a-storage-platform-to-build-airbnbs-search-backends-ecc577b05f06)
* [Learnings from Snowflake and Aurora: Separating Storage and Compute for Transaction and Analytics](https://www.singlestore.com/blog/separating-storage-and-compute-for-transaction-and-analytics/)

* [Jellyfish: Cost-Effective Data Tiering for Uber’s Largest Storage System](https://www.uber.com/blog/jellyfish-cost-effective-data-tiering/)

* [Open Sourcing Mantis: A Platform For Building Cost-Effective, Realtime, Operations-Focused Applications](https://netflixtechblog.com/open-sourcing-mantis-a-platform-for-building-cost-effective-realtime-operations-focused-5b8ff387813a)
  * [mantis](https://github.com/netflix/mantis)
  * [Building Netflix’s Distributed Tracing Infrastructure](https://netflixtechblog.com/building-netflixs-distributed-tracing-infrastructure-bb856c319304)

* [Scaling Cache Infrastructure at Pinterest](https://medium.com/pinterest-engineering/scaling-cache-infrastructure-at-pinterest-422d6d294ece)
* [Building scalable near-real time indexing on HBase](https://medium.com/pinterest-engineering/building-scalable-near-real-time-indexing-on-hbase-7b5eeb411888)

* [Paper Notes: Real-time Data Infrastructure at Uber](https://distributed-computing-musings.com/2022/08/paper-notes-real-time-data-infrastructure-at-uber/)
  * [Real-time Data Infrastructure at Uber](https://arxiv.org/abs/2104.00087)

* [Large Scale Ad Data Systems at Booking.com using the Public Cloud](https://medium.com/booking-com-development/large-scale-ad-data-systems-at-booking-com-using-the-public-cloud-c2f2c49fa7f2)
## 延迟服务 & 优先级队列

* [Dynein: Building an Open-source Distributed Delayed Job Queueing System](https://medium.com/airbnb-engineering/dynein-building-a-distributed-delayed-job-queueing-system-93ab10f05f99)
  * [dynein](https://github.com/airbnb/dynein)

* [FOQS: Scaling a distributed priority queue](https://engineering.fb.com/2021/02/22/production-engineering/foqs-scaling-a-distributed-priority-queue/)
* [FOQS: Making a distributed priority queue disaster-ready](https://engineering.fb.com/2022/01/18/production-engineering/foqs-disaster-ready/)
* [Scaling services with Shard Manager](https://engineering.fb.com/2020/08/24/production-engineering/scaling-services-with-shard-manager/)
* [Distributed delay queues based on Dynomite](https://netflixtechblog.com/distributed-delay-queues-based-on-dynomite-6b31eca37fbc)
  * [dynomite](https://github.com/Netflix/dynomite/wiki/Architecture)
  * [dyno-queues](https://github.com/Netflix/dyno-queues)
  * [Introducing Dynomite — Making Non-Distributed Databases, Distributed](https://netflixtechblog.com/introducing-dynomite-making-non-distributed-databases-distributed-c7bce3d89404)
* [高性能延迟服务实现之路]
* [Austin](https://gitee.com/zhongfucheng/austin)

* 技术夜校｜短信服务平台化实践
* 技术夜校｜从0到1构建一个完整的消息系统
* 得物App消息服务平台化建设演化
* 日均数亿推送稳定性监控实践 ｜ 得物技术


##业务系统设计
* [Rebuilding Payment Orchestration at Airbnb](https://medium.com/airbnb-engineering/rebuilding-payment-orchestration-at-airbnb-341d194a781b)
* [Airbnb 的统一支付数据读取流程](https://www.infoq.cn/article/X16w5IIdYwYPO38vF58o)
* [Unified Payments Data Read at Airbnb](https://medium.com/airbnb-engineering/unified-payments-data-read-at-airbnb-e613e7af1a39)
* [Engineering Uber’s Next-Gen Payments Platform](https://www.uber.com/blog/payments-platform/)

* [Avoiding Double Payments in a Distributed Payments System](https://medium.com/airbnb-engineering/avoiding-double-payments-in-a-distributed-payments-system-2981f6b070bb)
* [Airbnb’s Promotions and Communications Platform](https://medium.com/airbnb-engineering/airbnbs-promotions-and-communications-platform-6266f1ffe2bd)
* [Himeji: A Scalable Centralized System for Authorization at Airbnb](https://medium.com/airbnb-engineering/himeji-a-scalable-centralized-system-for-authorization-at-airbnb-341664924574)
* [How Airbnb Standardized Metric Computation at Scale](https://medium.com/airbnb-engineering/airbnb-metric-computation-with-minerva-part-2-9afe6695b486)
* [Rapid Event Notification System at Netflix](https://netflixtechblog.com/rapid-event-notification-system-at-netflix-6deb1d2b57d1)

* [Taming Content Discovery Scaling Challenges with Hexagons and Elasticsearch](https://doordash.engineering/2022/06/28/taming-content-discovery-scaling-challenges-with-hexagons-and-elasticsearch/)

* [Migrating Netflix's Viewing History from Synchronous Request-Response to Async Events](https://www.infoq.com/articles/microservices-async-migration/)
* [The Road to uChat: Building Uber’s Internal Chat Solution](https://www.uber.com/blog/uchat/)
* B站评论系统架构设计
## Storage
* [Open-sourcing LogDevice: A distributed data store for sequential data](https://engineering.fb.com/2018/09/21/core-data/open-sourcing-logdevice-a-distributed-data-store-for-sequential-data/)
* [LogDevice: a distributed data store for logs](https://engineering.fb.com/2017/08/31/core-data/logdevice-a-distributed-data-store-for-logs/)
* [Augmenting Flexible Paxos in LogDevice to improve read availability](https://engineering.fb.com/2022/03/07/core-data/augmenting-flexible-paxos-logdevice/)
* [Introducing and Open Sourcing Ambry](https://engineering.linkedin.com/blog/2016/05/introducing-and-open-sourcing-ambry---linkedins-new-distributed-)
* [HTTP/2 in infrastructure: Ambry network stack refactoring](https://engineering.linkedin.com/blog/2021/http-2-in-infrastructure--ambry-network-stack-refactoring)
  * [ambry.pdf](http://dprg.cs.uiuc.edu/data/files/2016/ambry.pdf)

* [File Systems Unfit as Distributed Storage Backends:Lessons from 10 Years of Ceph Evolution](https://www.pdl.cmu.edu/PDL-FTP/Storage/ceph-exp-sosp19.pdf)

* [Everything in its write place: Cloud storage abstraction with Object Store](https://dropbox.tech/infrastructure/abstracting-cloud-storage-backends-with-object-store)

* [Consolidating Facebook storage infrastructure with Tectonic file system](https://engineering.fb.com/2021/06/21/data-infrastructure/tectonic-file-system/)
  * [Facebook’s Tectonic Filesystem: Efficiency from Exascale](https://www.usenix.org/system/files/fast21-pan.pdf)

## 现代数据栈
* [Breaking State and Local Data Silos with Modern Data Architectures](https://blog.cloudera.com/breaking-state-and-local-data-silos-with-modern-data-architectures/)

* [A Flexible and Efficient Storage System for Diverse Workloads](https://blog.cloudera.com/a-flexible-and-efficient-storage-system-for-diverse-workloads/)



## KV
* [Under the Hood: Building and open-sourcing RocksDB](https://engineering.fb.com/2013/11/21/core-data/under-the-hood-building-and-open-sourcing-rocksdb/)
* [Future-proofing our metadata stack with Panda, a scalable key-value store](https://dropbox.tech/infrastructure/panda-metadata-stack-petabyte-scale-transactional-key-value-store)
* [Mussel — Airbnb’s Key-Value Store for Derived Data](https://medium.com/airbnb-engineering/mussel-airbnbs-key-value-store-for-derived-data-406b9fa1b296)
* [3 Innovations While Unifying Pinterest’s Key-Value Storage](https://medium.com/@Pinterest_Engineering/3-innovations-while-unifying-pinterests-key-value-storage-8cdcdf8cf6aa)

* [How we built a general purpose key value store for Facebook with ZippyDB](https://engineering.fb.com/2021/08/06/core-data/zippydb/)

* [分布式存储在B站的应用实践](https://zhuanlan.zhihu.com/p/570359883)

* [B站分布式KV存储实践](https://www.bilibili.com/read/cv15610515/)
* 百度信息流和搜索业务中的KV存储实践
* KV 存储引擎 - Badger源码分析


## MQ

* [MemQ: An efficient, scalable cloud native PubSub system](https://medium.com/pinterest-engineering/memq-an-efficient-scalable-cloud-native-pubsub-system-4402695dd4e7) 
  * [Unified PubSub Client at Pinterest](https://medium.com/pinterest-engineering/unified-pubsub-client-at-pinterest-397ccfaf508e)

* [Evaluating Apache Pulsar](https://zendesk.engineering/evaluating-apache-pulsar-92e6ed3fc792)
* [Building Reliable Reprocessing and Dead Letter Queues with Apache Kafka](https://www.uber.com/blog/reliable-reprocessing/)

## Cache
* [Improving Distributed Caching Performance and Efficiency at Pinterest](https://medium.com/pinterest-engineering/improving-distributed-caching-performance-and-efficiency-at-pinterest-92484b5fe39b)
* [Scaling Memcache at Facebook](https://www.usenix.org/system/files/conference/nsdi13/nsdi13-final170_update.pdf)
* [Introducing mcrouter: A memcached protocol router for scaling memcached deployments](https://engineering.fb.com/2014/09/15/web/introducing-mcrouter-a-memcached-protocol-router-for-scaling-memcached-deployments/)
* [Overlord Memcache and redis&cluster proxy](https://github.com/bilibili/overlord)
* [Cache warming: Agility for a stateful service](https://netflixtechblog.com/cache-warming-agility-for-a-stateful-service-2d3b1da82642)
* [大量数据的操作应该使用什么缓存策略？](https://www.zhihu.com/question/22336651/answer/2694018364)
* Redis数据倾斜与JD开源hotkey源码分析揭秘


## 任务调度
* [Beyond Interactive: Notebook Innovation at Netflix](https://netflixtechblog.com/notebook-innovation-591ee3221233)
* [Part 2: Scheduling Notebooks at Netflix](https://netflixtechblog.com/scheduling-notebooks-348e6c14cfd6)

* [Meson: Workflow Orchestration for Netflix Recommendations](https://netflixtechblog.com/meson-workflow-orchestration-for-netflix-recommendations-fc932625c1d9)

## 规则引擎
* [Building a Rule-Based Platform to Manage Netflix Membership SKUs at Scale](https://netflixtechblog.com/building-a-rule-based-platform-to-manage-netflix-membership-skus-at-scale-e3c0f82aa7bc)
* [Growth Engineering at Netflix- Creating a Scalable Offers Platform](https://netflixtechblog.com/growth-engineering-at-netflix-creating-a-scalable-offers-platform-69330136dd87)

## 性能优化
* [Netflix FlameScope](https://netflixtechblog.com/netflix-flamescope-a57ca19d47bb)
    * [flamescope](https://github.com/Netflix/flamescope)
* [A Microscope on Microservices](https://netflixtechblog.com/a-microscope-on-microservices-923b906103f4)
* [Java in Flames](https://netflixtechblog.com/java-in-flames-e763b3d32166)
* [Introducing Vector: Netflix’s On-Host Performance Monitoring Tool](https://netflixtechblog.com/introducing-vector-netflixs-on-host-performance-monitoring-tool-c0d3058c3f6f)  
* [Seeing through hardware counters: a journey to threefold performance increase](https://netflixtechblog.com/seeing-through-hardware-counters-a-journey-to-threefold-performance-increase-2721924a2822)
* [Saving 13 Million Computational Minutes per Day with Flame Graphs](https://netflixtechblog.com/saving-13-million-computational-minutes-per-day-with-flame-graphs-d95633b6d01f)
* [Trace Event, Chrome and More Profile Formats on FlameScope](https://netflixtechblog.com/trace-event-chrome-and-more-profile-formats-on-flamescope-5dfe9df5dfa9)

* [Performance @Scale 2019 recap](https://engineering.fb.com/2019/08/09/developer-tools/performance-scale-2019-recap/)

* [Consistent caching mechanism in Titus Gateway](https://netflixtechblog.com/consistent-caching-mechanism-in-titus-gateway-6cb89b9ce296)
* [2022 DoorDash Summer Intern Projects Article #2](https://doordash.engineering/2022/10/26/2022-doordash-summer-intern-projects-article-2/)
* [Optimising serverless for BBC Online](https://medium.com/bbc-product-technology/optimising-serverless-for-bbc-online-118fe2c04beb)

------------------------------------------------------
