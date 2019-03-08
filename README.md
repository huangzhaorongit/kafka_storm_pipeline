# kafka_storm_transformer
<h2>
  Diagram
</h2>
<h2>
  Requirements
</h2>
This project requires Ansible, Java8, Apache Maven and an AWS account. In addition, even though not required, Docker, redis-cli, Apache Kafka will be nice to have installed locally, to further explore various parts of this project.
<h2>
  Tools/Services Used
</h2>
  <ul>
  <li>Java</li>
  <li>Ansible</li>
  <li>Docker</li>
  <li>Apache Zookeeper</li>	
  <li>Apache Kafka</li>
  <li>Apache Storm</li>	
  <li>AWS</li>
    <ul>
      <li>EC2</li>
      <li>RDS</li>
      <li>Elasticache(Redis)</li>
      <li>S3</li>
    </ul>
  </ul>
<h2>
  Short Description
</h2>  
A real-time, dockerized Kafka event processor pipeline (utilizing an Apache Storm topology). The project is running in AWS with automated infrastructure deployment and execution, using Ansible.
<h2>
  Process Description
</h2>  
The events (in following format - <b>CustID,Balance</b>) for this pipline are being generated by kafka_scripts/SimpleProducer.java class included with this project. The inctances of the class above publish to Kafka topic from local machine. The topic acts as a source(<b>Spout</b> in Storm's world) for <b>StormLookup</b> topology, which for each event(<b>tuple</b> in Storm's world) does
<ul>
	<li><b>LookupBolt</b>
		<ol>
			<li>Extracts the <b>CustID</b> from the tuple </li>
			<li>Looks up the Redis cluster and gets the <b>SSN</b> for that <b>CustID</b></li>
			<li>Passes the <b>SSN</b> along with the <b>Balance</b> to both <b>RDSInserterBolt</b> and <b>S3WriterBolt</b></li>
		</ol>
		</li>
	<li><b>RDSInserterBolt</b>
		<ol>
			<li>For each tuple does an upsert in RDS PSQL <b>Balances</b> table</li>
		</ol>
	</li>
	<li><b>S3WriterBolt</b>
		<ol>
			<li>Accumulates the tuples received based on either sepcified count or specified time delta(whatever happens first)</li>
			<li>Generates a file based on above and writes into S3</li>
		</ol>
	</li>
</ul>
<h2>
  Execution
</h2>
In order to execute issue <b>ansible-playbook infrastructure.yml</b>, while using your own AWS user. Once ansible run is complete, run one or more instances of <b>SimpleProducer.java</b>. The pipeline will start populating the RDS and writing files into S3 at this point.
<h2>
  Execution Process Description
</h2>
	<ul>
		<li><b>ansible-playbook infrastructure.yml</b>
		<ol>
			<li>Creates a dedicated VPC for this project</li>
			<li>Creates a subnet inside that VPC, sets up an Internet Gateway and definesall the neccessary routes</li>
			<li>Creates a SecurityGroup to be assigned to different resources throughout this project</li>
			<li>Spins up EC2 instances and runs Zookeeper, Kafka and Storm Docker containers on them</li>
			<li>Creates an Elastichache(Redis) cluster and populates it with lookup data</li>
			<li>Deployes the storm topology</li>
		</ol>
			</li>
		<li><b>SimpleProducer</b>
		<ol>Produces events to kafka topic to be consumed by the pipeline</ol>
			</li>
	</ul>
<h2>
	To Do
</h2>
<ul>
<li>Storm Transformer</li>
<ol>
	<li>Change the Topology name from Obfuscator to CustomerLookupTopology</li>
	<li>Change the Obfuscator Bolt name to LookupBolt</li>
	<li>Modify the LookupBolt to do a Redis lookup</li>
	<li>Implement RDS writer bolt</li>
	<li>implement time/count based batching for S3WriterBolt</li>
</ol>
<li>PSQL</li>
<ol>
  <li>Start a psql database</li>
</ol>
</ul>
<h2>
Observations
</h2>
<ul>
	<li>If possible, always use Terraform for infrastructure creation. Ansible is good as far as working with already provisioned resources goes(i.e a kafka container is being spun up on an EC2 instance, however, terraform is much more intuitive as far as EC2 instance creation, itself, is concerned)</li>
</ul>
<h2>
  Warnings
</h2>
<ul>
  <li>Current configuration of this project will be using AWS services that are beyond the Free Tier!</li>
</ul>
