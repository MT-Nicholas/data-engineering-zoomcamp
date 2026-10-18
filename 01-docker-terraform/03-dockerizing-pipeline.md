---
prev_url: 02-virtual-environment.md
next_url: 04-postgres-docker.md
---
# Dockerizing the Pipeline

Now let's containerize the script. Create the following `Dockerfile` file.

Note that even though the defined file is a `Dockerfile`, all commands should be run using `podman` instead of `docker` , and they are compatible.


## Simple Dockerfile with pip

```dockerfile
# base Docker image that we will build on
FROM python:3.13.11-slim

# set up our image by installing prerequisites; pandas in this case
RUN pip install pandas pyarrow

# set up the working directory inside the container
WORKDIR /app
# copy the script to the container. 1st name is source file, 2nd is destination. `./` refers to current working directory, being `/app` as specified in previous step 
COPY pipeline.py pipeline.py ./

# define what to do first when the container runs
# in this example, we will just run the script
ENTRYPOINT ["python", "pipeline.py"]
```

**Explanation:**

- `FROM`: Base image (Python 3.13)
- `RUN`: Execute commands during build
- `WORKDIR`: Set working directory
- `COPY`: Copy files into the image
- `ENTRYPOINT`: Default command to run

### Build and Run

Let's build the image:

```bash
docker build -t test:pandas .
```

* The image name will be `test` and its tag will be `pandas`. If the tag isn't specified it will default to `latest`.
* `-t` denotes we are providing the tag (pandas).
* `.` denotes that the context directory for the build is the current directory - The docker build cannot access anything outside of this scope.

We can now run the container and pass an argument to it, so that our pipeline will receive it:

```bash
docker run -it test:pandas some_number
```

You should get the same output you did when you ran the pipeline script by itself.

> Note: To exit the iterative session, type exit in the bash
> Note: these instructions assume that `pipeline.py` and `Dockerfile` are in the same directory. The Docker commands should also be run from the same directory as these files.

## Dockerfile with uv

What about uv? Let's use it instead of using pip.

But first, let's add create a `.dockerignore` with these contents:

```dockerfile
.venv
__pycache__/
.git
```

```dockerfile
# Base image: official slim Python 3.13 (Debian, minimal extras, smaller image)
FROM python:3.13.11-slim

# Copy the uv binary from the official uv image into /bin, so `uv` is on PATH.
# Pinned to 0.12.3 to match uv_build's version range in pyproject.toml.
COPY --from=ghcr.io/astral-sh/uv:0.12.3 /uv /bin/

# Set the working directory inside the image (created if missing).
# All following COPY/RUN commands happen relative to /code.
WORKDIR /code

# Copy only the dependency definition files first.
# This layer (and the next) stay cached until one of these files changes.
COPY .python-version uv.lock pyproject.toml ./

# Create .venv and install dependencies exactly as pinned in uv.lock.
# --locked: fail if uv.lock is out of date with pyproject.toml
# --no-install-project: skip installing our own package (code isn't copied yet)
RUN uv sync --locked --no-install-project

# Copy the rest of the project (source code etc.) into /code.
# Files listed in .dockerignore (e.g. local .venv) are excluded.
COPY . .

# Run sync again, now that the source is present, to install the project itself.
# Dependencies are already installed, so this step is fast.
RUN uv sync --locked

# Put the venv's bin directory first on PATH, so `python` uses the venv
# (with all installed packages) without needing `uv run`.
ENV PATH="/code/.venv/bin:$PATH"

# Command run when the container starts. Any arguments to `podman run`
# are appended, e.g. `podman run test:pandas 10` -> python pipeline.py 10
ENTRYPOINT ["python", "pipeline.py"]
```

Note: We had some deviation from the course because uv automatically created a `/src/pipeline/__init__.py` file which was subsequently changing the treatment of our script as a project/package.