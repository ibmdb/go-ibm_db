go_ibm_db
==========

Interface for Go to DB2 for LUW,DB2 for iseries and DB2 for z/OS.

API Documentation
==================

For complete list of go_ibm_db APIs and example, please check [APIDocumentation.md] (https://github.com/ibmdb/go_ibm_db/APIDocumentation.md)

Prerequisite
=============

Golang should be installed in your system.

How to Install in Windows
=======================
```
go get -d github.com/ibmdb/go_ibm_db

go to installer folder in go_ibm_db (C:\Users\uname\go\src\github.com\ibmdb\go_ibm_db\installer) and run setup.go file (go run setup.go).

If you use the clidriver downloaded by the godriver then add the below path to your PATH env variable
C:\Users\uname\go\src\github.com\ibmdb\go_ibm_db\installer\clidriver\bin

Otherwise if you use your cli driver add path to your PATH env variable
path=\Path\To\Clidriver\bin

```

How to Install in Linux/Mac
===========================
```
go get -d github.com/ibmdb/go_ibm_db

go to installer folder in go_ibm_db (/home/Users/go/src/github.com/imdb/go_ibm_db/installer) and run setup.go file (go run setup.go).

If you use the clidriver downloaded by the godriver then set the below env variables:

export DB2HOME=/home/Users/go/src/github.com/imdb/go_ibm_db/installer/clidriver
export CGO_CFLAGS=-I$DB2HOME/include
export CGO_LDFLAGS=-L$DB2HOME/lib
(FOR LINUX)export LD_LIBRARY_PATH=/home/Users/go/src/github.com/ibmdb/go_ibm_db/installer/clidriver/lib
(FOR MACOS)export DYLD_LIBRARY_PATH=$DYLD_LIBRARY_PATH:/home/Users/go/src/github.com/ibmdb/go_ibm_db/installer/clidriver/lib

Otherwise if you use your cli driver then set the below env variables:

export DB2HOME=/Path/To/clidriver
export CGO_CFLAGS=-I$DB2HOME/include
export CGO_LDFLAGS=-L$DB2HOME/lib 
(FOR LINUX)export LD_LIBRARY_PATH=/Path/To/clidriver/lib
(FOR MACOS)export DYLD_LIBRARY_PATH=$DYLD_LIBRARY_PATH:/Path/To/clidriver/lib
```

How to run sample program
==========================

Example1.go:-
===========
```
package main

import (
    _ "github.com/ibmdb/go_ibm_db"
    "database/sql"
    "fmt"
)

func main(){
    con:="HOSTNAME=host;DATABASE=name;PORT=number;UID=username;PWD=password"
 db, err:=sql.Open("go_ibm_db", con)
    if err != nil{
        
		fmt.Println(err)
	}
	db.Close()
}
To run the sample:- go run example1.go
```

Example2.go:-
===========
```
package main

import (
    _ "github.com/ibmdb/go_ibm_db"
    "database/sql"
    "fmt"
)

func Create_Con(con string) *sql.DB{
 db, err:=sql.Open("go_ibm_db", con)
    if err != nil{
        
		fmt.Println(err)
		return nil
	}
	return db
}

//creating a table

func create(db *sql.DB) error{
    _,err:=db.Exec("DROP table SAMPLE")
	if(err!=nil){
    _,err:=db.Exec("create table SAMPLE(ID varchar(20),NAME varchar(20),LOCATION varchar(20),POSITION varchar(20))")
    if err != nil{
        return err
    }
	} else {
    _,err:=db.Exec("create table SAMPLE(ID varchar(20),NAME varchar(20),LOCATION varchar(20),POSITION varchar(20))")
    if err != nil{
        return err
    }
	}
	fmt.Println("TABLE CREATED")
    return nil
}

//inserting row

func insert(db *sql.DB) error{
    st, err:=db.Prepare("Insert into SAMPLE(ID,NAME,LOCATION,POSITION) values('3242','mike','hyd','manager')")
    if err != nil{
        return err
    }
    st.Query()
    return nil
}

//this api selects the data from the table and prints it

func display(db *sql.DB) error{
    st, err:=db.Prepare("select * from SAMPLE")
    if err !=nil{
        return err
    }
    err=execquery(st)
    if err!=nil{
        return err
    }
    return nil
}


func execquery(st *sql.Stmt) error{
    rows,err :=st.Query()
    if err != nil{
        return err
    }
	cols, _ := rows.Columns()
    fmt.Printf("%s    %s   %s    %s\n",cols[0],cols[1],cols[2],cols[3])
    fmt.Println("-------------------------------------")
    defer rows.Close()
    for rows.Next(){
        var t,x,m,n string
        err = rows.Scan(&t,&x,&m,&n)
        if err != nil{
            return err
        }
        fmt.Printf("%v  %v   %v         %v\n",t,x,m,n)
    }
    return nil
}

func main(){
    con:="HOSTNAME=host;DATABASE=name;PORT=number;UID=username;PWD=password"
	type Db *sql.DB
	var re Db
	re=Create_Con(con)
    err:=create(re)
	if err != nil{
        fmt.Println(err)
    }
    err=insert(re)
    if err != nil{
        fmt.Println(err)
    }
    err=display(re)
    if err != nil{
        fmt.Println(err)
    }
}
To run the sample:- go run example2.go
```

Example3.go:-(POOLING)
====================
```
package main

import (
    a "github.com/ibmdb/go_ibm_db"
	_"database/sql"
    "fmt"
	"time"
)

func main(){
    con:="HOSTNAME=host;PORT=number;DATABASE=name;UID=username;PWD=password";
	pool:=a.Pconnect()
	
	//SetConnMaxLifetime will atake the value in MINUTES
	db:=pool.Open(con,"SetConnMaxLifetime=1","PoolSize=100")
    st, err:=db.Prepare("Insert into SAMPLE values('hi')")
    if err != nil{
        fmt.Println(err)
    }
	st.Query()
	
	db1:=pool.Open(con)
    st1, err:=db1.Prepare("Insert into SAMPLE values('hi1')")
    if err != nil{
        fmt.Println(err)
    }
	st1.Query()
	
	db1.Close()
	db.Close()
	pool.Display()
	pool.Release()
	pool.Display()
}
To run the sample:- go run example3.go
```
For Running the Tests:
======================

1) Put your connection string in the main.go file in testdata folder

2) Now run go test command (use go test -v command for details) 


