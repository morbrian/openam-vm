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
  config.vm.box = "centos/7"

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
  config.vm.network "private_network", ip: "172.16.12.10"

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
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    # vb.gui = true
  
    # Customize the amount of memory on the VM:
    vb.memory = "8192"
  end
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
  config.vm.provision "shell", inline: <<-SHELL
    # basics
    yum install vim -y
    yum install unzip -y
    # hostname
    sed -i 's/localhost\.localdomain/proxy.172.16.12.10.xip.io/' /etc/sysconfig/network
    sysctl kernel.hostname=proxy.172.16.12.10.xip.io
    # use /etc/hosts since xip.io has rate limits that could cause intermittant issues
    echo "172.16.12.10 proxy.172.16.12.10.xip.io >> /etc/hosts"
    # firewall
    #iptables -I INPUT -i lo -j ACCEPT
    systemctl enable firewalld
    systemctl firewalld start
    firewall-cmd --zone=public --permanent --add-port=443/tcp
    #firewall-cmd --zone=public --permanent --add-port=80/tcp
    firewall-cmd --zone=public --permanent --add-port=22/tcp
    firewall-cmd --reload
    # os updates
    #echo "SKIPPING yum update to speed things up during dev"
    yum update -y
    # os config
    sed -i '$ i openam soft nofile 65536' /etc/security/limits.conf
    sed -i '$ i openam hard nofile 131072' /etc/security/limits.conf
    sed -i '$ i wildfly soft nofile 65536' /etc/security/limits.conf
    sed -i '$ i wildfly hard nofile 131072' /etc/security/limits.conf
    sed -i '$ i opendj soft nofile 65536' /etc/security/limits.conf
    sed -i '$ i opendj hard nofile 131072' /etc/security/limits.conf
    # httpd
    yum install httpd -y
    yum install mod_ssl -y
    /usr/sbin/setsebool -P httpd_can_network_connect 1
    # stage certs
    mkdir /vagrant/certs
    tar xf /vagrant/certgen.tgz -C /vagrant/certs
    tar xf /vagrant/certs/certgen/proxy.172.16.12.10.xip.io.tgz -C /vagrant/certs
    # using 'cat' because there were some odd issues when moving or copying the files, uncertain why
    cat /vagrant/certs/proxy.172.16.12.10.xip.io.crt > /etc/pki/tls/certs/proxy.172.16.12.10.xip.io.crt && chmod 600 /etc/pki/tls/certs/proxy.172.16.12.10.xip.io.crt
    cat /vagrant/certs/proxy.172.16.12.10.xip.io.key > /etc/pki/tls/private/proxy.172.16.12.10.xip.io.key && chmod 600 /etc/pki/tls/private/proxy.172.16.12.10.xip.io.key
    rm -f /etc/pki/tls/certs/ca-bundle.crt
    cat /vagrant/certs/certgen/ca.crt > /etc/pki/tls/certs/ca-bundle.crt && chmod 600 /etc/pki/tls/certs/ca-bundle.crt
    sed -i 's/\/etc\/pki\/tls\/certs\/localhost.crt/\/etc\/pki\/tls\/certs\/proxy.172.16.12.10.xip.io.crt/' /etc/httpd/conf.d/ssl.conf
    sed -i 's/\/etc\/pki\/tls\/private\/localhost.key/\/etc\/pki\/tls\/private\/proxy.172.16.12.10.xip.io.key/' /etc/httpd/conf.d/ssl.conf
    sed -i 's/#SSLCACertificateFile/SSLCACertificateFile/' /etc/httpd/conf.d/ssl.conf
    sed -i 's/#SSLVerifyClient/SSLVerifyClient/' /etc/httpd/conf.d/ssl.conf
    sed -i 's/#SSLVerifyDepth/SSLVerifyDepth/' /etc/httpd/conf.d/ssl.conf
    echo '#' >> /etc/httpd/conf/httpd.conf
    echo 'ProxyPass /openam               ajp://localhost:8009/openam' >> /etc/httpd/conf/httpd.conf
    echo 'ProxyPassReverse /openam        ajp://localhost:8009/openam' >> /etc/httpd/conf/httpd.conf
    echo '#' >> /etc/httpd/conf/httpd.conf
    echo 'ProxyPass /openidm               http://localhost:7070/openidm' >> /etc/httpd/conf/httpd.conf
    echo 'ProxyPassReverse /openidm        http://localhost:7070/openidm' >> /etc/httpd/conf/httpd.conf
    echo '#' >> /etc/httpd/conf/httpd.conf
    echo 'ProxyPass /idm               http://localhost:7070/idm' >> /etc/httpd/conf/httpd.conf
    echo 'ProxyPassReverse /idm        http://localhost:7070/idm' >> /etc/httpd/conf/httpd.conf
    systemctl enable httpd.service
    systemctl restart httpd.service
    # java
    yum install java-1.8.0-openjdk-devel -y
    # wildfly
    tar xzf /vagrant/wildfly-11.0.0.Final.tar.gz -C /opt
    ln -s /opt/wildfly-11.0.0.Final /opt/wildfly
    groupadd -r wildfly
    useradd -r -g wildfly -d /opt/wildfly -s /sbin/nologin wildfly
    chown -RH wildfly: /opt/wildfly
    chmod +x /opt/wildfly/bin/*.sh
    mkdir -p /etc/wildfly
    cp /opt/wildfly/docs/contrib/scripts/systemd/wildfly.conf /etc/wildfly/
    cp /opt/wildfly/docs/contrib/scripts/systemd/launch.sh /opt/wildfly/bin/
    cp /opt/wildfly/docs/contrib/scripts/systemd/wildfly.service /etc/systemd/system/
    systemctl daemon-reload
    systemctl enable wildfly
    systemctl start wildfly

    # OpenDJ
    sysctl --write fs.inotify.max_user_watches=524288 # TODO: verify this from docs
    groupadd -r opendj
    useradd -r -g opendj -d /opt/opendj -s /sbin/nologin opendj

    echo "Password12345" > /tmp/dspasswd

    # DS for AM Configuration Data
#    /opt/opendj/setup directory-server \
#     --rootUserDN "cn=Directory Manager" \
#     --rootUserPasswordFile /tmp/dspasswd \
#     --monitorUserPasswordFile /tmp/dspasswd \
#     --hostname proxy.172.16.12.10.xip.io \
#     --ldapPort 1389 \
#     --ldapsPort 1636 \
#     --httpsPort 9443 \
#     --adminConnectorPort 4444 \
#     --productionMode \
#     --profile am-config \
#     --set am-config/amConfigAdminPassword:Password12345 \
#     --acceptLicense

  unzip -q /vagrant/DS-eval-6.5.1.zip -d /opt
  chown -R opendj: /opt/opendj

  # DS for AM Identity Data
  sudo -uopendj bash -C /opt/opendj/setup directory-server \
   --rootUserDN "cn=Directory Manager" \
   --rootUserPasswordFile /tmp/dspasswd \
   --monitorUserPasswordFile /tmp/dspasswd \
   --hostname proxy.172.16.12.10.xip.io \
   --ldapPort 1389 \
   --ldapsPort 1636 \
   --httpsPort 9443 \
   --adminConnectorPort 4444 \
   --profile am-identity-store \
   --set am-identity-store/amIdentityStoreAdminPassword:Password12345 \
   --acceptLicense

   # add the base dn
   dsconfig \
    set-backend-prop \
    --hostname proxy.172.16.12.10.xip.io \
    --port 4444 \
    --bindDN "cn=Directory Manager" \
    --bindPassword Password12345 \
    --backend-name amIdentityStore \
    --add base-dn:dc=example,dc=com \
    --trustAll \
    --no-prompt

   # stop DS (required because import happens offline)
   /opt/opendj/bin/stop-ds

   # load the base ldif (example, need to provide actual ldif, this will fail until then)
   opendj/bin/import-ldif -b dc=example,dc=com -n amIdentityStore -l Example.ldif --offline

   # start DS
   /opt/opendj/bin/start-ds

   # TODO: OpenAM realm doesn't point to our baseDn by default, had to edit it manually

   #--baseDn dc=example,dc=com \
   #--addBaseEntry \

   #--productionMode \

   # DS -- it is possible to use a single DS to support AM Identity Data, AM Configuration Data, AM CTS and even Policy data.
   # it supports multiple profiles per "setup" command if we want to.
   # for now, we'll stick with Identity data only.
   # https://backstage.forgerock.com/docs/am/6.5/install-guide/#prepare-ext-stores

   # Amster
   mkdir /opt/amster_6.5.1
   unzip -q /vagrant/Amster-6.5.1.zip -d /opt/amster_6.5.1
   export JAVA_HOME=/usr/lib/jvm/java

   # AM
   # edit /opt/wildfly/bin/standalone.conf:
   #JAVA_OPTS="-Xms1024m -Xmx1024m -XX:MetaspaceSize=256M -XX:MaxMetaspaceSize=256m -Djava.net.preferIPv4Stack=true"
   #JAVA_OPTS="$JAVA_OPTS -Djboss.modules.system.pkgs=$JBOSS_MODULES_SYSTEM_PKGS -Djava.awt.headless=true"
   #JAVA_OPTS="$JAVA_OPTS -Dorg.apache.tomcat.util.http.ServerCookie.ALWAYS_ADD_EXPIRES=true"
   #JAVA_OPTS="$JAVA_OPTS -Dorg.forgerock.openam.ldap.secure.protocol.version=TLSv1.2"

   # inside the war, edit the file: WEB-INF/classes/bootstrap.properties
   # uncomment the config.dir property, and set as follows:
   # configuration.dir=/opt/am-config

   # provide am configuration folder
   mkdir /opt/am-config
   chown wildfly: /opt/am-config

   # extrac am, modify war for use with Wildfly
   unzip -q /vagrant/AM-eval-6.5.1.zip -d /opt
   mkdir /tmp/am-boot; pushd /tmp/am-boot
   jar xf /opt/openam/AM-eval-6.5.1.war WEB-INF/classes/bootstrap.properties
   sed -i 's/# configuration.dir=$/configuration.dir=\/opt\/am-config/' WEB-INF/classes/bootstrap.properties 
   jar uf /opt/openam/AM-eval-6.5.1.war WEB-INF/classes/bootstrap.properties
   popd

   # deploy war (we'll stop jboss to be safe)
   systemctl stop wildfly
   cp /opt/openam/AM-eval-6.5.1.war /opt/wildfly/standalone/deployments/openam.war
   chown wildfly: /opt/wildfly/standalone/deployments/openam.war
   systemctl start wildfly

   /opt/wildfly/bin/jboss-cli.sh --connect --commands="/subsystem=undertow/server=default-server/ajp-listener=ajpListener:add(socket-binding=ajp, scheme=https, enabled=true)"

   # for EAP / AS we would remove WEB-INF/jboss-all.xml

   # IDM
   groupadd -r openidm
   useradd -r -g openidm -d /opt/openidm -s /sbin/nologin openidm
   unzip -q /vagrant/IDM-eval-6.5.0.1.zip -d /opt
   chown -RH openidm: /opt/openidm

   keytool -importcert -file /vagrant/certs/certgen/ca.crt -keystore /opt/openidm/security/truststore -alias bcm-devel-ca -trustcacerts

   sed -i 's/openidm.port.http=8080/openidm.port.http=7070/' /opt/openidm/resolver/boot.properties
   sed -i 's/openidm.port.https=8443/openidm.port.https=7443/' /opt/openidm/resolver/boot.properties
   sed -i 's/openidm.port.mutualauth=8444/openidm.port.mutualauth=7444/' /opt/openidm/resolver/boot.properties
   sed -i 's/openidm.auth.clientauthonlyports=8444/openidm.auth.clientauthonlyports=7444/' /opt/openidm/resolver/boot.properties
   sed -i 's/openidm.host=localhost/openidm.host=proxy.172.16.12.10.xip.io/' /opt/openidm/resolver/boot.properties

   # change paths because behind proxy
   sed -i 's/"urlContextRoot" : "\//"urlContextRoot" : "\/idm\//' /opt/openidm/conf/ui.context-admin.json
   sed -i 's/"urlContextRoot" : "\//"urlContextRoot" : "\/idm\//' /opt/openidm/conf/ui.context-api.json
   sed -i 's/"urlContextRoot" : "\//"urlContextRoot" : "\/idm\//' /opt/openidm/conf/ui.context-oauth.json
   # this one is base path
   sed -i 's/"urlContextRoot" : "\//"urlContextRoot" : "\/idm/' /opt/openidm/conf/ui.context-enduser.json

   # TODO: see how to run as a service
   # for now, following full stack samples
   # run like this:
   #  sudo -uopenidm bash -C ./startup.sh -p samples/full-stack
   
#    # Extract web policy agent
#    unzip /vagrant/Apache_v22_Linux_64bit_4.0.0.zip -d /opt
#    chown -R apache:apache /opt/web_agents
  SHELL
end
