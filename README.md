# kazu-packages

source recipes and binary packages for **Kazu** (`kz`), the package manager of Nomikaze Linux.

every package here is built from source by a recipe, packed as a zstd-compressed tar, and indexed into `x86_64/index.kazu`. drop the built directory into a `kz` mirror and it just works.

## use it as a mirror

point `kz` at the built repo:

```
mirror main = https://raw.githubusercontent.com/Kerunomia/kazu-packages/main/x86_64
```

then:

```
kz update
kz find hello
kz install hello
```

## build

```
scripts/kzbuild recipes/hello/KZBUILD     build one package
scripts/kazu-mkrepo x86_64                refresh the index
scripts/rebuild-all                       build every recipe + index
```

`kzbuild` downloads each source, verifies its sha256, extracts, runs `build()`, then runs `package()` into a clean root and packs it.

## recipe format

a recipe is a `KZBUILD` shell file. no build system beyond bash, tar and zstd.

```
pkgname=hello
pkgver=2.12.3
pkgrel=1
pkgdesc="GNU hello, the canonical friendly greeting program"
arch=x86_64
libc=any
init=any
kernel=any
depends=()
provides=()
sources=(https://ftp.gnu.org/gnu/hello/hello-2.12.3.tar.gz)
sha256sums=(0d5f60154382fee10b114a1c34e785d8b1f492073ae2d3a6f7b147687b366aa0)

build() {
    cd "$srcdir/hello-2.12.3"
    ./configure --prefix=/usr --disable-nls
    make MAKEINFO=true
}

package() {
    cd "$srcdir/hello-2.12.3"
    make DESTDIR="$pkgdir" MAKEINFO=true install
}
```

`libc`, `init` and `kernel` describe what the package is built against. `any` means it works everywhere. a package can be built several times with different values to populate the choice matrix.

## packages

| package | version | description |
| --- | --- | --- |
| hello | 2.12.3 | the canonical friendly greeting program |
| zlib | 1.3.2 | compression library used by nearly everything |
| zstd | 1.5.7 | fast real-time compression |
| lz4 | 1.10.0 | extremely fast lossless compression |
| xxhash | 0.8.4 | fast non-cryptographic hash |

## layout

```
recipes/      one directory per package, each with a KZBUILD
scripts/      kzbuild, kazu-mkrepo, rebuild-all
x86_64/       built packages and index.kazu
```

## status

a seed set. the goal is a base system and the toolchain, built across every libc, init and kernel Nomikaze offers.
