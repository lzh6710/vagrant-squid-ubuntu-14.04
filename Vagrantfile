VAGRANTFILE_API_VERSION = "2"

$setup = <<-SCRIPT

sed -i "/mirror:\\/\\//d" /etc/apt/sources.list
sed -i "1ideb mirror://mirrors.ubuntu.com/mirrors.txt trusty main restricted universe multiverse" /etc/apt/sources.list
sed -i "1ideb mirror://mirrors.ubuntu.com/mirrors.txt trusty-updates main restricted universe multiverse" /etc/apt/sources.list
sed -i "1ideb mirror://mirrors.ubuntu.com/mirrors.txt trusty-backports main restricted universe multiverse" /etc/apt/sources.list
sed -i "1ideb mirror://mirrors.ubuntu.com/mirrors.txt trusty-security main restricted universe multiverse" /etc/apt/sources.list

apt-get update

apt-get install vim mc make gcc g++

cd /usr/local
wget http://www.squid-cache.org/Versions/v3/3.4/squid-3.4.8.tar.gz
tar zxvf squid-3.4.8.tar.gz
cd squid-3.4.8
./configure --prefix=/usr   --exec-prefix=/usr   --bindir=/usr/sbin   --sbindir=/usr/sbin   --sysconfdir=/etc/squid   --datadir=/usr/share/squid   --includedir=/usr/include   --libdir=/usr/lib   --libexecdir=/usr/lib/squid   --localstatedir=/var   --sharedstatedir=/usr/com   --mandir=/usr/share/man   --infodir=/usr/share/info   --x-includes=/usr/include   --x-libraries=/usr/lib   --enable-shared=yes   --enable-static=no   --enable-carp    --enable-storeio=aufs,ufs   --enable-removal-policies=heap,lru   --disable-icmp   --disable-delay-pools   --disable-esi   --enable-icap-client   --enable-useragent-log   --enable-referer-log   --disable-wccp   --enable-wccpv2   --disable-kill-parent-hack   --enable-snmp   --enable-cachemgr-hostname=localhost   --enable-arp-acl   --disable-htcp  --disable-forw-via-db   --disable-follow-x-forwarded-for   --enable-cache-digests    --disable-poll   --enable-epoll   --enable-linux-netfilter   --disable-ident-lookups   --enable-default-hostsfile=/etc/hosts    --with-default-user=vagrant   --with-large-files  --enable-mit=/usr   --with-logdir=/var/log/squid   --enable-http-violations   --enable-zph-qos   --with-filedescriptors=65536   --enable-gnuregex --enable-async-io=64 --with-aufs-threads=64  --with-pthreads --with-aio  --enable-default-err-languages=English --enable-err-languages=English --disable-hostname-checks --enable-underscores
make
make install

#apt-get install squid -y --no-install-recommends
stop squid3

apt-get install squidclient

mkdir /vagrant/cache -p
chown -R vagrant:vagrant /var/log/squid3
mv /etc/squid3/squid.conf /etc/squid3/squid.conf.orig
cp /vagrant/squid.conf /etc/squid3/squid.conf
sed -i "/vagrant\\/cache/d" /etc/init/squid3.conf
sed -i "/pre-start/a \\\twhile [ ! -d \\"/vagrant/cache\\" ]; do sleep 5; done" /etc/init/squid3.conf

squid3 -z
while [ ! -d "/vagrant/cache/0F/FF" ]; do sleep 5; done
start squid3

SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/trusty32"

  config.vm.network "private_network", ip: "192.168.56.250"

  config.vm.network :forwarded_port, guest: 22, host: 2200, id:'ssh', auto_correct:true
  
  config.vm.provision "shell", inline: $setup

  #config.ssh.port = 2200
end
