---------------------------------------------replica set and storing data in it ------------------------------------

1. Make diroctories for primary and secondary nodes
	
	sudo mkdir -p /srv/mongodb/rs0-0 /srv/mongodb/rs0-1 /srv/mongodb/rs0-2

2. Run ports as replica set 
	
	sudo mongod --replSet rs0 --port 2700 --dbpath /srv/mongodb/rs0-0 --oplogSize 128

	sudo mongod --replSet rs0 --port 2701 --dbpath /srv/mongodb/rs0-1 --oplogSize 128

	sudo mongod --replSet rs0 --port 2702 --dbpath /srv/mongodb/rs0-2 --oplogSize 128

3. Login to port which you want to set it as primary and initialize it
	
	a. log in to primary port
		mongo --port 2700   e.i. -> mongo --host <hostName> --port <portNumber>
	
	b. initialize the replicaset
		rs.initiate()

4. Access the replica set from golang
	
	a. import following packeges
		github.com/juju/mgo/v2
		github.com/juju/replicaset

	b. Access the primary node from golang
	
		Host := []string{
		"localhost:2700",
	}
	const (
		ReplicaSetName = "rs0"
	)
	session, err := mgo.DialWithInfo(&mgo.DialInfo{
		Addrs:          Host,
		ReplicaSetName: ReplicaSetName,
	})

5. Add members in replica set
	
	a. Run node as replicaset which you want to add as secondary 
		sudo mongod --replSet rs0 --port 2701 --dbpath /srv/mongodb/rs0-1 --oplogSize 128
	
	b. add the node as member from golang
		member := replicaset.Member{}
		member.Id = 1
		member.Address = "localhost:2701"
		err = replicaset.Add(session, member)

6. You can read, write, remove, create only on session of primery node from step 4.b
		
	
		
		

	
	