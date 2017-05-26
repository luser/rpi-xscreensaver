Start with https://github.com/sdt/docker-raspberry-pi-cross-compiler

docker pull sdthirlwall/raspberry-pi-cross-compiler
docker run -t -i --rm sdthirlwall/raspberry-pi-cross-compiler /bin/sh

cat > /rpxc/sysroot/etc/apt/sources.list <<EOF
deb http://archive.raspbian.org/raspbian jessie main
deb-src http://archive.raspbian.org/raspbian jessie main
deb http://archive.raspbian.org/raspbian jessie firmware
deb-src http://archive.raspbian.org/raspbian jessie firmware
EOF

apt-get install -y bc intltool pkg-config dpkg-dev
rpdo apt-get update
rpdo apt-get install dpkg-dev
rpdo apt-get build-dep xscreensaver
rpdo symlinks -cors /

cd $SYSROOT/build
curl https://www.jwz.org/xscreensaver/xscreensaver-5.36.tar.gz | tar xz
cd xscreensaver-5.36
sed -ri 's/\s+as_fn_error.*trivial ANSI C program.*$//' configure
CPPFLAGS=--sysroot=$SYSROOT CFLAGS="-O2 --sysroot=$SYSROOT" LDFLAGS=--sysroot=$SYSROOT pkg_config=$PWD/pkg-config ./configure --build=x86_64-unknown-linux-gnu --host=$HOST --prefix=$SYSROOT/build/xscreensaver/usr --with-x-app-defaults=$SYSROOT/build/xscreensaver/etc/X11/app-defaults
make -j8 && make -j8 install
cd ..
mkdir -p xscreensaver/debian
mkdir -p xscreensaver/DEBIAN
# cp control changelog xscreensaver/debian
chown -R root:root xscreensaver
rpdo 'cd xscreensaver; find . -type f -a -executable | xargs dpkg-shlibdeps'
cd xscreensaver
dpkg-gencontrol -DArchitecture=armhf -P.
cd ..
rm -rf xscreensaver/debian
dpkg-deb --build xscreensaver .


# On RPi:
sudo dpkg -i xscreensaver_5.36_armhf.deb
sudo apt-get -yf install
startx /usr/bin/xscreensaver -nosplash &
DISPLAY=:0 /usr/bin/xscreensaver-command -activate
