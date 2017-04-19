quagga_ospf
==========

This is a memo for building the ospf service by quagga.
The environment is in RHEL7.
----------

### install the quagga packages

    yum install quagga -y

### SELinux boolean to turn on for enabling to save the configuration file

    getsebool -a | grep zebra
    setsebool zebra_write_config on -P

### firewall-cmd permit the ospf protocol to establish the connection

    firewall-cmd --direct --add-rule ipv4 filter INPUT 99 -d 224.0.0.5 -p ospf -j ACCEPT --permanent

### also you can create the services xml for enabling the ospf service

    cat <<EOF > /etc/firewalld/services/ospf.xml
    <?xml version="1.0" encoding="utf-8"?>
    <service>
     <short>OSPF</short>
     <description></description>
     <port protocol="ospf" port=""/>
     <destination ipv4="224.0.0.5"/>
    </service>
    EOF

    firewall-cmd --add-service=ospf --permanent

### copy the ospf configuration sample file to /etc/quagga and change the user & gruop

    cp /usr/share/doc/quagga-0.99.22.4/ospfd.conf.sample /etc/quagga/ospfd.conf
    chown quagga:quagga /etc/quagga/ospfd.conf

### set up the auto startup for quagga & ospfd service

    systemctl enable zebra.service
    systemctl enable ospfd.service

### start up the quagga's service 

    systemctl restart zebra.service
    systemctl restart ospfd.service

### use vtysh command to build up the ospf protocol for A PC

    vtysh
    localhost# configure terminal
    localhost(config)# router ospf
    localhost(config-router)# redistribute connected
    localhost(config-router)# network 10.10.10.0/24 area 0
    localhost(config-router)# network 192.168.1.0/24 area 0
    localhost(config-router)# end
    localhost# write memory

### use vtysh command to build up the ospf protocol for B PC

    vtysh
    localhost# configure terminal
    localhost(config)# router ospf
    localhost(config-router)# redistribute connected
    localhost(config-router)# network 10.10.10.0/24 area 0
    localhost(config-router)# network 172.16.1.0/24 area 0
    localhost(config-router)# end
    localhost# write memory

### show the ospf status

    vtysh
    localhost# show ip ospf neighbor
    localhost# show ip ospf route
    localhost# show ip ospf interface

