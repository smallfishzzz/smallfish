# debian package management tips

## compile deb packages using debian\`s compilation project


**compile and packages** 
```bash
> dpkg-buildpackage -us -uc -nc           or 
> dpkg-buildpackage -us -b -rfakeroot
```

**compile but not package (under the source code directory)**
```bash
> ./debian/rules build      or 
> dh make 
```


**clean up complied files (under the source code directory)**
```bash
> ./debian/rules  clean     or 
> dh clean
```

## patching

**install devscripts**
> sudo apt install devscripts

**add quilt script environment for  quilt push and quilt  pop command**
``` bash
for where in ./ ../ ../../ ../../../ ../../../../ ../../../../../; do
    if [ -e ${where}debian/rules -a -d ${where}debian/patches ]; then
        export QUILT_PATCHES=debian/patches
        break
    fi
done
```


**apply patches**
```bash
>  quilt push -a       # apply  ./debian/patches
>  quilt pop -a         # remove all applied patches,  
```

## compilation options

**check compilation options**
> dpkg-buildflags

**modify compilation options**
1.  environment 
remove `-O2` flags form c and c++
```bash
> export  DEB_CFLAGS_STRIP="-O2"
> export  DEB_CXXFLAGS_STRIP="-O2"
```

append `-g` flags to c and c++ 
```bash
> export DEB_CFLAGS_APPEND="-g"
> export DEB_CXXFLAGS_APPEND="-g"
```

you can also write the flag to `/usr/local/etc/dpkg/buildflags.conf`, example:

``` bash
APPEND CFLAGS -ggdb -O3
STRIP CXXFLAGS -O2
```


**reference dpkg-buildflags**
https://www.man7.org/linux/man-pages/man1/dpkg-buildflags.1.html


## how to prevent binary file or xxx.so files from being striped

use vim to edit  ./debian/rules under the source code directory,  add new line
```bash
override_dh_strip:
	# don`t write anything
```

if this is not useful, you can also modify system files, but it is only suitable for use in debugging envrionments.

find the file /usr/share/perl5/Debian/Debhelper/Buildsystem/makefile.pm, find  `--strip-program=true` and modify it to `--strip-program=false`. please note that canceling the `--strip-program` wile cause the binary file and xxx.so file to become larger.
