[How to automatically restart a linux background process if it fails?](https://superuser.com/questions/507576/how-to-automatically-restart-a-linux-background-process-if-it-fails)

The easiest way would be to add it to [/etc/inittab](http://www.computerworld.com/article/2693438/unix-how-to-the-linux-etc-inittab-file.html), which is designed to do this sort of thing:

> **respawn** If the process does not exist, start the process. Do not wait for its termination (continue scanning the /etc/inittab file). Restart the process when it dies. If the process exists, do nothing and continue scanning the /etc/inittab file.

For example, you could do this:

```bash
# Run my stuff
myprocess:2345:respawn:/bin/myprocess
```

Buildroot has three possible init systems, so there are three ways to do this:

## BusyBox `init`

With this, one adds an entry to `/etc/inittab`.

```bash
::respawn:/bin/myprocess
```

Note that BusyBox `init` has an idiosyncratic `/etc/inittab` format. The second field is meaningless, and the first field *is not an ID* but a device basename.

## Linux "System V" `init`

Again, one adds an entry to `/etc/inittab`.

```bash
myprocess:2345:respawn:/bin/myprocess
```

## `systemd`

One writes a unit file in, say, `/etc/systemd/system/myprocess.service`:

```bash
[Unit]
Description=My Process

[Service]
ExecStart=/bin/myprocess
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable this to autostart at bootup with:

```bash
systemctl enable myprocess.service
```

Start it manually with:

```bash
systemctl start myprocess.service
```

## Further reading

- "[3.1.3 init system](http://buildroot.uclibc.org/downloads/manual/manual.html#_init_system)". *The Buildroot user manual*.

What about creating a subshell with a loop that calls constantly the same process?

If it ends, the next iteration of the loop goes on and starts it again.

```bash
(while true; do 
    /bin/myprocess
done) &
```

If the subshell dies, it's over though. The only possibility in that case would be to create another process (I'll call it necromancer) that checks whether yourprocess is alive, start it if it isn't and run this necromancer with cron, so that you can check that regularly.

Next step would be wondering what could happen if cron dies, but at some point you should feel safe and stop worrying.

You could make use of [Monit](https://mmonit.com/monit/) . It's really easy to use and quite flexible. See for example this configuration for restarting the Tomcat process on failure.

```bash
check process tomcat with pidfile /var/run/tomcat.pid
   start program = "/etc/init.d/tomcat start"
   stop  program = "/etc/init.d/tomcat stop"
   if failed port 8080 type tcp then restart
```

It also has a lot of [configuration examples](https://mmonit.com/wiki/Monit/ConfigurationExamples) for many use cases.

You may use [restarter](https://bitbucket.org/sivann/restarter)

```bash
start)
   restarter -c /bin/myprocess &
stop)
   pkill -f myprocess
```

On newer systems use systemd which solves all those trivial issues

In my case, as a quick-fix, I modified and used the solution of @Trylks to wrap the program I was launching. I wanted it to end only on clean exit.

Should run in most shells:

```bash
#!/bin/sh

echo ""
echo "Use: $0 ./program"
echo ""

#eg="/usr/bin/apt update"

echo "Executing $1 ..."

EXIT_CODE=1
(while [ $EXIT_CODE -gt 0 ]; do
    $1
    # loops on error code: greater-than 0
    EXIT_CODE=$?
done) &
```

(Edit): Sometimes programs hang without quitting, for no apparent reason. (Yes, of course there's always a reason but it can take a lot of time and effort to find it, particularly if it's not your own code.)

The problem I had was that the process (a Python server) was hanging from time to time, so I needed to regularly kill and restart it. I did it with a cron task that runs every couple of hours. Here's the shell script:

```bash
#!/bin/sh

# This cron script restarts the server
# Look for a running instance of myapp.py
p=$(ps -eaf | grep "[m]yapp.py")
# Get the second item; the process number
n=$(echo $p | awk '{print $2}')
# If it's not empty, kill the process
if [ "$n" ]
then
   kill $n
fi
# Start a new instance
python3 myapp.py
```

This answer is useful

0

Save this answer.Show activity on this post.

The process can be launched with `until loop` as it's been answered here: [How do I write a bash script to restart a process if it dies?](https://stackoverflow.com/a/697064/9066900). Until loop works with the requirement to run it the background, see sample:

```bash
#!/bin/sh
(until /bin/myprocess; do
    echo "'/bin/myprocess' crashed with exit code $?. Restarting..." >&2
    sleep 1
done) & 
```

Comparing to the @Trylks answer with `while` loop the last couldn't restart process when it's written in `entrypoint.sh` script of a `Dockerfile`. Despite it the `until loop` is working there too..
