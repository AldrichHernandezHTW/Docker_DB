# OBETA MySQL

A reproducible MySQL local environment using Docker Compose.

This project creates two databases automatically on first startup:

- `OBETA_STAGING`
- `OBETA_PRODUCTION`

It also creates an application user using the credentials defined in the `.env` file and grants access to both databases.

---

## Requirements

- Docker Desktop (recommended for macOS and Windows) - https://www.docker.com/products/docker-desktop/

---

## Project Structure

```text
obeta-mysql/
├── .env
├── .env.example
├── .gitignore
├── compose.yaml
├── README.md
└── mysql/
    ├── conf.d/
    │   └── my.cnf
    └── init/
        └── 001-init.sh
````

---

## Environment Variables

Create a local `.env` file based on `.env.example`.

Example:

```env
MYSQL_PORT=3306

MYSQL_ROOT_PASSWORD=Root123!
MYSQL_USER=obeta_user
MYSQL_PASSWORD=User123!
```

---

## File Permissions

The initialization script must be executable before starting the container.

Run:

```bash
chmod +x mysql/init/001-init.sh
```

You can verify the permissions with:

```bash
ls -l mysql/init/001-init.sh
```

It should show executable permissions, for example:

```bash
-rwxr-xr-x
```

If you want to make sure Git preserves the executable bit, run:

```bash
git update-index --chmod=+x mysql/init/001-init.sh
```

---

## Setup

1. Clone the repository.
2. Copy the example environment file:

```bash
cp .env.example .env
```

3. Update the values in `.env` if needed.
4. Make the init script executable:

```bash
chmod +x mysql/init/001-init.sh
```

5. Start the project:

```bash
docker compose up -d
```

---

## Check Container Status

```bash
docker compose ps
docker compose logs -f db
```

Wait until MySQL is ready to accept connections.

---

## Clean Restart

If you already started the container before and want to re-run the initialization scripts, remove the volume and start again:

```bash
docker compose down -v
docker compose up -d
```

This is necessary because MySQL initialization scripts inside `/docker-entrypoint-initdb.d` only run the first time the data directory is created.

---

## Databases Created Automatically

On first initialization, the container creates:

* `OBETA_STAGING`
* `OBETA_PRODUCTION`

It also creates the application user defined in `.env` and grants privileges on both databases.

---

## Connect from MySQL Workbench

Use the following connection settings:

### Application user

* **Hostname:** `127.0.0.1`
* **Port:** `3306`
* **Username:** value of `MYSQL_USER`
* **Password:** value of `MYSQL_PASSWORD`

### Root user

* **Hostname:** `127.0.0.1`
* **Port:** `3306`
* **Username:** `root`
* **Password:** value of `MYSQL_ROOT_PASSWORD`

---

## Verify the Databases

Open a MySQL shell inside the container:

```bash
docker exec -it mysql-obeta-db mysql -uroot -p
```

Then run:

```sql
SHOW DATABASES;
SELECT user, host FROM mysql.user;
```

You should see:

* `OBETA_STAGING`
* `OBETA_PRODUCTION`
* the user defined in `.env`

---

## Main Files

### `.env`

Stores local environment variables and credentials.

### `.env.example`

Template file for other developers.

### `compose.yaml`

Defines the MySQL service, ports, volumes, and healthcheck.

### `mysql/init/001-init.sh`

Initialization script that:

* creates both databases
* creates the application user
* grants privileges on both databases

### `mysql/conf.d/my.cnf`

MySQL custom configuration for charset and collation.

---

## Useful Commands

### Start services

```bash
docker compose up -d
```

### Stop services

```bash
docker compose down
```

### Stop and remove volumes

```bash
docker compose down -v
```

### View logs

```bash
docker compose logs -f db
```

### Open MySQL shell

```bash
docker exec -it mysql-obeta-db mysql -uroot -p
```

---

## Notes

* Do not commit the real `.env` file.
* Commit `.env.example` instead.
* If you change the initialization logic, use a clean restart with `docker compose down -v`.
* Keep `001-init.sh` executable so Docker can run it correctly during initialization.
