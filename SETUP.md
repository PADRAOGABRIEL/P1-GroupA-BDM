# 🔧 Setup

This guide shows how to start the project environment using Docker.

## Prerequisites

- Docker Desktop installed and running
- Docker Compose available

## 1) Clone/Open the project

Open a terminal in the project root folder (where `docker-compose.yml` is located).

## 2) Start the services

```bash
docker compose up --build
```

## 3) Access Jupyter and Spark UI

With the container running, open:

- Jupyter Lab: http://localhost:8888
- Spark UI: http://localhost:4040 (available when a Spark job is running)

Note: in this project, the Jupyter token is disabled in `docker-compose.yml`.

## 4) Run the notebook

In Jupyter Lab, open:

- `notebooks/etl.ipynb`

## Useful commands

Stop containers:

```bash
docker compose down
```

Stop and remove volumes (reset persisted data, if any):

```bash
docker compose down -v
```

Start in background:

```bash
docker compose up -d --build
```
