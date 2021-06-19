# Tips and Tricks

## Find and remove broken links

*Warning: Do NOT do this if you run containerized services that use persistent volumes as it will remove links that may be valid in the container but not on the host*

```sh
find -L <dir> -type l -exec sudo unlink {} \;
```

## Count number of downloaded packages

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
