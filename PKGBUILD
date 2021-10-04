# Maintainer: Patrick Northon <northon_patrick3@yahoo.ca>
# Contributor: Brad Ackerman <brad@facefault.org>
# Contributor: Felix Golatofski <contact@xdfr.de>
# Contributor: EoleDev
# Contributor: Jan Cholasta <grubber at grubber cz>
# Contributor: Xiao-Long Chen <chenxiaolong@cxl.epac.to>

pkgbase=freeipa
pkgname=(python-ipalib
         python-ipaclient
         freeipa-common
         freeipa-client-common
         freeipa-client)
pkgver=4.9.6
pkgrel=1
pkgdesc='The Identity, Policy and Audit system'
arch=('i686' 'x86_64')
url='http://www.freeipa.org/'
license=('GPL3')
makedepends=('openldap'
             'krb5>=1.15.1'
             'libxmlrpc'
             'popt'
             'gettext'
             'python'
             'python-setuptools'
             'nspr'
             'nss'
             'openssl'
             'ding-libs'
             'libsasl'
             'sssd>=1.13.0'
             'python-jinja'
             'python-pyasn1-modules')
options=(emptydirs)
validpgpkeys=('0E63D716D76AC080A4A33513F40800B6298EB963')
source=("https://releases.pagure.org/freeipa/freeipa-${pkgver}.tar.gz"
        "https://releases.pagure.org/freeipa/freeipa-${pkgver}.tar.gz.asc"
        platform.patch
        freeipa-client-update-sshd_config
        freeipa-client-update-sshd_config.hook
        nis-domainname.service
        ipaplatform.tar.gz)
sha256sums=('f211167513704c567f9e36fad16c0eba2961e3596d03f079d85a18bc9c644dc4'
            'b5cfc539e0df29bae52f8fddfb43a7d8d29e5cbdceff4417c897416a3afdea51'
            '4b3629f2733182f68b3d28c28f782773103b814c486cf4fdb15336163b08c82e'
            '9fbac49fa4bc23afe0c4d575ea2795f1da435399289dbd04c5a3ac47580e2a0d'
            '1e73f394d276357dcd578df7a349b1f381c9edc7b1c053ecf65f7a9255c0490d'
            '74a394af693e3677146eff18a770a4271fba961b2af93b15b8ae26157af1760a'
            '7e20412c9347106485adee06b5fcee174c67eb5a30b6730452e300dfc44faa5e')

prepare() {
    cd freeipa-${pkgver}

    rm -rf ipaplatform/arch

    patch -p1 -i "${srcdir}/platform.patch"
    tar xf "${srcdir}/ipaplatform.tar.gz"

    # Workaround: We want to build Python things twice. To be sure we do not mess
    # up something, do two separate builds in separate directories.
    cp -r ../freeipa-${pkgver} ../freeipa-${pkgver}-python3
}

