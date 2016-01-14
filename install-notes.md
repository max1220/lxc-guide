 Installation notes on Debian
==============================


 * on debian you most likely want to install the  git version:
 
    sudo apt-get install libgnutls28-dev libcap-dev # To get https and cap drop support
    # Additional dependencys for Lua: liblua5.1-0-dev
    git clone https://github.com/lxc/lxc
    cd lxc
    ./autogen
    ./configure
    # make sure it has support for everything you need
    
    make
    sudo make install
    
 * Config:
    mkdir -p ~/.local/share/lxc
    chmod +x ~/.local/share/lxc
    mkdir -p ~/.config/lxc
    
    cat <<EOF >> ~/.config/lxc/default.conf
    lxc.id_map u 0 558752 65536
    lxc.id_map g 0 558752 65536
    
    lxc.network.type = veth
    lxc.network.link = lxcbr0
    lxc.network.flags = up
    lxc.network.hwaddr = be:ef:xx:xx:xx:xx
    EOF
    # Note the uid/gid mapping in the first 2 lines:
    # these subuid/subgid ranges need to belong to you.
    # check what ranges you own by: cat /etc/sub*id | grep $USER
    # the default in mostly all guides does not work...
    # If you don't have any subuid/subgid ranges, try:
    #   sudo usermod --add-subuids 100000-165536 $USER
    #   sudo usermod --add-subgids 100000-165536 $USER
    # Check if these are blocked by anything beforehand!
    # You should now have a new subuid/subgid range!
    
    # Create config for lxc-user-nic, so we can have unpriviledged new interfaces
    sudo mkdir -p /etc/lxc
    echo "$USER veth lxcbr0 500" | sudo tee /etc/lxc/lxc-usernet
    
    # Allow unprivileged_userns_clone(Needed by LXC)
    sudo sysctl kernel.unprivileged_userns_clone=1
    
    
    # Create test container
    lxc-create -l DEBUG -o lxc.log --name vm-test0 -t download -- -d debian -r jessie -a amd64
    
    # See lxc.log for any fails.
    # Proceed with normal procedure