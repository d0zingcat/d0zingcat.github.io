---
author: ["Me"]
title: 'K8s on Bare Metal: Secret Management'
date: 2024-11-20T16:04:17+08:00
categories: []
tags: []
draft: true
---

## Vault

[After scale](https://discuss.hashicorp.com/t/solved-vault-in-ha-with-raft-issue-joining/38574), we have to exec into the pod and manually join the leader.

```
kubectl -n vault exec -ti vault-1 -- vault operator raft join http://vault-0.vault-internal:8200
```

Then do the unseal manually inside the pod and the pod goes healthy.

```
kubectl -n vault exec -ti vault-1 -- vault operator unseal
```

For more raft related info, turn to [this](https://developer.hashicorp.com/vault/docs/commands/operator/raft).



Although we may assume that we can use the root token to log into the vault (cli or web), the security should be ensured as vault is the core of secrets. In the simplest way, we can create a user/pass admin to enhance security.

```
k -n vault exec -it vault-0 -- sh 
vault auth enable userpass
vault write auth/userpass/users/mitchellh password=foo policies=admins
```



## External-Secret



## How to keep simplicity of db envs

## Intro

> 💡 All of the content below is based on the assumption that we run our applications inside a Kubernetes (K8s) cluster and use environment variables to pass credentials to the applications.

Most of our services and applications utilize middleware, such as Redis, MySQL, and Doris. Due to individual developers’ personal taste or preference, there are numerous variations of a single item.

Let’s say we want to reference the secrets of MySQL in our pod. A common case is we need following info:

| Name     | Mandatory                                               |
| -------- | ------------------------------------------------------- |
| Host     | Y                                                       |
| Port     | Y                                                       |
| Username | Y                                                       |
| Password | Y, actually we don’t allow empty password on production |
| Database | Y                                                       |

In most cases, people do not hardcode these parameters in code (except for databases, as one application may control more than one database). That’s why we need to set these parameters in the pod environment.

## Problems

### **Problem 1: Should we choose single DSN or several param envs for database connection?**

Commonly saying, a database library should have two kinds of methods to build the connection.

(Pseudocode based on Golang/Python)

**Option 1: Using DSN**

```sql
package main

import (
	"database/sql"
	"fmt"
	"os"
	_ "github.com/go-sql-driver/mysql"
)

func main() {
	db, err := sql.Open("mysql", GetenvDefault("dsn", "user:password@/dbname"))
	if err != nil {
		panic(err.Error())
	}
	defer db.Close()

	err = db.Ping()
	if err != nil {
		panic(err.Error())
	}

	fmt.Println("Successfully connected!")
}

func GetenvDefault(param, defaultVal string) string {
	v := os.Getenv(param)
	if v == "" {
		v = defaultVal
	}
	return v
}
```

**Option 2: Using Params**

```python
import pymysql
import os

# Establish a connection to the database
connection = pymysql.connect(host=os.environ.get('host', 'localhost'),
                             user=os.environ.get('user', 'user'),
                             password=os.environ.get('password', 'password'),
                             db=os.environ.get('dbname', 'dbname'))

try:
    # Create a cursor object
    cursor = connection.cursor()

    # Execute a query
    cursor.execute("SELECT VERSION()")

    # Fetch all the rows
    for row in cursor.fetchall():
        print(row)

finally:
    # Close the connection
    connection.close()
```

A `DSN` should follow this format:

```
[mysql://]{user}:{password}@[tcp(]{host}:{port}[)/{database}?{OTHER_OPTIONS}]
```

The value inside a `[]` means it’s not a mandatory. Apparently, if we just got one DSN, we cannot easily extract the user, password, host and so on from the string. Although some libs may provider a quick option to do this extraction like `db.Parse({DSN})` to get those we want, we can’t ensure every our stack or language provides such method, and IMO, we should not create such an parsing method ourselves as the standard is quite complicated(you have to adapt to different schemas) and cost of maintenance is much way too high. We should not reinventing the wheel either.

So, based on **KISS**(Keep it simple stupid), we can simply choose **option 2**. There’s no easier method than an assemble function.

```go
// ...

func AssembleDSN() string {
	host := GetenvDefault("host", "host")
	port := GetenvDefault("port", "port")
	user := GetenvDefault("user", "user")
	pass := GetenvDefault("password", "password")
	database := GetenvDefault("database", "database")
	dsn := fmt.Sprintf("%s:%s@%s:%s/%s", user, pass, host, port, database)
	return dsn
}

func main() {
	db, err := sql.Open("mysql", AssembleDSN())
	// ...
}
```

You can make your own `dsn` according to different requirements of libs.

### **Problem 2: What names to choose?**

Let’s see one example of what secrets we used for db connection:

```go
DORIS_CONN_BASE_URL: AgC9QmtgZfNgX+f/X8Uff2
DORIS_DB_NAME_PROD: AgAVO6FSLEURjMYdH3+WbmZ
DORIS_DB_NAME_TEST: AgAT2o4qlTkA1XoB6S4DoCS
DORIS_DBNAME: AgBd1gnuNCeqYXG3DbTR+gHDf0sMs
DORIS_DSN: AgCzIurDW59/TieWcOdzIkjfRsf3bOJV
DORIS_DSN_BASE: AgCFzmj1Eb+xJtINO8mfHLl0gP/

DORIS_HOST: AgBZ4GmV4wMLF+LoR2bAfJQEOvNihPP
DORIS_PASSWORD: AgA1siIEIMivi3WkjETcwEjHtdE
DORIS_PORT: AgAu55XPRFcS4sGBy/RMWpigWeN/cSm
DORIS_USER: AgBmL83JNb32cGaoxv/dyS7MIMhzK6V

MYSQL_CONN_BASE_URL: AgCxE5WmGkUxdr0E44jb5O
MYSQL_DB_NAME_PROD: AgCqq0734+A/XpHTePRAvmr
MYSQL_DB_NAME_TEST: AgBxN/yrddH6X0g5sIK2fLw
MYSQL_DBNAME: AgC+4LsrWX5J44fBxlL9zzcs4+GTn

MYSQL_DSN: AgAxbQ/4lAT/1rqK9mWvX7Zr1hiE1Eco
MYSQL_DSN_BASE: AgAGkcPElnvb28QyZtgS4rbA6/H
MYSQL_HOST: AgCg/7rkTmd2EzwxsjUIEJl1VGadJxR
MYSQL_PASSWORD: AgAX2DxBlGN1j6ELi/IuseKtAZ9
MYSQL_PORT: AgCjk0GMZ5Iai9ZyOq7vQU07EzVuAGy
MYSQL_USER: AgAhpydURYUdHE2ltXjGSUEBh89VXCQ

STREAMLOAD_DBNAME: AgB/YUHRP0HEL9c5Vo2CBjXq
STREAMLOAD_HOST: AgAGLIjQl0OTZaCQMkdO1PVAkB
STREAMLOAD_PASSWORD: AgBPloxZ4e9vjkMlngnwQy
STREAMLOAD_PORT: AgAgZhYy8R7MeNQK6boFwnaRWR
STREAMLOAD_USER: AgADyMQYpukVwDUzb2Tr5iQJlj
```

One single host(e.g. MYSQL_HOST) duplicates in

```
MYSQL_CONN_BASE_URL
MYSQL_DSN
MYSQL_HOST
```

Awful! If we have to change each single item, we have to update the `DSN`, `BASE_URL` too. Not to say the DORIS one.

> FYI, for Doris, the query port and streamload port is not the same, but the others are same(host, user, password, db). So If we want to change one single item, we need to update more than 6 places. That’s why this proposal should be emitted and this standardization should be pushed.

Let’s remove `DSN` and `BASE_URLs` first, then we have to consider what names to use.

We have a bunch of complicated workloads that use MySQL as a business storage, Redis as cache, Doris as an analytics Engine. There are a lot apps connect MySQL, Redis and Doris simultaneously. To identify the two(they both have host, port, user etc), we should add prefix(`MYSQL_`, `DORIS_`, `REDIS_`).

Someone may choose `MYSQL_USERNAME` against `MYSQL_USER`, `MYSQL_DBNAME` against `MYSQL_DATABASE` and etc. There’s nothing more than preferences, there’s nothing about right or wrong. But from my point of view, I prefer to use the common used names. We can get our answers from Github. After a few code searches that I found the most common used names are below(and there are no confusions caused by the names).

| MySQL          | Doris          | Redis          |
| -------------- | -------------- | -------------- |
| MYSQL_HOST     | DORIS_HOST     | REDIS_HOST     |
| MYSQL_PORT     | DORIS_PORT     | REDIS_PORT     |
| MYSQL_USER     | DORIS_USER     | REDIS_USER     |
| MYSQL_PASSWORD | DORIS_PASSWORD | REDIS_PASSWORD |
| MYSQL_DATABASE | DORIS_DATABASE | REDIS_DATABASE |

### Problem 3: Control range, who to control what?

- Cluster admin is in charge of cluster management, including credentials. From this aspect, admin should control secrets.
- Cluster admin also need to maintain connection base info including host, port. That’s transparent to developers.
- Developers control database to connect

Let’s split the configs into two parts, one is configmap(shared among apps) and another is secret.

- secret(as user/pass is a pair, we need to set both user and password in the same secret)
  - user
  - password
- configmap
  - host
  - port
  - database(if needed, you can set databse in code or deployement)

Here’s one example:

- configmap(shared within namespace, controlled by admin)

```go
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-shared
data:
  MYSQL_HOST: mysql-primary
  MYSQL_PORT: '3306'
  MYSQL_DBNAME: 'meex_prod'
```

- secret(each app has its own, controlled by admin)

In vault, we only need to set following credentials.

```go
MYSQL_USER=
MYSQL_PASSWORD=
DORIS_USER=
DORIS_PASSWORD=
```

We copy the doris streamload user and password property through external-secrets. Other properties will be extracted automatically.

```go
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: core-db
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: core-db
    creationPolicy: Owner
  dataFrom:
    - extract:
        key: /prod/core-db
  data:
    - secretKey: STREAMLOAD_USER
      remoteRef:
        key: /prod/core-db
        property: DORIS_USER
    - secretKey: STREAMLOAD_PASSWORD
      remoteRef:
        key: /prod/core-db
        property: DORIS_PASSWORD
```

In the deployment, we only need to refer configmap and secret.

```go
apiVersion: apps/v1
kind: Deployment
metadata:
  name: core-orderinfoprocessor
spec:
  replicas: 0
  selector:
    matchLabels:
  template:
    metadata:
      labels:
    spec:
      imagePullSecrets:
        - name: ghcr
      containers:
        - name: orderinfoprocessor
          image: ghcr.io/meshee-team/meex-core-orderinfoprocessor
          envFrom:
            - configMapRef:
                name: mysql-shared
            - configMapRef:
                name: doris-shared
            - secretRef:
                name: core-db
          # if your database is not in secret
          env:
	          - name: MYSQL_DATABASE
		          value: xxx
```

### Problem 4: I just want another name, how?

You can use envFrom to refer to specific key inside configmap or secret.

```go
apiVersion: apps/v1
kind: Deployment
metadata:
  name: core-stats-service
spec:
  replicas: 1
  selector:
    matchLabels:
  template:
    metadata:
      labels:
    spec:
      imagePullSecrets:
        - name: ghcr
      containers:
        - name: web
          image: ghcr.io/meshee-team/meex-stats-service
          envFrom:
            - configMapRef:
                name: mysql-shared
            - configMapRef:
                name: doris-shared
            - configMapRef:
                name: redis-shared
            - secretRef:
                name: stats-db
            - secretRef:
                name: stats-redis
          env:
            - name: MEEX_MYSQL_HOST
              valueFrom:
                configMapKeyRef:
                  name: mysql-shared
                  key: MYSQL_HOST
            - name: MEEX_MYSQL_PORT
              valueFrom:
                configMapKeyRef:
                  name: mysql-shared
                  key: MYSQL_PORT
            - name: MEEX_MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: stats-db
                  key: MYSQL_USERNAME
```

## Conclusion

Through the solutions above, we only need to maintain just one copy of each database which contains base connection info(host, port) and one secret each app(actually, each app should have their own db account).

Admin sets up secret and developers just need to follow the agreements to read parameters and refer the configmaps/secrets in deployment without having to dig into the secret/configmap to check the keys. This measure will greatly improve the efficiency of admin and developers.