build() {
    cd freeipa-${pkgver}

    export PYTHON=/usr/bin/python3
    # Workaround: make sure all shebangs are pointing to Python 3
    # This should be solved properly using setuptools
    # and this hack should be removed.
    find \
        ! -name '*.pyc' -a \
        ! -name '*.pyo' -a \
        -type f -exec grep -qsm1 '^#!.*\bpython' {} \; \
        -exec sed -i -e '1 s|^#!.*\bpython[^ ]*|#!/usr/bin/python3|' {} \;
    ./configure --prefix=/usr \
                --sysconfdir=/etc \
                --sbindir=/usr/bin \
                --with-vendor-suffix=-arch-${pkgrel} \
                --disable-server \
                --without-ipatests \
                --disable-pylint --without-jslint \
		--with-ipaplatform=arch

    mkdir -p ../install

    # Please put as much logic as possible into make install. It allows:
    # - easier porting to other distributions
    # - rapid devel & install cycle using make install
    #   (instead of full RPM build and installation each time)
    #
    # All files and directories created by spec install should be marked as ghost.
    # (These are typically configuration files created by IPA installer.)
    # All other artifacts should be created by make install.
    #
    # Exception to this rule are test programs which want to install
    # Python2/3 versions at the same time so we need to rename them. Yuck.

    # Python 3 installation needs to be done first. Subsequent Python 2 install
    # will overwrite /usr/bin/ipa and other scripts with variants using
    # python2 shebang.
    (cd ipaclient && make install DESTDIR=../../install)
    (cd ipalib && make install DESTDIR=../../install)
    (cd ipaplatform && make install DESTDIR=../../install)
    (cd ipapython && make install DESTDIR=../../install)

    # Python 2 installation
    make install DESTDIR="$PWD"/../install

    # remove files which are useful only for make uninstall
    find ../install -wholename '*/site-packages/*/install_files.txt' -exec rm {} \;

    mkdir -p ../install/etc/ipa
    mkdir -p ../install/etc/ipa/nssdb
    mkdir -p ../install/var/lib/ipa-client/pki
    mkdir -p ../install/var/lib/ipa-client/sysrestore

    touch ../install/etc/ipa/default.conf
    touch ../install/etc/ipa/ca.crt
}

package_python-ipalib() {
    pkgdesc='Python libraries used by IPA'
    arch=('any')
    depends=("freeipa-common=$pkgver-$pkgrel"
             'python-gssapi>=1.2.0'
             'gnupg'
             'keyutils'
             'python-cryptography>=1.4'
             'python-netaddr>=0.7.16'
             'sssd'
             'python-qrcode>=5.0.0'
             'python-pyasn1'
             'python-pyasn1-modules'
             'python-dateutil'
             'python-yubico>=1.2.3'
             'python-dbus'
             'python-setuptools'
             'python-six'
             'python-ldap'
             'python-netaddr'
             'python-dnspython>=1.15'
             'python-netifaces>=0.10.4'
             'python-pyusb')
    provides=("python-ipapython=$pkgver-$pkgrel"
              "python-ipaplatform=$pkgver-$pkgrel")

    cd freeipa-${pkgver}

    install -D -m644 -t"$pkgdir"/usr/share/doc/$pkgname README.md \
                                                        Contributors.txt

    local _file
    for _file in ../install/usr/lib/python3.*/site-packages/ipapython \
                 ../install/usr/lib/python3.*/site-packages/ipalib \
                 ../install/usr/lib/python3.*/site-packages/ipaplatform \
                 ../install/usr/lib/python3.*/site-packages/ipapython-*.egg-info \
                 ../install/usr/lib/python3.*/site-packages/ipalib-*.egg-info \
                 ../install/usr/lib/python3.*/site-packages/ipaplatform-*.egg-info
    do
        _file="${_file#../install/}"
        mkdir -p "$pkgdir"/"${_file%/*}"
        mv ../install/"$_file" "$pkgdir"/"$_file"
    done
}

package_python-ipaclient() {
    pkgdesc='Python libraries used by IPA client'
    arch=('any')
    depends=("freeipa-client-common=$pkgver-$pkgrel"
             "freeipa-common=$pkgver-$pkgrel"
             "python-ipalib=$pkgver-$pkgrel"
             'python-dnspython>=1.15'
             'python-jinja')
    optdepends=('oddjob: mkhomedir support')

    cd freeipa-${pkgver}

    install -D -m644 -t"$pkgdir"/usr/share/doc/$pkgname README.md \
                                                        Contributors.txt

    local _file
    for _file in ../install/usr/lib/python3.*/site-packages/ipaclient \
                 ../install/usr/lib/python3.*/site-packages/ipaclient-*.egg-info
    do
        _file="${_file#../install/}"
        mkdir -p "$pkgdir"/"${_file%/*}"
        mv ../install/"$_file" "$pkgdir"/"$_file"
    done
}

