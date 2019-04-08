title: snap占用/dev/loop0-/dev/loop11占用100%
date: 2019-01-21 10:20:36
tags:
- loop0
- Linux
categories:
- Linux
---

/dev/loop0-loop11占用100%,我们只要清理掉一下就可以了.
```shell
sudo apt autoremove --purge snapd
```