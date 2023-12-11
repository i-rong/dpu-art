   MST (Mellanox Software Tools) service\
        =====================================

   This script is used to start MST service, to stop it,
   and for some other operations with Mellanox devices
   like reset or enabling remote access.

   The mst commands are:
   -----------------------

   mst start [--with_msix] [--with_unknown] [--with_i2cdev] [--with_lpcdev]

       Create special files that represent Mellanox devices in
       directory /dev/mst. Load appropriate kernel modules and
       saves PCI configuration headers in temp-directory.
       After successfully completion of this command the MST driver
       is ready to work and you can invoke other Mellanox
       tools like Infiniburn or tdevmon.
       You can configure the start command by edit the configuration
       file: /etc/mft/mst.conf, for example you can rename you devices.

       Options:
       --with_msix           : Create the msix device.
       --with_unknown        : Do not check if the device ID is supported.
       --with_i2cdev         : Create Embedded I2C primary

   mst stop [--force]

       Stop Mellanox MST driver service, remove all special files/directories
       and unload kernel modules.

        Options:
        --force : Force try to stop mst driver even if it's in use.

   mst restart [--with_msix] [--with_unknown] [--with_i2cdev] [--with_lpcdev]

       Just like "mst stop" followed by  "mst start [--with_msix] [--with_unknown] [--with_i2cdev] [--with_lpcdev]"

   mst server start -p <port> [-s <passphrase>]
       Start MST server to allow incoming connection.
       Default port is 23108
       Use '-s' flag to define the passphrase used by the server.
       If no passphrase is provided, a random one will be generated.


   mst server stop
       Stop MST server.

   mst remote add <hostname>[:port] [-s <passphrase>]
       Establish connection with specified host on specified port
       (default port is 23108). Add devices on remote peer to local
       devices list. <hostname> may be host name as well as an IP address.
       Use '-s' flag to provide the host's passphrase.
       If no passphrase is provided, you will be prompted to insert one.

   mst remote del <hostname>[:port]
       Remove all remote devices on specified hostname. <hostname>[:port] should
       be specified exactly as in the "mst remote add" command.

   mst ib add [OPTIONS] <local_hca_id> <local_hca_port>
       Add devices found in the IB fabric for inband access.
       Requires OFED installation and an active IB link.
           If local_hca_id and local_hca_port are given, the IB subnet connected
           to the given port is scanned. Otherwise, the default subnet is scanned.
       OPTIONS:
            --discover-tool <discover-tool>: The tool that is used to discover the fabric.
                                             Supported tools: ibnetdiscover, ibdiagnet. default: ibdiagnet
            --add-non-mlnx : Add non Mellanox nodes.
            --topo-file <topology-file>: A prepared topology file which describes the fabric.
                         For ibnetdiscover: provide an output of the tool.
                         For ibdiagnet: provide LST file that ibdiagnet generates.
            --use-ibdr  : Access by direct route MADs. Available only when using ibnetdiscover tool, for 5th generation devices.
            NOTE: if a topology file is specified, device are taken from it.
                  Otherwise, a discover tool is run to discover the fabric.

   mst ib del
       Remove all inband devices.

   mst cable add [OPTIONS] [params]
       Add the cables that are connected to 5th gen devices.
       There is an option to add the cables found in the IB fabric for Cable Info access,
       requires OFED installation and active IB links.
            If local_hca_id and local_hca_port are given, the IB subnet connected
            to the given port is scanned. Otherwise, all the devices will be scanned.
        OPTIONS:
            --with_ib: Add the inband cables in addition to the local PCI devices.
                params: <local_hca_id> <local_hca_port>

   mst cable del
       Remove all cable devices.

   mst gearbox add
       Add the gearbox devices, and gb manager (fpga), that are connected to host.

   mst gearbox del
       Remove all gearbox devices.

   mst jtag add
       Add jtag devices that are connected to host.

   mst jtag del
       Remove all jtag devices.

   mst retimer add
       Add retimer devices that are connected to host.

   mst retimer del
       Remove all retimer devices.

   mst nvl add
       Add NVL devices.

   mst nvl del
       Remove all NVL devices.

   mst status

       Print current status of Mellanox devices

       Options:
       -v run with high verbosity level (print more info on each device)

   mst save

       Save PCI configuration headers in temp-directory.

   mst load

        Load PCI configuration headers from temp-directory.

   mst rm <pci-device>

        Remove the corresponding special file from /dev/mst directory.

   mst add <pci-device>

        Add the corresponding special file from /dev/mst directory.

   mst version

       Print the version info