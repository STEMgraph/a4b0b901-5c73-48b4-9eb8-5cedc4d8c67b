<!---
{
  "id": "a4b0b901-5c73-48b4-9eb8-5cedc4d8c67b",
  "depends_on": ["a0f6c77d-9645-4e6c-80dc-a80608786266","2d1d315d-bb92-48c0-b19f-19529a45e5ff","b9e34253-61e2-4fb7-bd36-388c370fa765"],
  "author": "Stephan Bökelmann",
  "first_used": "2025-05-06",
  "keywords": ["systemd", "python", "http.server", "virtualenv", "unit files"]
}
--->

# Running Python Services with `systemd`

> In this exercise you will learn how to run Python-based tasks as persistent services using `systemd`. Furthermore we will explore how to define services that use a Python virtual environment.

### Introduction

Linux services are not limited to compiled daemons or scripts written in Bash. Many tasks you’d like to run on boot — such as serving a static website or continuously monitoring a directory — are perfectly suitable for implementation in Python. With `systemd`, you can easily manage these Python scripts as full-fledged services.

In this exercise, you will first run a static website using Python’s built-in HTTP server under `~/static_website/`. Then, you'll set up a second service that executes a Python script using a dedicated virtual environment. This teaches you how to use `ExecStart` to activate environments and keep Python services isolated and maintainable. Learning how to correctly configure environment paths and permissions is a key takeaway.

By the end of this exercise, you will have created two services that are automatically started on boot, logged by the journal, and can be restarted or stopped on demand using standard `systemctl` commands.

### Further Readings and Other Sources

* [Python HTTP server module](https://docs.python.org/3/library/http.server.html)
* [DigitalOcean: How To Use Systemctl](https://www.digitalocean.com/community/tutorials/how-to-use-systemctl-to-manage-systemd-services-and-units)
* [Arch Wiki: Python Virtual Environments](https://wiki.archlinux.org/title/Python/Virtual_environment)
* [Red Hat Developer Blog: Run your Python application as a systemd service](https://developers.redhat.com/blog/2021/09/21/how-to-run-your-python-applications-as-a-systemd-service)

---

### Tasks

#### 1. Python HTTP Server as a Service

Ensure the directory exists and has content:

```sh
mkdir -p ~/static_website
echo "<h1>Hello from systemd!</h1>" > ~/static_website/index.html
```

Create a unit file at `~/.config/systemd/user/httpserver.service`:

```ini
[Unit]
Description=Python Simple HTTP Server
After=network.target

[Service]
ExecStart=/usr/bin/python3 -m http.server 8080 --directory=/home/YOUR_USERNAME/static_website
Restart=always

[Install]
WantedBy=default.target
```

Replace `YOUR_USERNAME` with your actual username.

Then enable and start the service:

```sh
systemctl --user daemon-reload
systemctl --user enable httpserver.service
systemctl --user start httpserver.service
```

Check status and access the site at [http://localhost:8080](http://localhost:8080).

#### 2. Python Script with Virtual Environment

Create the Python project:

```sh
mkdir -p ~/my_py_service
cd ~/my_py_service
python3 -m venv venv
source venv/bin/activate
pip install requests
```

Create `script.py`:

```python
# ~/my_py_service/script.py
import time
import requests

while True:
    r = requests.get("https://example.com")
    with open("output.txt", "a") as f:
        f.write(f"Fetched at {time.ctime()}: {r.status_code}\n")
    time.sleep(300)
```

Create a unit file at `~/.config/systemd/user/mypyservice.service`:

```ini
[Unit]
Description=Python Script with venv
After=network.target

[Service]
WorkingDirectory=/home/YOUR_USERNAME/my_py_service
ExecStart=/home/YOUR_USERNAME/my_py_service/venv/bin/python script.py
Restart=on-failure

[Install]
WantedBy=default.target
```

Again, replace `YOUR_USERNAME` with your actual username.

Enable and start the service:

```sh
systemctl --user daemon-reload
systemctl --user enable mypyservice.service
systemctl --user start mypyservice.service
```

---

### Questions

1. Why is `--directory` used in the `http.server` service?
2. What is the purpose of a virtual environment in Python?
3. How does `systemd` know to restart a failed service?
4. How does `systemctl --user` differ from system-wide service management?

---

### Advice

Running Python applications as services offers convenience and reliability. Using `systemctl --user` avoids the need for root access and is ideal for per-user services. Always use absolute paths in your unit files and test your script manually before deploying it under `systemd`. For virtual environments, ensure paths are correct and environment dependencies are frozen for reproducibility. If your script needs to communicate with other system components, consider extending this with environment variables or socket activation in a future exercise.
