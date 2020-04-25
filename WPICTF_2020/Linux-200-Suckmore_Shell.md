# **Linux: 200 Suckmore Shell**

For this challenge you had to log in to smsh@smsh.wpictf.xyz, you are presented with a jailed shell with limited functionality and the flag taunting you in the home directory:

```
> ?
Failed to execute command, error: 2
> help
Failed to execute command, error: 2
> ls
flag
> cat flag



^C>
```
There are other tools on there, with some potentials like vi for shell or direct reading, find with the exec switch and various others but there seemed to be issues with many of them.

```
> grep
Usage: grep [OPTION]... PATTERNS [FILE]...
Try 'grep --help' for more information.
> find
.
./.bashrc
./.bash_logout
./.bash_profile
./flag
> sed
Usage: sed [OPTION]... {script-only-if-no-other-script} [input-file]...

```

So with sed accesible we can use it to search for a pattern in the file, in this case a capital W because of the flag format.

```
> sed -n /W/p flag
echo "WPI{SUckmoreSoftwareN33dz2G3TitTogeTHER}"
```


Et voila, the flag

