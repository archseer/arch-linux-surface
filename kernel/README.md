# Build

```
MAKEFLAGS="-j4" makepkg -s --skippgpcheck
```

# To reproduce from a vanilla kernel PKGBUILD

- Follow the steps at https://wiki.archlinux.org/index.php/Kernel/Arch_Build_System#Getting_the_Ingredients
- Rename the package
- Add the necessary patches to source and sha256sums lists
