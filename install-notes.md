 Installation notes on Debian
==============================


 * on debian you most likely want to install the  git version:

    ```bash
    sudo apt-get install libgnutls28-dev libcap-dev # To get https and cap drop support
    # Additional dependencys for Lua: liblua5.1-0-dev
    git clone https://github.com/lxc/lxc
    cd lxc
    ./autogen
    ./configure
    # make sure it has support for everything you need
    
    make
    sudo make install
    
    
    
    # now we need to install cgmanager
    cd ..
    sudo apt-get install libpam0g-dev libdbus-1-dev libnih-dev libnih-dbus-dev
    git clone https://github.com/lxc/cgmanager
    cd cgmanager
    ./bootstrap.sh
    ./configure
    make
    sudo make install
    
    ```
    
 * Config:

    ```bash
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
    
    # make that persistent
    echo -e "\n\n\n kernel.unprivileged_userns_clone = 1" | sudo tee -a /etc/sysctl.conf
    
    
    # you now need to configure networking. I'd recommend reading https://wiki.debian.org/LXC/SimpleBridge
    # A NATed setup is recommended if you have a machine on a network not your own
    # (Without the DHCP Server belonging to you)
    # On most desktop systems you have network manager installed, so I'd recommend using that to create
    # the bridge. (Try using nmtui if the GUI tools annoy you)
    
    
    
    # Create test container
    lxc-create -l DEBUG -o lxc.log -n vm-test0 -t download -- -d debian -r jessie -a amd64
    
    # Start test container
    lxc-start -l DEBUG -o lxc.log -n vm-test0 -t download -- -d debian -r jessie -a amd64
    
    
    # See lxc.log for any fails.
    # Proceed with normal procedure
   ```
   
