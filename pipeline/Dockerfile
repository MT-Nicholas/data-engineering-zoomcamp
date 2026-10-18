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
# Previously copied files are re-copied; but this is normal and negligible
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