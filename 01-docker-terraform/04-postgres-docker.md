---
prev_url: 03-dockerizing-pipeline.md
next_url: 05-data-ingestion.md
---
# Running PostgreSQL with Docker

Now we want to do real data engineering. Let's use a Postgres database for that.

You can run a containerized version of Postgres that doesn't require any installation steps. You only need to provide a few _environment variables_ to it as well as a _volume_ for storing data.

*Note: Assuming we are still in `pipeline` as our working dir*

## Running PostgreSQL in a Container

Here's how to run the container:

```bash
docker run -it --rm \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v ny_taxi_postgres_data:/var/lib/postgresql \
  -p 5432:5432 \
  postgres:18
```

### Explanation of Parameters

* `-e` sets environment variables (user, password, database name)
* `-v ny_taxi_postgres_data:/var/lib/postgresql` creates a **named volume**
  * Docker manages this volume automatically (and seperately from the container)
  * Data persists even after container is removed because the volume is its own entity (VERY IMPORTANT !)
  * Volume is stored in Docker's internal storage
* `-p 5432:5432` maps port 5432 from container to host
* `postgres:18` uses PostgreSQL version 18 (latest as of Dec 2025)

### Alternative Approach - Bind Mount

First create the directory in a physical location in your current dir, then map it:

```bash
mkdir ny_taxi_postgres_data

docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql \
  -p 5432:5432 \
  postgres:18
```

### Named Volume vs Bind Mount

* **Named volume** (`name:/path`): Managed by Docker, easier (abstracted away, not accessible, less control) [Default Choice]
* **Bind mount** (`/host/path:/container/path`): Direct mapping to filesystem on host (physically accessible, more control to see/edit files directly, but easier to accidentally mess up)

- How to tell apart?
  - Bare name = Named Volume
  - Path starting with `/` or `./` = Bind Mount

### `--rm` and volumes
- `--rm` removes the container, **not** named volumes. Data survives and reattaches next run.
- `--rm` **does** delete anonymous (unnamed) volumes (i.e. if the -v argument did not specify a name (Named Volume) or path (Bind Mount), the volume would be deleted with the container

```bash
podman volume ls
podman volume inspect ny_taxi_postgres_data
podman volume rm ny_taxi_postgres_data
```

## Connecting to PostgreSQL

Once the container is running, we can log into our database with [pgcli](https://www.pgcli.com/).

Install pgcli:

```bash
uv add --dev pgcli
uv add --dev pgcli "psycopg[binary]"
```

The `--dev` flag marks this as a development dependency (not needed in production). It will be added to the `[dependency-groups]` section of `pyproject.toml` instead of the main `dependencies` section. This way, when we productionize into a docker container, these set of requirements would not be considered. Because we only need these requirements as part of development; for production we do no need, so we can skip, to keep production lightweight.

Now use it to connect to Postgres:

```bash
uv run pgcli -h localhost -p 5432 -u root -d ny_taxi
```

* `uv run` executes a command in the context of the virtual environment
* `-h` is the host. Since we're running locally we can use `localhost`.
* `-p` is the port.
* `-u` is the username.
* `-d` is the database name.
* The password is not provided; it will be requested after running the command.

When prompted, enter the password: `root`

## Basic SQL Commands

Try some SQL commands:

```sql
-- List tables
\dt

-- Create a test table
CREATE TABLE test (id INTEGER, name VARCHAR(50));

-- Insert data
INSERT INTO test VALUES (1, 'Hello Docker');

-- Query data
SELECT * FROM test;

-- Exit
\q
```
