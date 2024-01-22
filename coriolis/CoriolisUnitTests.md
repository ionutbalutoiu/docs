# Coriolis Unit Tests

## Environment in Docker container

Create a dev container as an unit test environment:

```bash
docker run -d -v $HOME/Work/coriolis:/coriolis --name coriolis_python_3.10 python:3.10-bullseye sleep infinity
```

NOTE: this assumes that Coriolis repository is cloned at `$HOME/Work/coriolis` location, and the tested Python version is `3.10`.

Exec into the container:

```bash
docker exec -it coriolis_python_3.10 bash
```

Install the unit tests dependencies:

```bash
apt-get update
apt-get install -y default-libmysqlclient-dev build-essential pkg-config git

pip3 install tox
```

Run the unit tests via tox:

```bash
cd /coriolis

tox -e py3 -v
```

This runs all the unit tests in parallel.

To run a specific unit test, we can use `pytest` in conjunction with `-k` parameter.

Install `pytest` in the tox environment:

```bash
source /coriolis/.tox/py3/bin/activate

pip3 install pytest
```

run a specific unit test:

```bash
pytest coriolis/tests -k test_set_task_error_os_morphing
```
