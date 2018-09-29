# Docker images for XRootD with SciTokens Support

This repository contains a Docker image with XRootD configured to
serve HTTP with SciTokens support.

To run, the host needs to have the following:

* `/etc/grid-security` (or similar) with the traditional "grid layout" of a `certificates`
  subdirectory with CAs and `hostcert.pem` / `hostkey.pem` as a valid hostcert.
* A directory containing files to export.

These directories will need to be bind-mounted into the container.

## Setting up Docker
```
$ sudo apt-get install apt-transport https ca-certificates curl software-properties-common
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable" // <- currently testing on Ubuntu
$ sudo apt-get update
$ sudo apt-get install docker-ce
```
Ensuring you have Docker running fine, run:
```
$ apt-cache policy docker-ce
$ sudo systemctl status docker
```
To add yourself to the docker group so you won't have to sudo all the time:
```
$ sudo usermod -aG docker USERNAME_HERE
$ su USERNAME_HERE
$ id -nG // <- to check
```
## Testing your instance
```
$ git clone https://github.com/scitokens/docker-xrootd-scitokens.git
```
cd into the directory and run
```
$ docker build .
```
To build a new docker in the directory 
Running a quick 
```
$ docker images
```
to see a row with:
```
REPOSITORY    TAG     IMAGE ID    CREATED     SIZE
<none>        <none>  rndmid      x sec ago   446MB
```
And now we run the docker image:
```
$ docker run -p 443:1094 --volume /etc/grid-security/:/etc/grid-security --volume /tmp/exportfiles:/export RNDMID_HERE 
```
Now that will hang your terminal, so we should open up a new terminal...
### In a new terminal...
In your ~USER directory, run
```
$ . venv/bin/activate
$ cd /tmp/exportfiles
$ mkdir demo
$ cd demo
$ chmod 777 .
$ ls -la // <- for sanity
$ cd ~USER
$ python test_http.py "https://FQDN/demo/nameofyourfile.txt" "write:/" "read:/"
```
Now it should send! Your "Got Token" is your encoded SciToken! Run this to make sure you did it correctly:
```
$ ls -lh /tmp/exportfiles/demo/
```
And it should give you something like:
```
total 51M
-rw-r--r-- 1 999 998 51M Sep 27 14:54 blah
```
And if you look at your other terminal with the running docker image, you should see the log of an XrootdBridge with your hostname. Congrats!

## Originally
To use, start a docker container with the following command:

```
docker run  -t -i \
    -p 127.0.0.1:443:1094 \
    --volume /etc/grid-security/:/etc/grid-security \
    --volume /tmp/exportfiles:/export \
    scitokens/xrootd-scitokens
```

This will export `/exportfiles` on port 443, only binding to localhost.

By default, the `scitokens.cfg` allows a few known SciTokens issuers, mapping them to
a sub-directory of the exported directory (e.g., `/osg` and `/cms`).  This may work for
quick demos, but production cases will likely want to overwrite this file with the site's
preferred issuers.

# Example Client Usage

Once you acquire a SciToken, it can be used by simply setting it in an `Authorization`
header:

```
$ curl -H 'Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6ImMxY2UwYzYyOWUwYzc3YTMzOGQwNjc2NDU5NDQ4NTA3Y2VkYzA4N2JiNTNlOGVmY2FjMzVhNjUxZTZmMDQxMzYifQ.eyJzdWIiOiJiYm9ja2VsbSIsImV4cCI6MTUxNzAyNDQ2NCwiaXNzIjoiaHR0cHM6Ly9zY2l0b2tlbnMub3JnL29zZy1jb25uZWN0IiwiaWF0IjoxNTE3MDIwODY0LCJzY3AiOiJyZWFkOi8iLCJuYmYiOjE1MTcwMjA4NjR9.0U-fmIvkkPgW3ElAeOczi10jxrItPWtlFgf2wQuU1a7bd2JEVQ9jocDYU0X7NKecvM3b3Q62D6QZqI4t8Q6rkIzR9jZAU4yG01L6_eq338D-Y6Xth56u44o8FaIVqOJY7mmMCBQGPoGPYyaZ4kO4PnMTu_2vhtjeyhOFOYJxUsFq5jnczfdikXyuJHkDuKSfal1JGeymQfxvamRew8ZOzyAQ-ONRCdn1aKhPVg6k9AfoTmeA6ijlBPmHmXJbbHKcVRaTVvFQvYX51eg6HX_qLeSqSN2InZfWlYj54IEDTAozic_89UzrH17heh3kL_S6gA4Y2ZNv3sgE05ke17u_KA' -k https://127.0.0.1:1094/osg/hello_world.txt
Hello OSG!
```
