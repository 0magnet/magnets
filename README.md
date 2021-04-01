# magnets

A golang web app for ecommerce inventory management

Uses [cockroachdb](https://www.cockroachlabs.com/docs/v20.2/build-a-go-app-with-cockroachdb-upperdb) for database and [upper/db](https://tour.upper.io/queries/01) as the database access layer.

Tested on Archlinux

## Prerequisite

```
yay -S cockroachdb go
```
it's recommended to use `cockroachdb-bin` for faster testing and deployment

## Single Node

in a terminal:
```
make clean0 certs0 single-node
```

cockroach node is started. **Proceed to database setup**

## Cluster Setup

Nodes act as access points to the database. This implementation does not consider them to be critical for data redundancy. Nodes can be started as-needed to give an access point (in this example) within the local network to the database. Follow along with the [upstream documentation of this process](https://www.cockroachlabs.com/docs/stable/deploy-cockroachdb-on-premises.html)

(note - you will need to change the Makefile for your specific cluster. Dummy values are present as placeholders)

```
make certs
```

The certificates are generated, and compressed into an archive. These must be copied to the corresponding node before continuing. Refer to the linked documentation above for a description of this process.

In this example, it is assume that this repository is cloned to the GOPATH on the nodes, and that, for instance, `certs1.tar.gz` is extracted into the cloned repository folder and is renamed `certs` from `certs1`.

## Starting the cluster
**after you have edited the Makefile**
on each node, beginning with the primary instance
```
make start1
```

the local instance
```
make start0
```

the third node
```
make start2
```

the fourth node
```
make start3
```

## Initialize the cluster
**you must edit the makefile first**
```
make init
```

## Database setup

in a new terminal or tab:
```
make db-secure
```

## Sync dependencies
```
go mod init
go mod vendor -v
```

## Running the application

Without arguments, the database connection will be briefly established and the help menu prints before exiting:
```
go run main.go
```

result:
```
Initializing cockroachDB connection
Usage: magnets -hdDctCepvir
  -C, --createpartno string   Create a part by providing the part number
  -c, --createtables          Create the tables if they do not exist
  -D, --deleteall             Delete the products in the products database
  -d, --droptables            Drop tables
  -e, --exportcsv             Export a csv to export01.csv
  -h, --help                  show this help menu
  -i, --importcsv             Import csv from http://127.0.0.1:8079/export01.csv
  -p, --printinventory        Print the inventory to the terminal
  -r, --run                   run the web app
  -t, --testprod              create test product
  -v, --vprintinventory       More verbose printinventory
```
Note: the order of the flags in `Usage: magnets -hdDctCepvir` is the order in which the flags are executed. Unfortunately the flags are (automatically) listed in the help menu in alphabetical order.

A standard invocation which demonstrates the basic functionality of this application is as follows:
```
go run main.go -ctr
```

* `-c` creates the table if it does not exist
* `-t` creates a test product
* `-p` print the inventory to the terminal
* `-r` run the web app

Output:
```
$ go run main.go -ctpr
Initializing cockroachDB connection
Creating 'products' table
Creating test product 'dummy'
Creating second test product 'dummy2'
2021/04/01 12:23:37 products:
product[646274800931078145]:
	partno:		test
	Image1:		test.jpg
	Name:		dummy
	Qty:		10
	Price:		1.00
	Enable:		true
product[646274800950607873]:
	partno:		test1
	Name:		dummy2
	Qty:		100
	Price:		1.00
	Enable:		true
listening on http://127.0.0.1:8040 using gorilla router
```

## Managing Inventory

A new product can be created with the `-C` flag, specifying the partno of the product to be created
```
$ go run main.go -C test2
```
result:
```
Initializing cockroachDB connection
Creating product with part number: test2'
```

The web app is more or less a display for the inventory. Updating, adding, and deleting inventory is expected to be done with importing and exporting CSV directly to cockroachDB.

The export command has been included in the Makefile
```
make export
```
the above command is called by `go run main.go -e` so be aware of the dependence upon the Makefile for that functionality

**CSV IMPORT IS NOT WORKING ROBUSTLY YET**

For importing CSV, create a directory called test
```
mkdir -p test
```
generate the export and copy it into the test folder
```
go run main.go -e && cp -b exprt01.csv test/export01.csv
```
start a webserver in the test directory on port 8079. For the convenience of the author, darkhttpd is used
```
darkhttpd . --port 8079
```
run the command for importing the CSV
```
go run main.go -i
```

**PLEASE NOTE:** The CSV format is very sensitive and many editors can change the formatting, such as libreoffice. It is recommended to use a text editor, such as atom, for editing the csv until these issues can be addressed.

## Production
Assuming a self-hosted instance (deployed on Archlinux)

requires caddy

```
yay -S caddy
```

reverse proxy to port 80 from 8040

```
sudo caddy reverse-proxy --from example.com --to localhost:8040
```
