# Coriolis Unit Tests

## Environment in Docker container

Create an Ubuntu container as an unit test environment:

```bash
docker run -d -v $HOME/Work/coriolis:/coriolis --name coriolis_ubuntu_jammy ubuntu:jammy sleep infinity
```

NOTE: this assumes that Coriolis repository is cloned at `$HOME/Work/coriolis` location.

Exec into the container:

```bash
docker exec -it coriolis_ubuntu_jammy bash
```

Install the unit tests dependencies:

```bash
apt-get update
apt-get install -y python3-dev default-libmysqlclient-dev build-essential pkg-config tox git
```

Run the unit tests via tox:

```bash
cd /coriolis

tox -e py3 -v
```

This runs all the unit tests in parallel.

To run a specific unit test, we can use `pytest` in conjunction with `-k` parameter.

Install `pytest`:

```bash
pip3 install pytest
```

run a specific unit test:

```bash
pytest coriolis/tests -k test_set_task_error_os_morphing
```
