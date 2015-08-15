# Maintainer: Keshav P R <(the.ridikulus.rat) (aatt) (gemmaeiil) (ddoott) (ccoomm)>
# Contributor: cruznick <cruznick@archlinux.us>
# Contributor: fsckd <fsckdaemon@gmail.com>

## Tunables: change y to n to disable
_rm_build_dirs="${_rm_build_dirs:-n}"        # remove build directories

pkgname="burg-efi-x86_64-bzr"
pkgver=1844
pkgrel=6
pkgdesc="Brand-new Universal loadeR from GRUB2 - Built for x86_64 UEFI"
url="http://code.google.com/p/burg/"
license="GPL3"
arch=('x86_64')

makedepends=('bzr' 'ruby' 'bdf-unifont' 'python2' 'ncurses')
depends=('gettext' 'freetype2' 'dosfstools' 'efibootmgr')
optdepends=('mtools')

conflicts=('burg-bios' 'burg-emu')
provides=('burg-efi-x86_64' 'burg' 'burg-bzr')

# options=('!makeflags')
changelog='burg.changelog'
backup=('etc/default/burg' 'etc/burg.d/40_custom')

source=('burg.default' 'arch-burg.patch')

sha256sums=('914da1384390e8cd63f335564075843eb52172d67a174dd44226f0710d5f0cde'
            '57fa4d1ab439a3e716cf60f5eda533969f8d4a46b6425e85f0529d1897897446')

install='burg.install'

_bzrmod="burg"
_bzrtrunk="lp:${_bzrmod}"

_builddir="${_bzrmod}-build"

if [ "${CARCH}" = 'i686' ]
then
    echo "This package can be built only in a x86_64 system. Exiting."
    exit 1
fi

_common_configure_opts="--host="${CARCH}-pc-linux-gnu" --program-prefix="" \
                        --prefix="/usr" --bindir="/usr/bin" \
                        --sbindir="/usr/sbin" --mandir="/usr/share/man" \
                        --infodir="/usr/share/info" --sysconfdir="/etc" \
                        --disable-werror"

_update_bzr() {
    
    cd "${srcdir}/"
    
    msg "Connecting to the server...."
    
    if [[ ! -d "${srcdir}/${_bzrmod}" ]] ; then
        bzr branch "${_bzrtrunk}"
    else
        cd "${srcdir}/${_bzrmod}" && bzr pull "${_bzrtrunk}"
    fi
    
    msg "BZR checkout done or server timeout"
    
}

_build_dir() {
    
    local _rm_builddir='0'
    
    while (( "$#" ))
    do
        if [[ "$1" == '-r' ]]; then
            _rm_builddir='1'
        else
            _builddir="${_bzrmod}-$1-build"
        fi
        
        shift
    done
    
    rm -rf "${srcdir}/${_builddir}"
    
    if [[ "${_rm_builddir}" == '0' ]] ; then
        cp -rip "${srcdir}/${_bzrmod}" "${srcdir}/${_builddir}"
        cd "${srcdir}/${_builddir}/"
    fi
    
}

_build_common() {
    
    cd "${srcdir}/${_builddir}/"
    
    ## Requires python2
    sed 's|python |python2 |g' -i ./autogen.sh || true
    
    echo
    
    ./autogen.sh
    
    echo
    
}

_build_uefi_x86_64() {
    
    msg "Building burg uefi x86_64...."
    
    _build_dir
    
    echo
    
    ## Patch to include Archlinux Kernels and Initramfs files in burg.cfg
    patch -Np1 -i "${srcdir}/arch-burg.patch"
    
    echo
    
    _build_common
    
    echo
    
    cd "${srcdir}/${_builddir}/"
    
    ## fix unifont.bdf location so burg-mkfont can create *.pf2 files
    sed 's|/usr/share/fonts/unifont|/usr/share/fonts/misc|g' -i ./configure || true
    
    _uefi_configure_opts="${_common_configure_opts} --with-platform=efi --target=x86_64 --disable-efiemu --disable-grub-emu-usb"
    ./configure ${_uefi_configure_opts}
    
    echo
    
    CFLAGS="" make
    
    echo
    
}

build() {
    
    _update_bzr
    
    echo
    
    _build_uefi_x86_64
    
    echo
    
}

package() {
    
    cd "${srcdir}/${_builddir}/"
    
    make DESTDIR="${pkgdir}/" install
    
    echo
    
    ## Delete deprecated burg-mkconfig helper file
    rm -f "${pkgdir}/usr/lib/burg/update-burg_lib" || true
    
    ## install /etc/default/burg
    install -D -m0644 "${srcdir}/burg.default" "${pkgdir}/etc/default/burg"
    
    if [[ "${_rm_build_dirs}" == 'y' ]]; then
        _build_dir -r
    fi
    
}
