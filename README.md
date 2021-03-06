# Ansible Cassandra Cluster Stress 

A framework for running and automating Cassandra stress tests on Amazon EC2.

This repository contains Ansible playbooks and scripts for running
Cassandra performance tests on EC2 with multiple Cassandra servers and
multiple cassandra-stress loaders, as well as adding, stopping and
starting nodes.

Optional Graphite server, with [Tessera](http://urbanairship.com/blog/2014/06/30/introducing-tessera-a-graphite-frontend) front-end, collect and present
relevant metrics.

### Prerequisites

Configure EC2 credentials:

```sh
export EC2_KEY_NAME=<...>
export AWS_ACCESS_KEY_ID=<...>
export AWS_SECRET_ACCESS_KEY=<...>
```

Make sure your EC2 SSH key is included in the keychain:

```sh
ssh-add <key-file>
```

To avoid prompting for SSH key confirmation
```
export ANSIBLE_HOST_KEY_CHECKING=False
```
Make sure you have boto version 2.34.0 or above.
To check boto version:
```
pip show boto
```
To update boto
```
sudo pip install --upgrade boto
```

To make sure your server use a unique name on EC2, set
ANSIBLE_EC2_PREFIX
```
export ANSIBLE_EC2_PREFIX=tzach
```

### Setup

#### Launch Graphite server
```
./ec2-setup-graphite.sh
```
You can access the Graphite UI at port 8080, or the Tessera UI at
port 80.


#### Launch Cassandra cluster
```
./ec2-setup-cassandra.sh <options>
```

  **options**
  * ```-e "cluster_nodes=2"``` - number of cluster nodes (default 2)
  * ```-e "instance_type=m3.large"``` - type of EC2 instance
  * ```-e "num_tokens=6"``` - set number of vnode per server


#### Launch Loaders
```
./ec2-setup-loadgen.sh <options>
```

  **options**
  * ```-e "load_nodes=2"``` - number of loaders  (default 2)
  * ```-e "instance_type=m3.large"``` - type of EC2 instance

#### Add nodes to existing cluster
```
./ec2-add-node-to-cluster.sh
```

  **options**
  * ```-e "stopped=true"``` - start server is stopped state
  * ```-e "instance_type=m3.large"``` - type of EC2 instance
  * ```-e "cluster_nodes=2"``` - number of nodes to add (default is 2)

#### Start stopped nodes (one by one)
```
./ec2-start-server.sh <number of servers>
```

### Run Load

```
./ec2-stress.sh <iterations> -e "load_name=late_night" <more options>
```

**Options**
All parameters are available at
[group_vars/all](https://github.com/cloudius-systems/ansible-cassandra-cluster-stress/blob/master/group_vars/all)

To override each of the parameter, use the ``-e "param=value"
notation. Examples below:

* ```-e "populate=2500000"``` - key population per loader (default is 1000000)
* ```-e "command=write"``` [read, write,mixed,..] setting stress command
* ```-e "stress_options='-schema \"replication(factor=2)\"'"```
* ```-e "stress_options='-errors ignore'"```
* ```-e "command_options='cl=LOCAL_ONE -mode native cql3'"```
* ```-e "command_options=duration=60m"```
* ```-e "profile_dir=/tmp/cassandra-stress-profiles/" -e "profile_file=cust_a/users/users1.yaml" -e "command_options=ops\\(insert=1\\)"```

Make sure **load name is unique**  so the new results will not
override an old run in the same day.

Load name can not not include *-* (dash)

### Results

The results are automatically collected and copy to your local
```/tmp/cassandra-results/``` directory, including summary files per load. 

### Stop and start nodes
Stop a server in the cluster

```
./ec2-stop-server.sh external_ip(s)
```

Stop and clean node data
```
./ec2-stop-server.sh external_ip -e "clean_data=true"
```

This will stop Cassandra service on this server, while keeping it in
the cluster.

Use ```./ec2-start-server.sh``` to start stopped servers, or just
update the **stopped** tag back to false.
The next stress test will restart the Cassandra service.

### Terminate Servers
(except the Graphite server)
```
./ec2-terminate.sh
```

## License
The Cassandra Cluster Stress is distributed under the Apache License.
See LICENSE.TXT for more.
