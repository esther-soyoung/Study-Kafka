# KSQL, SQL for Kafka
  
## Kappa Architecture
Before digging into KSQL, the understanding of Kappa architecture better come first. Kappa architecture refers to a data pipeline architecture that deploys unified computational tool regardless of size or duration of the data. Unlike Lambda architecture which stores data based on batch processing, Kappa architecture stream-processes data whenever query comes in.
![kappa architecture](img/kappa.png)
  
KSQL completes this unified one-shot architecture by simplyfing and unifying the querying step.
![ksql](img/ksql.png)
  
### KSQL Server
KSQL servers execute queries just as other databases, 
1. Receives query.  
2. Rewrites query.  
3. Logical plan for the query.  
4. Physical plan for the query.  
5. Executes the query.
but storing table meta data in its internal memory or a special Kafka topic named *ksql__commands*. 
  
### KSQL Client
KSQL Client transfers queries to KSQL servers and receives the result just as other database clients, but also supports **stream** type data which is **immutable**. 

## Reference
[Oreilly](https://www.oreilly.com/content/applying-the-kappa-architecture-in-the-telco-industry/)
[Confluent](https://docs.confluent.io/current/ksql/docs/concepts/ksql-architecture.html)

