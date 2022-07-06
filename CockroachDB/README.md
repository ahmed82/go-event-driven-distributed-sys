# CockroachDB
Use a local cluster
## Download the executable

Use PowerShell to download the CockroachDB v22.1.2 archive for Windows and copy the binary into your PATH:
```ps
$ErrorActionPreference = "Stop"; [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12;$ProgressPreference = 'SilentlyContinue'; $null = New-Item -Type Directory -Force $env:appdata/cockroach; Invoke-WebRequest -Uri https://binaries.cockroachdb.com/cockroach-v22.1.2.windows-6.2-amd64.zip -OutFile cockroach.zip; Expand-Archive -Force -Path cockroach.zip; Copy-Item -Force "cockroach/cockroach-v22.1.2.windows-6.2-amd64/cockroach.exe" -Destination $env:appdata/cockroach; $Env:PATH += ";$env:appdata/cockroach"

```
```ps
PS cockroach version
```

Find the binary installation path
```
Get-Command cockroach
or
gcm cockroach
```
Then set the env in the path

 Or 
 ## Use Docker
 ```
 C:\Users\username> docker pull cockroachdb/cockroach:v22.1.2
 ```
 

----------------------------------------------------------------------------

## Set up CockroachDB 

#### Set up CockroachDB  Cluster with three nodes

For **Windows** remove the `--background` flag as it is not suported.

```
cockroach start \
--insecure \
--store=orders-1 \
--listen-addr=localhost:26257 \
--http-addr=localhost:8080 \
--join=localhost:26257,localhost:26258,localhost:26259 \
--background
```

```
cockroach start \
--insecure \
--store=orders-2 \
--listen-addr=localhost:26258 \
--http-addr=localhost:8081 \
--join=localhost:26257,localhost:26258,localhost:26259 \
--background
```
```
cockroach start \
--insecure \
--store=orders-3 \
--listen-addr=localhost:26259 \
--http-addr=localhost:8082 \
--join=localhost:26257,localhost:26258,localhost:26259 \
--background
```

#### Use the cockroach init command to perform a one-time initialization of the cluster, sending the request to any node on the --join list.
```
cockroach init --insecure --host=localhost:26257
```

#### Start a SQL Shell for CockroachDB:
```
cockroach sql --insecure --host=localhost:26257
```

#### Create user
```
cockroach user set ahmed --insecure
```

#### Create Databases
```
cockroach sql --insecure -e 'CREATE DATABASE eventstoredb'
cockroach sql --insecure -e 'CREATE DATABASE ordersdb'
```

#### Grant privileges to the ahmed user
```
cockroach sql --insecure -e 'GRANT ALL ON DATABASE ordersdb TO ahmed'
cockroach sql --insecure -e 'GRANT ALL ON DATABASE eventstoredb TO ahmed'
```
---------------------------------------------------
----------------------------------------------------
## Run sample code

```
cockroach demo  --no-example-database
```
Copy the (sql) connection string in the SQL shell welcome text.
In a terminal set the DATABASE_URL environment variable to the connection string that you copied earlier:
```
$env:DATABASE_URL = "<connection-string>"
```
```
git clone https://github.com/cockroachdb/quickstart-code-samples
cd quickstart-code-samples/node
or
cd quickstart-code-samples/go
```
The code sample in this directory does the following:

- Connects to CockroachDB Cloud with the node-postgres driver using the connectiong string set in the DATABASE_URL environment variable.
or for Go app
- Connects to CockroachDB Cloud with the pgx driver using the connectiong string set in the DATABASE_URL environment variable.
**Then**
- Creates a table.
- Inserts some data into the table.
- Reads the inserted data.
- Prints the data to the terminal.

**For NodeJs**
```
npm install
node app.js
```
**For Go**
```
go mod init basic-sample
go mod tidy
go run main.go
```
Resources
- [User Authorization](https://www.cockroachlabs.com/docs/cockroachcloud/user-authorization?filters=client)
- [Learn CockroachDB SQL](https://www.cockroachlabs.com/docs/cockroachcloud/learn-cockroachdb-sql)
- [Build a Go App with CockroachDB and GORM](https://www.cockroachlabs.com/docs/v22.1/build-a-go-app-with-cockroachdb-gorm?filters=local)
- [Start a Local Cluster (Insecure)](https://www.cockroachlabs.com/docs/v22.1/start-a-local-cluster)
- [Deploy a Local Cluster with Kubernetes](https://www.cockroachlabs.com/docs/v22.1/orchestrate-a-local-cluster-with-kubernetes?filters=helm)

Take a moment to understand the flags you used:

- The `--insecure` flag makes communication unencrypted.
- Since this is a purely local cluster, `--listen-addr=localhost:26257` and `--http-addr=localhost:8080` tell the node to listen only on `localhost`, with port `26257` used for internal and client traffic and port 8080 used for HTTP requests from the DB Console.
- The `--store` flag indicates the location where the node's data and logs are stored.
- The `--join` flag specifies the addresses and ports of the nodes that will initially comprise your cluster. You'll use this exact `--join` flag when starting other nodes as well.
- For a cluster in a single region, set 3-5 `--join` addresses. Each starting node will attempt to contact one of the join hosts. In case a join host cannot be reached, the node will try another address on the list until it can join the gossip network.
- The `--background` flag starts the `cockroach` process in the background so you can continue using the same terminal for other operations.

NOTE: `--background` is deprecated on Windows OS.
