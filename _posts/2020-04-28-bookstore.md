---
title: Bookstore - Golang based scalable API's 
author: Aman Gupta
date: 2020-04-28 07:10:00 +0800
categories: [Development, Golang]
tags: [golang, api, cassandra, elasticsearch, mysql, gorilla-mux, go-gin]
---

This project consists of 3 standalone API's and 2 shared libraries all written in golang:1.14.  
The API's are `users-api`, `oauth-api`, `items-api`  
The shared libraries are `oauth-client`, `utils`  
It explores three different types of databases which are `MySql`, `Cassandra` and `elasticSearch`, and uses the most starred `go clients` for each of them.

The bookstore project offers following features:
* Create, Update, Delete, Get User.
* User data protection with `X-Public` header check.
* Issue access token if user exists.
* Use access token to communicate with main items api (validating token).
* Create, Get Item. 

The project structure of the bookstore api's follows strict MVC architecture principles, which makes the code highly readable and configurable. 

> For shipping the api's, we use docker. 

Below is the flow of request from client:
![Desktop View]({{ "/assets/img/sample/client-api.png" | relative_url }})

***

## Components

***
### [Bookstore_users-api](https://github.com/theguptaji/bookstore_users-api)

`Users-api` revolves around the domain `user`
whose definition is as follows:

```go
type User struct {
	Id          int64  `json:"id"`
	FirstName   string `json:"first_name"`
	LastName    string `json:"last_name"`
	Email       string `json:"email"`
	DateCreated string `json:"date_created"`
	Status      string `json:"status"`
	Password    string `json:"password"`
}
```

It offers the following endpoints:

|Http_Method|Endpoint|Function|
|:---|:--|---:|
|GET | "/ping"  | returns "pong"
|POST | "/users" | creates new user
|GET | "/users/:user_id" | get user by user_id
|PUT | "/users/:user_id" | update user with user_id
|PATCH | "/users/:user_id" | update user with user_id (ignore empty)
|DELETE| "/users/:user_id" | delete user with user_id
|GET | "/internal/users/search" | returns active user
|POST | "/users/login" | if user exists, returns user

It uses a Mysql database for its persistance layer. After setting up MySql we need to make a database named `userDB`, and create a table `users` with the following schema.

```
    +--------------+-------------+------+-----+---------+----------------+
    | Field        | Type        | Null | Key | Default | Extra          |
    +--------------+-------------+------+-----+---------+----------------+
    | id           | bigint      | NO   | PRI | NULL    | auto_increment |
    | first_name   | varchar(45) | YES  |     | NULL    |                |
    | last_name    | varchar(45) | YES  |     | NULL    |                |
    | email        | varchar(45) | NO   | UNI | NULL    |                |
    | date_created | varchar(45) | NO   |     | NULL    |                |
    | status       | varchar(45) | NO   |     | NULL    |                |
    | password     | varchar(32) | NO   |     | NULL    |                |
    +--------------+-------------+------+-----+---------+----------------+
```

***
### [Bookstore_oauth-api](https://github.com/theguptaji/bookstore_oauth-api)

`Oauth-api` revolves around the domain `access-token` and `access-token-request`
whose definition is as follows:

```go
type AccessToken struct {
	AccessToken string `json:"access_token"`
	UserId      int64  `json:"user_id"`
	ClientId    int64  `json:"client_id"`
	Expires     int64  `json:"expires"`
}
```

```go
type AccessTokenRequest struct {
	GrantType string `json:"grant_type"`
	Scope     string `json:"scope"`

	// Used for password grant type
	Username string `json:"username"`
	Password string `json:"password"`

	// Used for client_credentials grant type
	ClientId     string `json:"client_id"`
	ClientSecret string `json:"client_secret"`
}
```

It offers the following endpoints:

|Http_Method|Endpoint|Function|
|:---|:--|---:|
|GET | "/oauth/access_token/:access_token_id" | returns access token
|POST | "/oauth/access_token" | creates new access token

If an oauth client sends a GET request with an **access_token_ID**, the API checks it the access token exists and then returns the corresponding AccessToken struct(id, userid, client_id, expires).  

If an oauth client sends POST request with **access_token_request** , the API creates a new access_token with corresponding user

It uses a Cassandra database for its persistance layer. After setting up Cassandra we need to make a keyspace named `oauth`, and create a table `access_tokens` with the following schema.

```sql
    CREATE TABLE oauth.access_tokens (
        access_token text PRIMARY KEY,
        client_id bigint,
        expires bigint,
        user_id bigint
    )
```

***

### [Bookstore_items-api](https://github.com/theguptaji/bookstore_items-api)

`items-api` revolves around the domain `Item` and `queries`.
whose definition is as follows:

```go
    type Item struct {
        Id                string      `json:"id"`
        Seller            int64       `json:"seller"`
        Title             string      `json:"title"`
        Description       Description `json:"description"`
        Pictures          []Picture   `json:"pictures"`
        Video             string      `json:"video"`
        Price             float32     `json:"price"`
        AvailableQuantity int         `json:"available_quantity"`
        SoldQuantity      int         `json:"sold_quantity"`
        Status            string      `json:"status"`
    }
```

```go
type EsQuery struct {
	Equals []FieldValue `json:"equals"`
}

type FieldValue struct {
	Field string `json:"field"`
	Value interface{} `json:"value"`
}
```

It offers the following endpoints:

|Http_Method|Endpoint|Function|
|:---|:--|---:|
|GET | "/ping"  | returns "pong"
|POST | "/items" | creates new item
|GET | "/items/{id}" | get item by item_id
|POST | "/items/search" | search item with queries


It uses Elasticsearch for its persistance layer. After setting up Elasticsearch, we need to configure the `shards` for out database, which can be done by making a POST request to database i.e. `127.0.0.1:9200/items` with following body:  

```json
    {
        "settings": {
            "index" : {
                "number_of_shards" : 4,
                "number_of_replicas" : 2
            }
        }
    }
```

***
### [Bookstore_utils-go](https://github.com/theguptaji/bookstore_utils-go)

This is a shared package for our bookstore project which consists of custom `logger` and `error` interface. It is not only limited to this project, and can be used in any golang project.

***
### [Bookstore_oauth-go](https://github.com/theguptaji/bookstore_oauth-go)

A client package to interact with Oauth API, it can simply be imported in any API and use Oauth functions like validation, login_request etc.
***
