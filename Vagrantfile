VAGRANTFILE_API_VERSION = "2"

$setup = <<-SCRIPT

sed -i "/mirror:\\/\\//d" /etc/apt/sources.list
sed -i "1ideb mirror://mirrors.ubuntu.com/mirrors.txt trusty main restricted universe multiverse" /etc/apt/sources.list
sed -i "1ideb mirror://mirrors.ubuntu.com/mirrors.txt trusty-updates main restricted universe multiverse" /etc/apt/sources.list
sed -i "1ideb mirror://mirrors.ubuntu.com/mirrors.txt trusty-backports main restricted universe multiverse" /etc/apt/sources.list
sed -i "1ideb mirror://mirrors.ubuntu.com/mirrors.txt trusty-security main restricted universe multiverse" /etc/apt/sources.list

apt-get update

apt-get install squid -y --no-install-recommends
stop squid3

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
def local_cache(box_name)
  cache_dir = File.join(File.expand_path("~/.vagrant.d"),
                        'cache',
                        'apt',
                        box_name)
  partial_dir = File.join(cache_dir, 'partial')
  FileUtils.mkdir_p(partial_dir) unless File.exists? partial_dir
  cache_dir
end
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/trusty32"

  config.vm.network "private_network", ip: "192.168.56.250"

  config.vm.network :forwarded_port, guest: 22, host: 2200, id:'ssh', auto_correct:true

  cache_dir = local_cache(config.vm.box)
  config.vm.synced_folder cache_dir,
                         "/var/cache/apt/archives/"

  config.librarian_puppet.puppetfile_dir = 'puppet'
  config.librarian_puppet.resolve_options = { :force => true }

  puppet_args = [ '--confdir .' ]
  puppet_args += ['--verbose', '--debug', '--graph'] if ENV['DEBUG']

  options = {
    options: puppet_args,
    facter:  { fqdn: 'precise.vagrant' }
  }

  config.vm.provision :puppet, options do |puppet|
    puppet.manifests_path = 'puppet'
    puppet.manifest_file  = 'init.pp'
    puppet.temp_dir = '/tmp/vagrant-puppet'
    puppet.working_directory = '/tmp/vagrant-puppet/manifests'
  end
  
  config.vm.provision "shell", inline: $setup
end
