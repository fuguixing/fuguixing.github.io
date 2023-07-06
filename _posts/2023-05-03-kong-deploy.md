---
layout:       post
title:        "Deploying Kong using Docker Compose"
author:       "Fugui"
header-style: text
catalog:      true
tags:
    - API gateway
    - Kong
---

> Kong

In today's world, APIs are an essential part of any software system. They enable software components to interact with each other, making it easier to build complex systems. [Kong](https://github.com/Kong/kong) is a popular open-source API gateway that simplifies the process of managing APIs. In this blog post, we will explore how to deploy Kong using Docker Compose.

Docker Compose is a tool that simplifies the process of deploying multi-container Docker applications. Here is an example Docker Compose file for deploying Kong:

```shell
version: '3'

services:

  kong-database:
    image: postgres:9.6
    container_name: kong-database
    ports:
      - 5432:5432
    environment:
      - POSTGRES_USER=kong
      - POSTGRES_DB=kong
      - POSTGRES_PASSWORD=kong
    networks:
      - kong-net
    volumes:
      - "db-data-kong-postgres:/var/lib/postgresql/data"

  kong-migrations:
    image: kong
    environment:
      - KONG_DATABASE=postgres
      - KONG_PG_HOST=kong-database
      - KONG_PG_PASSWORD=kong
      - KONG_CASSANDRA_CONTACT_POINTS=kong-database
    command: kong migrations bootstrap
    restart: on-failure
    networks:
      - kong-net
    depends_on:
      - kong-database

  kong:
    image: kong:2.6.0-alpine
    container_name: kong
    environment:
      - LC_CTYPE=en_US.UTF-8
      - LC_ALL=en_US.UTF-8
      - KONG_DATABASE=postgres
      - KONG_PG_HOST=kong-database
      - KONG_PG_USER=kong
      - KONG_PG_PASSWORD=kong
      - KONG_CASSANDRA_CONTACT_POINTS=kong-database
      - KONG_PROXY_ACCESS_LOG=/dev/stdout
      - KONG_ADMIN_ACCESS_LOG=/dev/stdout
      - KONG_PROXY_ERROR_LOG=/dev/stderr
      - KONG_ADMIN_ERROR_LOG=/dev/stderr
      - KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl
    restart: on-failure
    ports:
      - 8000:8000
      - 8443:8443
      - 8001:8001
      - 8444:8444
    links:
      - kong-database:kong-database
    networks:
      - kong-net
    depends_on:
      - kong-migrations

  konga:
    image: pantsel/konga
    ports:
      - 1337:1337
    links:
      - kong:kong
    container_name: konga
    environment:
      - NODE_ENV=production

volumes:
  db-data-kong-postgres:

networks:
  kong-net:
    external: false
```

— Fugui，Data engineer、Machine learning Engineer、Software Engineer.
