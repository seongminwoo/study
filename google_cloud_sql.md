## Google Cloud SQL 개요
### https://cloud.google.com/sql/
* MySQL database that lives in Google's cloud. 기존 MySQL에 비해 몇가지 추가기능과 제한사항이 있음.
* Google handles replication(sync, async), patch management and database management.
* fast, don’t run out of space, reliable storage.
* Instances available up to 16GB RAM, 500GB storage.
* pay-per-use model(pay for a database only when it’s being accessed) - SaaS 서비스 사업자에게 적합
* data is replicated in many geographic locations ,failover automatically.(ISO/IEC 27001 compliant)
* 사용자가 DB 인스턴스 설치 위치를 결정할 수 있다. ex) 미국IDC ,유럽 IDC 등
* data is automatically encrypted.(ASE-256)
* Easier Migration.
* No Lock-in.
* [web Console or a command-line interface.](https://cloud.google.com/sql/docs/getting-started)
* [Google Cloud Platform Pricing Calculator](https://cloud.google.com/products/calculator/)

## 성능/확장성 측면에서 엔터프라이즈급으로 사용이 가능한 것인가 ?
scale-up은 16GB of RAM and 500GB data storage가 max.
가이드에 ideal for small to medium-sized applications라고 나와 있는걸로 봐서 대규모 엔터프라이즈급에서는 사용하기 힘들어보임.
또한 유료 사용 Packages 등급에 메모리와 저장소 용량만 나오지 성능에 중요한 요소중 하나인 CPU 스펙이 나오질 않음.
고가용성과 사용성은 매우 우수.
we cannot guarantee to restore instances larger than 250GB within 24 hours (smaller instances restore a lot faster).

https://cloud.google.com/sql/docs/launch-checklist
```
If you want the benefits of Google managing updates to MySQL, replicating your data automatically, and high availability, then Cloud SQL is likely the best choice for your application. If your goal is pure performance and you’re prepared to manage databases and data, then MySQL on a Google Compute Engine VM is likely a better choice for your application.
```

https://cloud.google.com/sql/docs/introduction
* Restrictions
https://cloud.google.com/sql/faq#supportmysqlfeatures
[Performance schema](https://dev.mysql.com/doc/refman/5.6/en/performance-schema.html)
[InnoDB memcached plugin](https://dev.mysql.com/doc/refman/5.6/en/innodb-memcached.html)

## Google Cloud SQL을 지원하려면 어떻게 해야하냐?
GCE 클라우드 안에서든 밖에서는 Cloud SQL을 설치하면 IP가 나오기 때문에 IP로 접근 가능.
```
$ gcloud sql instances patch YOUR_INSTANCE_NAME --assign-ip

In the output, find the "ipAddress" field. Use this as the IP address your applications or tools use to connect to the instance.
```

## FAQ
### https://cloud.google.com/sql/faq
### How does Google Cloud SQL failover work?
수 초간의 downtime은 있지만 auto failover. ip주소 인스턴스 이름 그대로 사용.
```
Failover is designed to be transparent to your applications, so that after failover, an instance has the same instance name, IP address, and firewall rules. During the failover there will typically be a few seconds downtime as the instance starts up in a new zone. However, in some cases, the InnoDB crash-recovery process may take longer, delaying the time before the instance is up. 
```

### Do I need to use the Google Developers Console to manage Google Cloud SQL?
Console 뿐만 아니라 [Cloud SQL API](https://cloud.google.com/sql/docs/admin-api/)를 이용해서 애플리케이션에서 mysql 인스턴스를 동적으로 제어 가능.
