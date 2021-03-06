[![Build Status](https://travis-ci.org/atsid/accumulo-mojo
.svg?branch=master)](https://travis-ci.org/atsid/accumulo-mojo
)
accumulo-mojo
=============

Maven Mojo for Accumulo.  Useful for integration testing.

This project is composed of two maven modules.  The module accumulo-maven-plugin is a maven plugin that enables integration test to use an Accumulo instance.  In addition to testing this project, accumulo-plugin-tests provide an example of using accumulo-maven-plugin.

To try accumulo-mojo out, execute the following maven commands :

 * Run 'mvn clean install'  
 * If all assertions pass in the integ test, then Output should be something like:
 
<pre>
	[INFO] ------------------------------------------------------------------------
	[INFO] Reactor Summary:
	[INFO] 
	[INFO] Accumulo Mojo Parent .............................. SUCCESS [0.509s]
	[INFO] Maven Accumulo Plugin ............................. SUCCESS [3.693s]
	[INFO] Accumulo Plugin Integration Tests ................. SUCCESS [25.665s]
	[INFO] ------------------------------------------------------------------------
	[INFO] BUILD SUCCESS
	[INFO] ------------------------------------------------------------------------
	[INFO] Total time: 30.369s
	[INFO] Finished at: Fri Apr 26 17:09:17 PDT 2013
	[INFO] Final Memory: 32M/639M
	[INFO] ------------------------------------------------------------------------
</pre>

Temporary directories will be created in the default temp location at runtime.  These will be removed when the mojo shuts down.

Before running this plugin make sure that there are not any instances of Hadoop, Zookeeper, or Accumulo running locally.  Mojo behavior is not defined when servers already exist.

### System Requirements

 * Linux OS
 * Maven (Tested with Maven3 only)

You do not need to install or configure Hadoop, Zookeeper, or Accumulo in order to use this plugin.  These dependencies will be downloaded automatically by Maven at runtime.

POM Usage
=============

### Add dependency plugin

```xml
  		<plugin>
				<groupId>com.atsid.mojo</groupId>
				<artifactId>accumulo-maven-plugin</artifactId>
				<version>0.2.1-SNAPSHOT</version>
				<executions>
					<execution>
						<goals>
							<goal>start-accumulo</goal>
							<goal>stop-accumulo</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
```
      
### Use in integration tests

  
```java
    @Test
    public void testCustomIterator() throws TableExistsException, TableNotFoundException, IOException, AccumuloException, AccumuloSecurityException {
      String tableName = "testTable";
    
      ZooKeeperInstance instance = new ZooKeeperInstance("accumulo", "localhost:2181");
    
      Connector connector = instance.getConnector("root", "password");
    
      connector.tableOperations().create(tableName);
    
      // Write test data
      BatchWriter writer = connector.createBatchWriter(tableName, 10000L, 1000L, 4);
      Mutation m = new Mutation(new Text("myRow"));
      m.put(new Text("IepIngestionInformation"), new Text("start-time"), new Value("SomeValue".getBytes()));
      m.put(new Text("IepIngestionInformation"), new Text("end-time"), new Value("SecondValue".getBytes()));
      writer.addMutation(m);
      writer.close();
    
      // Read test data
      BatchScanner scanner = connector.createBatchScanner(tableName, new Authorizations(), 1);
      List<Range> ranges = new ArrayList<Range>();
      ranges.add(new Range());
      scanner.setRanges(ranges);
     
      Iterator<Entry<Key,Value>> scannerIter = scanner.iterator();
    
    ...

```

### Custom iterator support in integration tests

The "start-accumulo" goal supports custom iterators.  You just need to make sure your custom iterator is on the maven build classpath.  If the custom iterator is not part of the current project in the dependency chain then a plugin dependency should be added.  This is done by adding a &lt;dependencies&gt; tag to the plugin definition in your pom.xml.  See the plugin configuration format at http://maven.apache.org/ref/3.0.4/maven-model/maven.html#class_plugin


Command Line Usage
=============

Most goals are designed to work within the integration test environment.  Only the "standalone" and "shell" goals are supposed to be used from the command line.

Please note that this plugin is not available on most public maven repositories.  If you plan to use this plugin without a project pom then you must make sure that maven can find the plugin.

This plugin has a "help" goal that will provide basic usage information.
 * List of available goals: "mvn com.atsid.mojo:accumulo-maven-plugin:0.4.0:help"
 * Parameters for a specific goal: "mvn com.atsid.mojo:accumulo-maven-plugin:0.4.0:help -Ddetail -Dgoal=&lt;goalName&gt;"

#### Run Accumulo server goal
 * Run 'mvn com.atsid.mojo:accumulo-maven-plugin:0.4.0:standalone -DdefaultTables="table1,table2"'
 * A log message will be written to the console when Accumulo is initialized
 * Two tables will be created (table1 and table2).  If this parameter is omitted then no tables are created.
 * Default Instance: accumulo
 * Default Zookeeper: localhost:2181
 * Default User: root
 * Default Password: password
 * When ready to shut down the Accumulo instance press CTRL-C
 * Maven project not required to execute this goal

#### Run Accumulo shell goal
 * Run 'mvn com.atsid.mojo:accumulo-maven-plugin:0.4.0:shell'
 * Default Instance: accumulo
 * Default Zookeeper: localhost:2181
 * Default User: root
 * Default Password: password
 * Maven project not required to execute this goal
