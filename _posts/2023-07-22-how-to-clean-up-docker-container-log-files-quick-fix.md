---
title: "quick fix: how to free some diskspace when docker container logging is filling up your drive"
date: 2023-07-22T08:47:34+02:00
categories: [docker,kubernetes]
tags: [cleanup, docker, log, container, diskspace]
---

So you think you've done everything done correctly when it comes to container logging.\
Created a nice [daemon.json](https://docs.docker.com/config/containers/logging/configure/#configure-the-default-logging-driver) in the `/etc/docker` directory that should rotate your logs\
Notice the word "should" because apparently this only works for newly created containers and those settings won't be picked up by containers already present on your system.\
So now you're getting phonecalls, alerts of filesystems filling up are pooring into your mailbox and you need to act now...

<!--more-->

## The Quick Fix Answer

Ok, if restarting Docker and re-creating the container is not an option today and you just want to reclaim some disk space from `/var/lib/docker` that has been taken from you,
this is a quick and dirty way of doing it, without breaking anything:

- First, find the ten biggest items in `/var/lib/docker` using this command (output will be shown in GB):
```bash
du -a -B G  /var/lib/docker | sort -n -r | head -n 10
```
Output:
```bash
output:
45G     /var/lib/docker
26G     /var/lib/docker/containers
22G     /var/lib/docker/containers/5d2d817421265d3901783ee2a4e7c374717377cdf516ed2caabd89fafba2f50e/5d2d817421265d3901783ee2a4e7c374717377cdf516ed2caabd89fafba2f50e-json.log
22G     /var/lib/docker/containers/5d2d817421265d3901783ee2a4e7c374717377cdf516ed2caabd89fafba2f50e
19G     /var/lib/docker/overlay2
4G      /var/lib/docker/containers/fbc9823e4a891e75ccce86b2f309ca990fa97b7a447e43d8102b9f0120d7a7f6/fbc9823e4a891e75ccce86b2f309ca990fa97b7a447e43d8102b9f0120d7a7f6-json.log
4G      /var/lib/docker/containers/fbc9823e4a891e75ccce86b2f309ca990fa97b7a447e43d8102b9f0120d7a7f6
3G      /var/lib/docker/overlay2/eafe551b9b6b714e46c84bbcedf50196ba98c9d745fd84f49ab8c47521e92653/merged
3G      /var/lib/docker/overlay2/eafe551b9b6b714e46c84bbcedf50196ba98c9d745fd84f49ab8c47521e92653
3G      /var/lib/docker/overlay2/aa8d40ef50ab86bf2c3a60b8d50d600c32c68b2970685f147ba91cd7ac142b0f/merged
```
Hmm that 22G json logfile should give us back some space, but how do I know which container is using it?
- Take the first 10 or so characters of the container-id and run this command: \
```bash
docker ps | grep 5d2d8174212
kube-apiserver
```
In my case the result was: `kube-apiserver`  (your result could be different )

- Optional step to verify if this container is really using that json log file:
```bash
docker inspect --format='{{.LogPath}}' kube-apiserver
```
- Now  that we've found the culprit, let's overwrite the log with a blank line:
```bash
sudo sh -c 'echo "" > $(docker inspect --format="{{.LogPath}}" kube-apiserver)'
```
A quick look at the diskspace should now reveal that some diskspace has been returned to the system.

Done!

More background info on this matter can be found here:\
[docker logging driver](https://docs.docker.com/config/containers/logging/configure/#configure-the-default-logging-driver)
[docker logging issue](https://github.com/docker/cli/issues/1148)
