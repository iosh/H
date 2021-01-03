---
title: command line
date: 2021-01-03 14:02:30
tags: linux
---

the useful command line on linux

<!-- more -->

the test command platform is ubuntu 20.11.3 of armbian

## common

- in bash , use the Tab to complete arguments or list all available commands and ctrl - r to search through command histor

- in bash , use the ctrl-w to delete last wrd, and ctrl-u to delete the all line. ctrl-e to move cursor to end of line. ctrl-k to delete cursor to the end line, ctrl-l to clear the screen

- use the history to see recent commands. follow with !n to excute again. !$ for last argument and !! for last command.

- use the cd to get directory, access files relative to you home directory with the ~ prefix, in sh scripts refer to eht home dirctory as $HOME

- use the cd - can to go back to previous working directory

- pstree -p is a helpful display of the process tree

- use the pgrep and pkill to find or signal processes by name (-f is helpful)

- use nohup or disown to keep running a background process forever

- check what processes are listening via `netstat -lntp` or `ss -plat` or `lsof -iTCP -sTCP:LISTEN -p -n`

- use the `lsof` and `fuser` for open sockets and files

- use the `uptime` or `w` to know how long time the system been running

- use the alias to create shortcuts for commonly used commands. for example `alias ll='ls -latr'` create a new alias `ll`, save aliases, shell settings ,and functions you commonly use in `~/.bashrc`

- put the settings of enviroment variables as well as commands tha should be excuted when you login in `~/.bash_profile`

- in bash scripts, use `set -x`(or the variant `set -v` which logs raw input, includeing unexpanded variables and comments) for debugging out
  put , use stric model unless you hanve a good reason not to: use `set -e` to abort on error (nonzero exit code) . use `set -u` to detect unset variable usages. consider `set -o popefail` too, to abort on errors withon ppes. for more involved scripts also use `trap` on EXIT or ERR, a useful habit is start a script like this, which will make it detect an abort on common error and print a message:

```bash
set -e pipefail
trap "echo 'error : Script failed: see faild comman above'" ERR
```

- in bash scripts, subshells are convenient ways to group commands, A common example is to temporatily move to a different working directory

```bash
# do somethin in current dir
(cd /root/script/dir && other-command)
# continue ub original dir
```

- the output of a command can be treated like a file via `<(some command)` (know as process substitution) for example compare local /etc/hosts with a remote one:

```bash
diff /etc/hosts <(ssh somehost cat /etc/hosts)
```

- a simple web server for all files in the current directory `python -m http.server 8888`

- for switching the shell to another user use `su username` or `su - username` the default username to root.

## Processing files and date

- to locate a file by name in the current directory `find . -iname "*something*" `to find a file anywhere by name, use `locate somethin`

- user the `grep -r` to search throuth source or date file. including

- to conver HTML to text: `lynx -dump -stdin`

- for markdown , HTML and all kinds for document conversion, try `pandoc`, for example to convert a markdown document to word format: `pandoc README.md --form markdown --to docx -o tmp.docx`

- use `xmlstarlet` to handle XML ,it is old but good.

- for JSON use jq, for YAML use shyaml, for Excel or CSV files use `scvkit`.

- you can set a specitic command;s environment by refixing its invocation with the enviroment var iable settings, as in `TZ=Asia/Shanghai date`

- date and time : to get the current date and time in the helful ISO 8601 format. use `date -u +"%Y-%m-%dT%H:%M:%SZ"`, to manipluate date and time expressions , ue dateadd, datadiff, strtime.

- use `zless` `zmore` `zcat` and `zgrep` to operate on compressed fils.

## System debugging

- for web debugging `curl` and `curl -I` are handy, or their `wget` equivalents, or the more modern `httpie`

- use the `top` or `htop` wo show current cpu/disk status, `iostat` and `iotop` use the `iostat -mxz 15` for basic cpu and detiled per partion disk stas and performance insight

- use `netstat` and `ss` can show network donnection details.

- for a quick overview of what's happening on a system. `dstat` is expecially useful, for broadest overview with details use `glances`

- to know memory status, run and understand the output of `free` and `vmstat` , in particular be aware the cached values is memeory held by the linux kernel as file cache, so effectively counts toward the free value

- for looking at why a disk is full `ncdu` saves time over the usual commands lie `du -sh *`

- to find which socket or process is using band width try `iftop` or `nethogs`

- the `ab` tool is helpful for quick-and-dirty checking of web server performance . for more complex load testing. try `siege`

- for more serious network debugging `wireshark` `tshark` or `ngrep`




