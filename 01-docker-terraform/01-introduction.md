---
next_url: 02-virtual-environment.md
---

# Introduction to Docker

Docker is a _containerization software_ that allows us to isolate software in a similar way to virtual machines but in a much leaner way.

A Docker image is a _snapshot_ of a container that we can define to run our software, or in this case our data pipelines. By exporting our Docker images to Cloud providers such as Amazon Web Services or Google Cloud Platform we can run our containers there.


## Why Docker?

Docker provides the following advantages:

- Reproducibility: Same environment everywhere
- Isolation: Applications run independently
- Portability: Run anywhere Docker is installed

They are used in many situations:

- Integration tests: CI/CD pipelines
- Running pipelines on the cloud: AWS Batch, Kubernetes jobs
- Spark: Analytics engine for large-scale data processing
- Serverless: AWS Lambda, Google Functions

Optionally, we may use Podman, which is an open-source equivalent.

## Basic Docker Commands

Check Docker version:

```bash
docker --version
podman --version
```

Run a simple container:

```bash
docker run hello-world
podman run hello-world
```

Run something more complex:

```bash
docker run ubuntu
podman run
```

Nothing happens. Need to run it in `-it` (iterative) mode:

```bash
docker run -it ubuntu
podman run -it ubuntu
```

To exit the session, type exit and enter.

We don't have `python` there so let's install it:

```bash
apt update && apt install python3
python3 -V
```

## Stateless Containers

Important: Docker containers are stateless - any changes done inside a container will NOT be saved when the container is killed and started again. We can pause the container and restart it later (this will preserve state), but every time we run a container from the same image, it starts from the same base state.

When you exit the container and use it again, the changes are gone:

```bash
docker run -it ubuntu # or podman, for all commands
python3 -V
```

This is good, because it doesn't affect your host system. Let's say you do something crazy like this:

```bash
docker run -it ubuntu
rm -rf / # don't run it on your computer!
```

Next time we run it, all the files are back.

## Managing Containers

But, this is not _completely_ correct. The state is saved somewhere. We can see stopped containers:

```bash
docker ps -a
```

We can restart one of them, but we won't do it, because it's not a good practice. They take space, so let's delete them:

```bash
docker rm $(docker ps -aq)
```

Next time we run something, we add `--rm`: (to delete automatically on exit)

```bash
docker run -it --rm ubuntu
```

## Different Base Images

There are other base images besides `hello-world` and `ubuntu`. For example, for Python see below. Note the naming convention: `<name>:<label>`

```bash
docker run -it --rm python:3.9.16
# add -slim to get a smaller version of python
```

This one starts `python`. If we want bash, we need to overwrite `entrypoint`:

```bash
docker run -it \
    --rm \
    --entrypoint=bash \
    python:3.9.16-slim

ls .
python3 --version
```

## Volumes

So, we know that with docker we can restore any container to its initial state in a reproducible manner. But what about data? A common way to do so is with _volumes_.

The durable relationship is that host files (on local machine) can be shared with a container (the isolated component) through a mounted volume (one-way sharing):

![Local host files can be shared with a Docker container through a mounted volume.](images/docker-volume-mapping.png)

Let's create some data in `test`:

```bash
mkdir test
cd test
touch file1.txt file2.txt file3.txt # Create files
echo "Hello from host" > file1.txt # Type text into one of the files
cd ..
```

Now let's create a simple script `test/list_files.py` that shows the files in the folder:

```python
from pathlib import Path

current_dir = Path.cwd()
current_file = Path(__file__).name

print(f"Files in {current_dir}:")

for filepath in current_dir.iterdir():
    if filepath.name == current_file:
        continue

    print(f"  - {filepath.name}")

    if filepath.is_file():
        content = filepath.read_text(encoding='utf-8')
        print(f"    Content: {content}")
```

Now let's map this to a Python container.

The `-v` denotes the volumes. It requires a mapping of `local_dir:container_dir`. Below we are mapping `test` from current working directory to `app/test` in our docker image's directory

```bash
docker run -it \
    --rm \
    -v $(pwd)/test:/app/test \
    --entrypoint=bash \
    python:3.9.16-slim
```

Inside the container, run:

```bash
cd /app/test
ls -la
cat file1.txt
python list_files.py
```

You'll see the files from your host machine are accessible in the container!
