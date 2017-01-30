# GALERA CLUSTER FOR MYSQL
synchronous replication에 기반한 MySQL Multimaster Cluster.

## What is Galera?
이탈리라어. 갤리(galley)선(특히 고대 그리스나 로마 시대 때 주로 노예들에게 노를 젓게 한 배)
갤리선이 빠르게 가기 위해선 수많은 선원들이 synchronously하게 노젓기를 할 수 있도록 해야하는데 galeracluster의 역할을 표현한다.

## 개요 및 특징
http://galeracluster.com/products/
write set replication (wsrep) interface의 구현체.
2007년 첫 릴리즈. open source.
OpenStack에서도 사용. 가이드 : http://docs.openstack.org/high-availability-guide/content/ha-aa-db-mysql-galera.html
* Synchronous replication => consistency.
* Active-active multi-master topology => zero failover time.
* Read and write to any cluster node
* Automatic membership control, failed nodes drop from the cluster(elasticity)
* Automatic node joining
* True parallel replication, on row level
* Direct client connections, native MySQL look & feel
* No slave lag
* No lost transactions
* Both read and write scalability
* Smaller client latencies

## WSREP?
https://launchpad.net/wsrep/
certification 기반의 multi-master replication을 위한 인터페이스.
transaction (commit)write sets을 관리하는 애플리케이션 callback API와 replication plugin(library) call API를 정의한다.
```
wsrep API defines a set of application callbacks and replication library calls necessary to implement synchronous writeset replication of transactional databases and similar applications. It aims to abstract and isolate replication implementation from application details. Although the main target of this interface is a certification-based multi-master replication, it is equally suitable for both asynchronous and synchronous master/slave replication.
```

## 설치 컴퍼넌트
* wsrep(replication) provider
* MySQL patch

최소 3대의 hosts애 설치 필요함.
클러스터에 먼저 한대(Primary Component) 띄우고
`# service mysql start --wsrep-new-cluster`
SHOW STATUS LIKE 'wsrep_cluster_size';
```
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 1     |
+--------------------+-------+
```
나머지 노드들 띄우면 클러스터에 합류하게됨.
`# service mysql start`
```
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 2     |
+--------------------+-------+
```

## Why High Availability and Cloud are more difficult for databases?
DB는 stateless 하지 않다. data가 크면 copy하는데 만 수시간. 스냅샷 포인트도 고려해야 하고, 서비스 중단없이 투입하려면 replication 기술이 요함.

## database clustering solutions vs Galera-Cluster-for-MySQL
http://galeracluster.com/wp-content/uploads/2013/10/Minimizing-downtime-and-maximizing-elasticity-with-Galera-Cluster-for-MySQL.pdf

* no scale-out benefit at all (active-passive) 
* can only scale read-only transactions (master-slave)
* Well known database clustering solutions typically have at least two of the following
downsides
1) risk of lost data due to asynchronous replication, 
2) Up to 50% performance overhead, 
3) downtime at planned and unplanned failovers may be up to several minutes, 
4) poor compatibility compared to running with a non-clustered database,
5) prohibitively difficult to use
6) not suitable for Widea Area Network replication between datacenters/continents,
7) prohibitively expensive to use.


보통 Synchronous replication이 network latency에 취약한데, Galera는 certification-based replication를 통해 이를 해결했다고함. 심지어 여러 대륙간에 clustering 해도 문제없다고 함. Google Cloud SQL처럼 여러 대륙에 clustering 해서 재난 대비 및 end user에 response time 줄이는 효과.

https://www.nginx.com/blog/mysql-high-availability-with-nginx-plus-and-galera-cluster/
http://www.severalnines.com/mysql-load-balancing-haproxy-tutorial
http://www.severalnines.com/clustercontrol

## CLUSTER DEPLOYMENT VARIANTS
http://galeracluster.com/documentation-webpages/deploymentvariants.html

## DATABASE REPLICATION
http://galeracluster.com/documentation-webpages/introduction.html
### ASYNCHRONOUS AND SYNCHRONOUS REPLICATION
#### Advantages of Synchronous Replication
High Availability - no data loss, consistent
Improved Performance - transaction on all nodes in parallel.
Causality across the Cluster - causality across the whole cluster.

#### Disadvantages of Synchronous Replication
m = n X o X t (n: number of nodes, o: number of operation, t: tps)
즉, nodes의 증가는 응답시간과 conflict, deadlock 발생확률의 기하급수적인 증가 야기.

### SOLVING THE ISSUES IN SYNCHRONOUS REPLICATION
certification-based replication system.

* Group Communication
* Write-sets
* Database State Machine
* Transaction Reordering 

## CERTIFICATION-BASED REPLICATION
optimistic execution.
클러스터 내 모든 노드에 commit 복제시 certification test를 수행하는데 primary key 기반으로 conflicts을 체크 해서 빠른듯.

http://galeracluster.com/documentation-webpages/certificationbasedreplication.html

### WHAT DB Features CERTIFICATION-BASED REPLICATION REQUIRES
* Transactional Database 
* Atomic Changes
* Global Ordering

### HOW CERTIFICATION-BASED REPLICATION WORKS
http://galeracluster.com/documentation-webpages/certificationbasedreplication.html

## REPLICATION API
http://galeracluster.com/documentation-webpages/architecture.html
* Database Management System (DBMS) : The database server that runs on the individual node. Galera Cluster can use MySQL, MariaDB or Percona XtraDB.
* wsrep API : The interface and the responsibilities for the database server and replication provider. It consists of:
	* wsrep hooks The integration with the database server engine for write-set replication.
	* dlopen() The function that makes the wsrep provider available to the wsrep hooks.
* Galera Replication Plugin : The plugin that enables write-set replication service functionality.
* Group Communication plugins : The various group communication systems available to Galera Cluster. For instance, gcomm and

## ISOLATION LEVELS
The SNAPSHOT-ISOLATION : REPEATABLE-READ 와 SERIALIZABLE 중간

## STATE TRANSFERS
State Snapshot Transfer, Incremental State Transfer

## 기타
* 화이트페이퍼 : http://galeracluster.com/wp-content/uploads/2013/10/Minimizing-downtime-and-maximizing-elasticity-with-Galera-Cluster-for-MySQL.pdf
* github : https://github.com/codership/galera
* 용어 : http://galeracluster.com/documentation-webpages/glossary.html
