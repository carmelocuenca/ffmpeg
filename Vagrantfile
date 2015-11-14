# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ubuntu/trusty64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.

  #
  # https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu#CompilationInstallation
  #
  config.vm.provision "shell", inline: <<-SHELL
    # Get the Dependencies
    apt-get update
    sudo apt-get -y --force-yes install autoconf automake build-essential libass-dev libfreetype6-dev \
      libsdl1.2-dev libtheora-dev libtool libva-dev libvdpau-dev libvorbis-dev libxcb1-dev libxcb-shm0-dev \
      libxcb-xfixes0-dev pkg-config texinfo zlib1g-dev
    mkdir ~/ffmpeg_sources

    #
    # Compilation & Installation
    #

    # Yasm
    cd ~/ffmpeg_sources
    wget http://www.tortall.net/projects/yasm/releases/yasm-1.3.0.tar.gz
    tar xzvf yasm-1.3.0.tar.gz
    cd yasm-1.3.0
    ./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin"
    make
    make install
    make distclean

    # libx264
    cd ~/ffmpeg_sources
    wget http://download.videolan.org/pub/x264/snapshots/last_x264.tar.bz2
    tar xjvf last_x264.tar.bz2
    cd x264-snapshot*
    PATH="$HOME/bin:$PATH" ./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin" --enable-static
    PATH="$HOME/bin:$PATH" make
    make install
    make distclean
    
    # libx265
    sudo apt-get -y --force-yes install cmake mercurial
    cd ~/ffmpeg_sources
    hg clone https://bitbucket.org/multicoreware/x265
    cd ~/ffmpeg_sources/x265/build/linux
    PATH="$HOME/bin:$PATH" cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX="$HOME/ffmpeg_build" -DENABLE_SHARED:bool=off ../../source
    make
    make install
    make distclean
  
    # libfdk-aac
    cd ~/ffmpeg_sources
    wget -O fdk-aac.tar.gz https://github.com/mstorsjo/fdk-aac/tarball/master
    tar xzvf fdk-aac.tar.gz
    cd mstorsjo-fdk-aac*
    autoreconf -fiv
    ./configure --prefix="$HOME/ffmpeg_build" --disable-shared
    make
    make install
    make distclean

    # libmp3lame
    sudo apt-get -y --force-yes install nasm
    cd ~/ffmpeg_sources
    wget http://downloads.sourceforge.net/project/lame/lame/3.99/lame-3.99.5.tar.gz
    tar xzvf lame-3.99.5.tar.gz
    cd lame-3.99.5
    ./configure --prefix="$HOME/ffmpeg_build" --enable-nasm --disable-shared
    make
    make install
    make distclean

    #libopus
    cd ~/ffmpeg_sources
    wget http://downloads.xiph.org/releases/opus/opus-1.1.tar.gz
    tar xzvf opus-1.1.tar.gz
    cd opus-1.1
    ./configure --prefix="$HOME/ffmpeg_build" --disable-shared
    make
    make install
    make clean

    # libvpx
    cd ~/ffmpeg_sources
    wget http://storage.googleapis.com/downloads.webmproject.org/releases/webm/libvpx-1.4.0.tar.bz2
    tar xjvf libvpx-1.4.0.tar.bz2
    cd libvpx-1.4.0
    PATH="$HOME/bin:$PATH" ./configure --prefix="$HOME/ffmpeg_build" --disable-examples --disable-unit-tests
    PATH="$HOME/bin:$PATH" make
    make install
    # make clean

    # ffmpeg
    cd ~/ffmpeg_sources
    wget http://ffmpeg.org/releases/ffmpeg-snapshot.tar.bz2
    tar xjvf ffmpeg-snapshot.tar.bz2
    cd ffmpeg
    PATH="$HOME/bin:$PATH" PKG_CONFIG_PATH="$HOME/ffmpeg_build/lib/pkgconfig" ./configure \
      --prefix="$HOME/ffmpeg_build" \
      --pkg-config-flags="--static" \
      --extra-cflags="-I$HOME/ffmpeg_build/include" \
      --extra-ldflags="-L$HOME/ffmpeg_build/lib" \
      --bindir="$HOME/bin" \
      --enable-gpl \
      --enable-libass \
      --enable-libfdk-aac \
      --enable-libfreetype \
      --enable-libmp3lame \
      --enable-libopus \
      --enable-libtheora \
      --enable-libvorbis \
      --enable-libvpx \
      --enable-libx264 \
      --enable-libx265 \
      --enable-nonfree
    PATH="$HOME/bin:$PATH" make
    make install
    make distclean
    hash -r

    cp -r ~/ffmpeg_build ~/ffmpeg
    cd ~
    tar -cvzf /vagrant/ffmpeg.tar.gz ffmpeg/
  SHELL
end
