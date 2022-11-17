1. Create docker for mongo ports in vm1 and vm2 as given in directory here vm1 is primary
2. Generate keyauth file 

    2.1 openssl rand -base64 756 > keyauth
    
    4.2 sudo chmod 400 keyauth
    
    5.3 sudo chown 999:999 keyauth

3. Keep same keyauth file in both vms i.e copy key from keyauth file and create file with same name and give same permissions to the file in both vms

4. Give same mount path as given in two config files or else it will generate issues

5. Make docker up in vm1 
    docker file :

    version: "3.8"
    services:
      mongo1:
      image: mongo:4.0.4
      container_name: mongo1
      environment:
       - MONGO_DATA_DIR=/data/db
       - MONGO_LOG_DIR=/dev/null
       - MONGO_INITDB_ROOT_USERNAME=admin
       - MONGO_INITDB_ROOT_PASSWORD=admin
      #extra_hosts:
      #- "mongo2:"<IP of vm2>"
      volumes:
        - ./data/dbdata:/data/db
        - ./keyauth:/data/keyauth
      #command: mongod --port 27017 --keyFile /data/keyauth --dbpath /data/db --replSet rs0 --bind_ip 0.0.0.0
      command: mongod  --port 27017 --bind_ip 0.0.0.0
      network_mode: host


    5.1 It will create the data directory and will save the user name and password in env
    
    5.2 make the docker down in vm1
    
    5.3 comment the env variables and run the docker in vm1 again
    
    docker file :

    version: "3.8"
    services:
      mongo1:
      image: mongo:4.0.4
      container_name: mongo1
      <!-- environment:
       - MONGO_DATA_DIR=/data/db
       - MONGO_LOG_DIR=/dev/null
       - MONGO_INITDB_ROOT_USERNAME=admin
       - MONGO_INITDB_ROOT_PASSWORD=admin -->
      #extra_hosts:
      #- "mongo2:"<IP of vm2>"
      volumes:
        - ./data/dbdata:/data/db
        - ./keyauth:/data/keyauth
      command: mongod --port 27017 --keyFile /data/keyauth --dbpath /data/db --replSet rs0 --bind_ip 0.0.0.0
      <!-- command: mongod  --port 27017 --bind_ip 0.0.0.0 -->
      network_mode: host

6. Make the docker up in vm2
    docker file:
    version: "3"
    services:
      mongo2:
      image: mongo:4.0.4
      container_name: mongo2
      volumes:
        - ./data/dbdata:/data/db
        - ./keyauth:/data/keyauth
     #extra_hosts:
       #- "mongo1:"<IP of vm1>"
      command: mongod --port 37017 --keyFile /data/keyauth --dbpath /data/db --replSet rs0 --bind_ip 0.0.0.0
      network_mode: host 
    
    6.1 If any Issue comes remove the created data directory
    
    6.2 Check if same auth key file present in both vms
    
    6.3 Give same permissions to authkey as shown in 2.2 and 2.3

7. After making docker up in vm2 check if port connection is there in both vms using telnet/mongosh/mongo (port connectivity is must needed)
    
    7.1 To check telnet connectivity in both vm
        a.In vm1 terminal
            telnet <Ip of vm2>  <port of vm2>
        b.In vm2 terminal
            telnet <Ip of vm1>  <port of vm1>
        OR
    
    7.2 To check mongo connectivity in both vm
        a.In vm1 terminal
            mongo --host <Ip of vm2> --port <port of vm2>
        b.In vm2 terminal
            mongo --host <Ip of vm1> --port <port of vm1>
        OR
    
    7.2 To check mongosh connectivity in both vm
        a.In vm1 terminal
            mongosh --host <Ip of vm2> --port <port of vm2>
        
        b.In vm2 terminal
            mongosh --host <Ip of vm1> --port <port of vm1>
    
    7.3 Any one of above connectivity check is must needed if connectivity is there then proceed further

8. Go to primary node i.e vm1

    8.1 After making docker up execute following commands
        
        a. Bash into primary node container
            docker exec -it mongo1 bash
        
        b. Login with userName and password given in env in docker
            mongo --port 27017 --host 192.168.29.156 -u admin -p admin --authenticationDatabase admin
        
        c. Initiate replicaset and add member
            rs.initiate( 
                {
                    _id : "rs0",members: 
                    [
                        { 
                            _id : 0, 
                            host : "192.168.29.156:27017"
                        },
                        { 
                            id : 1, 
                            host : "192.168.29.112:37017" 
                        }
                    ]
                }
            ) 
        
        d. check status
            rs.status()
        
        e. If current node is showing as secondary logout and relogin then check the status
        
        f. You can configure the nodes using rs.conf() aswell

        g. Now create a common app user foe all nodes hit the following commands in same place
            
            1.1 admin = db.getSiblingDB("admin")
            
            1.2  admin.createUser
            
            (
                {
                    user: "appuser",
                    pwd: "adminpw",
                    roles: [ { role: "root", db: "admin" } ]
                }
            )
            
        h. You can use the appuser and password to login from other nodes

9. How to login from other nodes
    9.1. Bash into primary node container
            docker exec -it mongo1 bash
    
    9.2. Login with userName and password given in env in docker
           mongo --port 37017 --host 192.168.29.112 -u appuser -p adminpw --authenticationDatabase admin

10. The Replicaset Setup is done

Note:
    1. Now you can add data in db from primary node and test the replication
    
    2. You can follow the process of vm1 to make any primary node in your system 
    
    3. This replication only work in different servers, vms and machines

