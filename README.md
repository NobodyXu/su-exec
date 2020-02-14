# su-exec
switch user and group id and exec without passwd, can be used to replace sudo when building container.

## Purpose

This is a simple tool that will simply execute a program with different
privileges. The program will be exceuted directly and not run as a child,
like su and sudo does, which avoids TTY and signal issues (see below).

Notice that su-exec depends on being run by the root user, non-root
users do not have permission to change uid/gid, or you need to setuid on it.

## Usage

```shell
su-exec user-spec command [ arguments... ]
```

`user-spec` is either a user name (e.g. `nobody`) or user name and group
name separated with colon (e.g. `nobody:ftp`). Numeric uid/gid values
can be used instead of names. Example:

```shell
$ su-exec apache:1000 /usr/sbin/httpd -f /opt/www/httpd.conf
```

### Replace `sudo`

If you compile softwares in a container, you probably need `sudo`, since compiling with root may not be
a good idea and some `Makefile` like `lede` even forbidden building as root.

However, `sudo` is such a overkill for unattented auto-build of a container since
 - It requires dependencies to be installed.
 - You need to configure `sudo` to allow password-less `sudo` for your user
 - You cannot run `sudo apt-get remove -y sudo` to uninstall, you have to somewhat switch to root user without `sudo`
 to uninstall it.

So how to replace `sudo` with `su-exec` for containers? Simple, just execute the following lines with `root`:

```
cd /usr/local/bin/

wget https://github.com/NobodyXu/su-exec/releases/download/v0.2.1/su-exec
chmod a+xs su-exec
```

Removing `su-exec` is pretty simple:

```
su-exec root:root rm /usr/local/bin/su-exec
```

## TTY & parent/child handling

Notice how `su` will make `ps` be a child of a shell while `su-exec`
just executes `ps` directly.

```shell
$ docker run -it --rm alpine:edge su postgres -c 'ps aux'
PID   USER     TIME   COMMAND
    1 postgres   0:00 ash -c ps aux
   12 postgres   0:00 ps aux
$ docker run -it --rm -v $PWD/su-exec:/sbin/su-exec:ro alpine:edge su-exec postgres ps aux
PID   USER     TIME   COMMAND
    1 postgres   0:00 ps aux
```

## Possible Vulnerability

`su-exec` is not like `sudo` but more like `su`, it does not modify any environment variables other than `HOME`, which might be undesirable.
To workaround, use `su-exec env var=val command arg`.

## Why reinvent gosu?

This does more or less exactly the same thing as [gosu](https://github.com/tianon/gosu)
but it is only 10kb instead of 1.8MB.

