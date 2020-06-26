# Cassandra with Java

## Maven Dependency

Define the following Cassandra dependency in the ``pom.xml``

```
<dependency>
      <groupId>com.datastax.cassandra</groupId>
      <artifcatId>cassandra-driver-core</artifactId>
      <version>3.1.0</version>
 </dependency>
 ```
 
 In order to test the code with an embedded database server we should also add the cassandra-unit dependency
 
 ```
 <dependency>
      <groupId>org.cassandraunit</groupId>
      <artifactId>cassandra-unit</artificatId>
      <version>3.0.0.1</version>
  </dependency>
  ```
  
  + In order to connect to Cassandra from Java, we need to build a Cluster object.
  + An address of a node needs to be privided as a contact point. If we don't provide a port number, the default port (9042) will be used.
  
  ```
  public class CassandraConnector {
        
        private Cluster cluster;
        private Session session;
        
        public void connect(String node, Integer port) {
                Builder b = Cluster.builder().addContactPoint(node);
                
                if (port != null) {
                      b.withPort(port);
                }
                cluster = b.build()
                
                session = cluster.connect();
        }
        
        public Session getSession() {
          return this.session;
        }
        
        public void close() {
           session.close();
           cluster.close();
        }
}
  