package_freeipa-common() {
    pkgdesc='Common files used by IPA'
    arch=('any')
    conflicts=('freeipa-python')
    replaces=('freeipa-python')

    cd freeipa-${pkgver}

    install -D -m644 -t"$pkgdir"/usr/share/doc/$pkgname README.md \
                                                        Contributors.txt

    local _file
    for _file in ../install/usr/share/locale/*/*/ipa.mo
    do
        _file="${_file#../install/}"
        mkdir -p "$pkgdir"/"${_file%/*}"
        mv ../install/"$_file" "$pkgdir"/"$_file"
    done
}

package_freeipa-client-common() {
    pkgdesc='Common files used by IPA client'
    arch=('any')

    cd freeipa-${pkgver}

    install -D -m644 -t"$pkgdir"/usr/share/doc/$pkgname README.md \
                                                        Contributors.txt

    local _file
    for _file in ../install/etc/ipa/nssdb \
                 ../install/var/lib/ipa-client/pki \
                 ../install/var/lib/ipa-client/sysrestore \
                 ../install/usr/share/ipa/client/* \
                 ../install/usr/share/man/man5/default.conf.5*
    do
        _file="${_file#../install/}"
        mkdir -p "$pkgdir"/"${_file%/*}"
        mv ../install/"$_file" "$pkgdir"/"$_file"
    done
}

package_freeipa-client() {
    pkgdesc='IPA authentication for use on clients'
    depends=("freeipa-client-common=$pkgver-$pkgrel"
             "freeipa-common=$pkgver-$pkgrel"
             "python-ipaclient=$pkgver-$pkgrel"
             'python-augeas'
             'cyrus-sasl-gssapi'
             'ntp'
             'krb5'
             'authselect'
             'curl>=7.21.7'
             'yp-tools'
             'libxmlrpc'
             'sssd>=1.14.0'
             'certmonger>=0.78'
             'nss'
             'bind-tools'
             'oddjob'
             'python-gssapi>=1.2.0'
             'autofs'
             'nfsidmap'
             'nfs-utils'
             'chrony')
    conflicts=('freeipa-admintools')
    replaces=('freeipa-admintools')
    install=freeipa-client.install

    cd freeipa-${pkgver}

    install -D -t"$pkgdir"/usr/share/libalpm/scripts \
            "$srcdir"/freeipa-client-update-sshd_config
    install -D -m644 -t"$pkgdir"/usr/share/libalpm/hooks \
            "$srcdir"/freeipa-client-update-sshd_config.hook \

    install -D -m644 -t"$pkgdir"/usr/share/doc/$pkgname README.md \
                                                        Contributors.txt
                                                        
    install -Dm644 "$srcdir/nis-domainname.service" -t "$pkgdir/usr/lib/systemd/system"
    install -dm755 "$pkgdir/etc/krb5.conf.d"

    local _file
    for _file in ../install/etc/bash_completion.d \
                 ../install/usr/bin/ipa \
                 ../install/usr/bin/ipa-client-install \
                 ../install/usr/bin/ipa-client-automount \
                 ../install/usr/bin/ipa-certupdate \
                 ../install/usr/bin/ipa-getkeytab \
                 ../install/usr/bin/ipa-rmkeytab \
                 ../install/usr/bin/ipa-join \
                 ../install/usr/share/man/man1/ipa.1 \
                 ../install/usr/share/man/man1/ipa-getkeytab.1* \
                 ../install/usr/share/man/man1/ipa-rmkeytab.1* \
                 ../install/usr/share/man/man1/ipa-client-install.1* \
                 ../install/usr/share/man/man1/ipa-client-automount.1* \
                 ../install/usr/share/man/man1/ipa-certupdate.1* \
                 ../install/usr/share/man/man1/ipa-join.1*
    do
        _file="${_file#../install/}"
        mkdir -p "$pkgdir"/"${_file%/*}"
        mv ../install/"$_file" "$pkgdir"/"$_file"
    done
}
