Vagrant.configure("2") do |config|
  
 config.vm.box = "ubuntu/focal64"

 config.vm.provider "virtualbox" do |vb|
   vb.memory = "4096"
   vb.cpus = "3"
 end
  config.vm.hostname = "router"      # Name needs to be present for fleet server to work properly out of the box
  # Kibana port forwarding
  config.vm.network "forwarded_port", guest: 5601, host: 5601
  
  
  
  # Elasticsearch
  config.vm.provision "shell", inline: <<-SHELL
    wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -  
    sudo apt-get install apt-transport-https -y
    echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
    sudo apt-get update && sudo apt-get install elasticsearch -y
    sudo /bin/systemctl daemon-reload
    sudo /bin/systemctl enable elasticsearch.service
  SHELL
  config.vm.provision "file", source: "elasticsearch.yml", destination: "~/elasticsearch.yml"
  config.vm.provision "shell", inline: <<-SHELL
    sudo mv -f /home/vagrant/elasticsearch.yml /etc/elasticsearch/elasticsearch.yml
    sudo systemctl start elasticsearch.service
  SHELL
    
  #Kibana
  config.vm.provision "shell", inline: <<-SHELL
    sudo apt install kibana -y
    sudo /bin/systemctl daemon-reload
    sudo /bin/systemctl enable kibana.service
  SHELL
  config.vm.provision "file", source: "kibana.yml", destination: "~/kibana.yml"  
  config.vm.provision "shell", inline: <<-SHELL
    echo $(yes | sudo /usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto) >> /home/vagrant/elasticpass
    echo "elasticsearch.password: $( cat /home/vagrant/elasticpass | sed 's/PASSWORD/\\n/g' | grep "kibana_system =" | \
           awk {'print $3'} )" >> /home/vagrant/kibana.yml
    sudo mv -f /home/vagrant/kibana.yml /etc/kibana/
    sudo chown root.kibana /etc/kibana/kibana.yml  
    sudo systemctl start kibana.service
    SHELL

    #Elastic-agent + fleet server
    config.vm.provision "shell", inline: <<-SHELL
      sudo apt-get install jq -y #JSON text filter
      #wget https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-7.15.2-linux-x86_64.tar.gz
      #tar -xzf elastic-agent-7.15.2-linux-x86_64.tar.gz
      wget https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-7.17.0-linux-x86_64.tar.gz
      tar -xzf elastic-agent-7.17.0-linux-x86_64.tar.gz

      # cd elastic-agent-7.15.2-linux-x86_64
      # For manual installation first log in and create a fleet from kibana
      # !!! Replace sed 's/PASSWORD/\\n/g' with sed 's/PASSWORD/\n/g' for manual fleet creation !!!
      #  sudo ./elastic-agent install -f --fleet-server-es=http://localhost:9200 \
      #  --fleet-server-service-token=$(curl -k -u "elastic:$(cat /home/vagrant/elasticpass | sed 's/PASSWORD/\\n/g' | grep "elastic =" | awk {'print $3'})" \
      #  -X POST http://localhost:5601/api/fleet/service-tokens --header 'kbn-xsrf: true' | jq | grep value | awk {' print $2 '} | tr -d '"')
    SHELL


    # Updating the system
  config.vm.provision "shell", inline: <<-SHELL
    sudo apt remove multipath-tools -y  #we don't have devices for this daemon and it just spams in log-files
    sudo apt-get update
    sudo apt-get full-upgrade -y
  SHELL

  config.vm.provision "file", source: "vagrant.pub", destination: "/home/vagrant/vagrant.pub"  #default vagrant key

end

#To make a box, don't forget to import a vagrant.pub before exiting from ssh session.