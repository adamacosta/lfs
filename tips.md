# Tips and Tricks

## Find and remove broken links

*Warning: Do NOT do this if you run containerized services that use persistent volumes as it will remove links that may be valid in the container but not on the host*

*Other Warning: Do NOT do this in any directories mounted from another filesystem, as those will also not be valid links from the mount point.*

```sh
find -L <dir> -type l -exec sudo unlink {} \;
```

## Count number of downloaded packages

This is not perfect because some packages have separate documentation or simply come as multiple tarballs but are still really one package, so it will overcount.

```sh
find /sources -path "/sources/patches" -prune -o -type f -not -name "*.patch" | wc -l
```

## Pull just filenames from the URL lists

*Doesn't completely work because of the GitHub URLs that use a tag instead of a filename to identify the tarball.*

```sh
cat wget-list | sed -E 's|^http.*/([A-Za-z0-9\._-]+\.tar\.[bgx]z.*)|\1|' | grep -v 'patch' | grep -v 'http' | cut -d '/' -f 1
```

## Check latest GNOME

https://download.gnome.org/sources/?C=M&O=D is all sources sorted by date, a good way to see what might be out of date.
