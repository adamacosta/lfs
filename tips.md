# Tips and Tricks

### Find and remove broken links

*Warning: Do NOT do this if you run containerized services that use persistent volumes as it will remove links that may be valid in the container but not on the host*

```sh
find -L <dir> -type l -exec sudo unlink {} \;
```

### Count number of downloaded packages

```sh
find /sources -path "/sources/patches" -prune -o -type f -not -name "*.patch" | wc -l
```
