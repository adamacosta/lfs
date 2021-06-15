# Tips and Tricks

Find and remove broken links:

```sh
find -L <dir> -type l -exec sudo unlink {} \;
```
