
## 1. Overview[](https://www.baeldung.com/linux/bash-daemon-script#overview)

In this tutorial, we’ll learn how to run a bash script as a daemon in the background.

## 2. Daemonizing Script as a Regular User[](https://www.baeldung.com/linux/bash-daemon-script#daemonizing-script-as-a-regular-user)

**We can make use of bash processes to run our script in the background.** We launch background tasks or processes with the _&_ operator.

For example, let’s daemonize this simple script which we’ve placed at _/home/baeldung/script.sh_:

```bash
# Print a message every 60 seconds.
while :; do
    sleep 60
    echo "Slept for 60 seconds!"
done
```

**Running the script with the [_nohup_](https://www.baeldung.com/linux/job-control-disown-nohup) command ensures that it’s not killed when we exit the terminal:**

```bash
$ nohup ./script.sh &
nohup: appending output to nohup.out
```

_nohup_ logs any output from the script into the _nohup.out_ file:

```bash
$ cat nohup.out 
Slept for 60 seconds!
Slept for 60 seconds!
...
```

We can close the terminal after starting the script.

**For killing the process, we can log the launched process’s PID into a file and use it with [_kill_](https://www.baeldung.com/linux/kill-background-process#1-thekillcommand):**

```bash
$ nohup ./script.sh &
nohup: appending output to nohup.out
$ echo $! > ./script.pid
$ cat script.pid
4946
```

The _$!_ variable stores the PID of the newly launched process. Therefore, we can just kill the script:

```bash
$ kill $(cat ./script.pid)
```

**While the method we’ve seen here is straightforward, it does require extra user intervention for the startup and shutdown of the script.**

## 3. Daemonizing Script System-Wide[](https://www.baeldung.com/linux/bash-daemon-script#daemonizing-script-system-wide)

We just learned how to run scripts in the background as regular users. Now, we’ll cover more sophisticated methods which allow us to control the script via the system’s service manager.

### 3.1. Using _systemd_[](https://www.baeldung.com/linux/bash-daemon-script#1-using-systemd)

**We can use _systemd_ [unit files](https://www.freedesktop.org/software/systemd/man/systemd.service.html) to launch our script at boot.** Unit files describe how the system should execute the given program.

Let’s create a unit file at _/etc/systemd/system/script_daemon.service_:

```bash
[Unit]
Description=Script Daemon

[Service]
Type=simple
User=baeldung
ExecStart=/home/baeldung/script.sh
Restart=on-failure

[Install]
WantedBy=default.target
```

We set the user that the script will run with the _User=_ option and the path to the script with the _ExecStart=_ option. The various options are documented in the [_systemd.exec_](https://www.freedesktop.org/software/systemd/man/systemd.exec.html) man page.

Now, let’s enable the service:

```swift
$ sudo systemctl daemon-reload
$ sudo systemctl enable script_daemon.service
Created symlink /etc/systemd/system/shutdown.target.wants/script_daemon.service → /etc/systemd/system/script_daemon.service
```

Here, we reload the _systemd_ config and enable our service using the [_systemctl_](https://www.man7.org/linux/man-pages/man1/systemctl.1.html) tool, which helps manage _systemd_ services.

**We can also add dependencies to various other services.**

For example, we might want to ensure that the network is available before starting our script. We do this with the _After=_ option:

```bash
[Unit]
Description=Script Daemon
After=network.target

[Service]
...
```

We can restart the service with _sudo systemctl restart script_daemon.service_. It will be auto-started at boot.

### 3.2. Using _/etc/rc.local_[](https://www.baeldung.com/linux/bash-daemon-script#2-using-etcrclocal)

The _/etc/rc.local_ file consists of commands that we want to run. Hence, we can just append the path to our script into this file:

```bash
$ echo "/home/baeldung/script.sh" >> /etc/rc.local
```

The script is executed as the _root_ user by default. For dropping privileges, we can execute the command as a different user with _[su](https://www.baeldung.com/linux/run-as-another-user):_

```shell
$ echo "su - baeldung /home/baeldung/script.sh" >> /etc/rc.local
```

**We should note that this command should be the last one run from _/etc/rc.local_ since it blocks execution.  
**

To make it non-blocking, we can use the _nohup_ command with the _&_ operator that we learned about earlier.

**Further, we should avoid this approach using the _/etc/rc.local_ file if _systemd_ is available as it is a legacy file.**

