---
title:  "Docker를 돌리고있는 서버 용량이 꽉찼을 때"
date:   2019-01-21 12:05:00
tags: [docker, log, mapper]
categories: projects
---

평소와 같이 logstash에 파이프라인을 추가하려고 서버에 들어갔다가, 자동완성을 하려고 배시에 cd p를 입력한 다음 tab키를 눌렀더니 다음과 같은 텍스트가 출력됐다.
{% highlight bash %}
$ cd p-bash: cannot create temp file for here-document: No space left on device
{% endhighlight %}
남은 용량이 얼마나 있는지 확인하려고 다음명령어를 쳤다.
{% highlight bash %}
$ df -h
Filesystem                              Size  Used Avail Use% Mounted on
udev                                    3.9G     0  3.9G   0% /dev
tmpfs                                   798M   79M  720M  10% /run
/dev/mapper/[HOST이름]--vg-root   90G  4.3G   82G   6% /
tmpfs                                   3.9G     0  3.9G   0% /dev/shm
tmpfs                                   5.0M     0  5.0M   0% /run/lock
tmpfs                                   3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/sda1                               472M  153M  295M  35% /boot
tmpfs                                   798M     0  798M   0% /run/user/1003
overlay                                  90G  4.3G   82G   6% /var/lib/docker/overlay2/a5d0d5380f3e692bccb926c9fbf1c07ba6abc73db094b0e526d71aa19fd51537/merged
shm                                      64M     0   64M   0% /var/lib/docker/containers/ef5745582de101dddc056068c78e024388169af634315348427a80d59148a324/mounts/shm
{% endhighlight %}