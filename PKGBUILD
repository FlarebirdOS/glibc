pkgname=(glibc glibc-32bit)
pkgbase=glibc
pkgver=2.42
pkgrel=1
arch=('x86_64')
url="https://www.gnu.org/software/libc/"
license=('GPL-2.0-or-later' 'LGPL-2.1-or-later')
makedepends=('linux-api-headers' 'tzdata')
options=('!lto')
source=(https://ftp.gnu.org/gnu/${pkgbase}/${pkgbase}-${pkgver}.tar.xz
    https://www.linuxfromscratch.org/patches/downloads/${pkgbase}/${pkgbase}-${pkgver}-fhs-1.patch
    ld.so.conf
    nsswitch.conf)
sha256sums=(d1775e32e4628e64ef930f435b67bb63af7599acb6be2b335b9f19f16509f17f
    643552db030e2f2d7ffde4f558e0f5f83d3fabf34a2e0e56ebdb49750ac27b0d
    dad04a370e488aa85fb0a813a5c83cf6fd981ce01883fc59685447b092de84b5
    44b218ce4dc5949c2d4e6b2f4e8d1e5b42ff518b792b0b7f1f137b1167e60d04)

prepare() {
    cd ${pkgbase}-${pkgver}

    patch -Np1 -i ${srcdir}/${pkgbase}-${pkgver}-fhs-1.patch

    sed -e '/unistd.h/i #include <string.h>' \
        -e '/libc_rwlock_init/c\
    __libc_rwlock_define_initialized (, reset_lock);\
    memcpy (&lock, &reset_lock, sizeof (lock));' \
        -i stdlib/abort.c

    install -vdm755 ${pkgbase}-build ${pkgbase}-build-32bit
}

build() {
    cd ${pkgbase}-${pkgver}

    local configure_args=(
        --disable-werror
        --disable-nscd
        --enable-kernel=5.4
        --enable-stack-protector=strong
        --enable-multi-arch
        --with-pkgversion="Flarebird Glibc ${pkgver}"
    )

    CFLAGS=${CFLAGS/-Wp,-D_FORTIFY_SOURCE=3/}

    (
        cd ${pkgbase}-build

        echo "rootsbindir=/usr/sbin" >> configparms
        echo "complocaledir=/usr/lib/locale" >> configparms

        ../configure                    \
            ${configure_options}        \
            --enable-cet                \
            libc_cv_slibdir=/usr/lib64  \
            "${configure_args[@]}"

        make
    )

    (
        cd ${pkgbase}-build-32bit

        export CC="${CHOST}-gcc -m32 -mstackrealign"
        export CXX="${CHOST}-g++ -m32 -mstackrealign"

        CFLAGS=${CFLAGS/-fno-omit-frame-pointer -mno-omit-leaf-frame-pointer/}

        echo "rootsbindir=/usr/sbin" >> configparms
        echo "complocaledir=/usr/lib/locale" >> configparms

        ../configure                         \
            --prefix=/usr                    \
            --host=i686-flarebird-linux-gnu  \
            --build=${CHOST}                 \
            --libdir=/usr/lib32              \
            --libexecdir=/usr/lib32          \
            libc_cv_slibdir=/usr/lib32       \
            "${configure_args[@]}"

        make
    )
}

package_glibc() {
    pkgdesc="GNU C Library"
    depends=('linux-api-headers' 'tzdata')
    backup=(
        etc/ld.so.conf
        etc/nsswitch.conf
    )

    cd ${pkgbase}-${pkgver}/${pkgbase}-build

    make DESTDIR=${pkgdir} install

    install -vdm755 ${pkgdir}/usr/lib/locale

    ${pkgdir}/usr/bin/localedef --prefix=${pkgdir} -i C -f UTF-8 C.UTF-8
    ${pkgdir}/usr/bin/localedef --prefix=${pkgdir} -i en_US -f ISO-8859-1 en_US
    ${pkgdir}/usr/bin/localedef --prefix=${pkgdir} -i en_US -f UTF-8 en_US.UTF-8
    ${pkgdir}/usr/bin/localedef --prefix=${pkgdir} -i zh_CN -f GB18030 zh_CN.GB18030
    ${pkgdir}/usr/bin/localedef --prefix=${pkgdir} -i zh_CN -f GBK zh_CN.GBK
    ${pkgdir}/usr/bin/localedef --prefix=${pkgdir} -i zh_CN -f UTF-8 zh_CN.UTF-8
    ${pkgdir}/usr/bin/localedef --prefix=${pkgdir} -i zh_CN -f GB2312 zh_CN.GB2312

    install -vm644 ${srcdir}/nsswitch.conf ${pkgdir}/etc/
    install -vm644 ${srcdir}/ld.so.conf ${pkgdir}/etc/

    install -vdm755 ${pkgdir}/etc/ld.so.conf.d
}

package_glibc-32bit() {
    pkgdesc="GNU C Library (32-bit)"
    depends=("glibc=${pkgver}")
    options+=('!emptydirs')

    cd ${pkgbase}-${pkgver}/${pkgbase}-build-32bit

    make DESTDIR=$PWD/DESTDIR install

    install -vdm755 ${pkgdir}/usr/lib{32,64}
    ln -sv usr/lib32 ${pkgdir}/lib32

    cp -a DESTDIR/usr/lib32/* ${pkgdir}/usr/lib32/

    install -vDm644 DESTDIR/usr/include/gnu/{lib-names,stubs}-32.h -t ${pkgdir}/usr/include/gnu/

    install -vdm755 ${pkgdir}/etc/ld.so.conf.d
    echo "/usr/lib32" >> ${pkgdir}/etc/ld.so.conf.d/glibc-32bit.conf

    ln -svf /usr/lib32/ld-linux.so.2 ${pkgdir}/usr/lib64/ld-linux.so.2
}
