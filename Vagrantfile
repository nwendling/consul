# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = '2'

@script = <<SCRIPT
SRCROOT="/opt/go"
SRCPATH="/opt/gopath"

# Get the ARCH
ARCH=`uname -m | sed 's|i686|386|' | sed 's|x86_64|amd64|'`

# Install Go
sudo apt-get update
sudo apt-get install -y build-essential git-core

# Install Go
cd /tmp
wget --quiet --no-check-certificate https://storage.googleapis.com/golang/go1.4.2.linux-${ARCH}.tar.gz
tar -xvf go1.4.2.linux-${ARCH}.tar.gz
sudo mv go $SRCROOT
sudo chmod 775 $SRCROOT
sudo chown vagrant:vagrant $SRCROOT

# Setup the GOPATH; even though the shared folder spec gives the consul
# directory the right user/group, we need to set it properly on the
# parent path to allow subsequent "go get" commands to work.
sudo mkdir -p $SRCPATH
sudo chown -R vagrant:vagrant $SRCPATH

cat <<EOF >/tmp/gopath.sh
export GOPATH="$SRCPATH"
export GOROOT="$SRCROOT"
export PATH="$SRCROOT/bin:$SRCPATH/bin:\$PATH"
EOF
sudo mv /tmp/gopath.sh /etc/profile.d/gopath.sh
sudo chmod 0755 /etc/profile.d/gopath.sh
source /etc/profile.d/gopath.sh

# Install go tools
go get golang.org/x/tools/cmd/cover
SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.provision 'shell', inline: @script, privileged: false
  config.vm.synced_folder '.', '/opt/gopath/src/github.com/hashicorp/consul'

  %w[vmware_fusion vmware_workstation].each do |_|
    config.vm.provider 'p' do |v|
      v.vmx['memsize'] = '2048'
      v.vmx['numvcpus'] = '2'
      v.vmx['cpuid.coresPerSocket'] = '1'
    end
  end

  # Note we use older boxes here to avoid glibc version check problems with
  # the built C dependencies.
  config.vm.define '64bit' do |n1|
    n1.vm.box = 'chef/ubuntu-10.04'
  end

  config.vm.define '32bit' do |n2|
    n2.vm.box = 'chef/ubuntu-10.04-i386'
  end

  config.push.define "www", strategy: "local-exec" do |push|
    push.script = "scripts/website_push.sh"
  end
end
