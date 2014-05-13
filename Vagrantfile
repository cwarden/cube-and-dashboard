VAGRANTFILE_API_VERSION = "2"

$script = <<SCRIPT
# Install required packages
sudo apt-get update
sudo apt-get -y install git mongodb g++ stunnel4 curl make

# Setup stunnel for SSL access
if [ ! -f /etc/stunnel/cert.pem ]; then
  cd /etc/stunnel
  sudo openssl genrsa -out key.pem 2048
  sudo openssl req -new -x509 -key key.pem -out cert.pem -days 3650 -subj /C=US/ST=Oregon/L=Portland/O=GreatVines/CN=localhost/
fi
# Copy cert back to host so it can be imported into keystore for browser
sudo cp /etc/stunnel/cert.pem /srv/stunnel
sudo ln -nsf /srv/stunnel/cube.conf /etc/stunnel/
sudo perl -pi -e 's/ENABLED=./ENABLED=1/' /etc/default/stunnel4

# Install correct version of node.js
test -d ~/.nvm || git clone https://github.com/creationix/nvm.git ~/.nvm
source ~/.nvm/nvm.sh
nvm install 0.10
nvm use 0.10

# Start up cube and log-dashboard
npm install -g supervisor foreman

cd /srv/cube
test -f .env || cp .env.example .env
npm install
sudo nf -p 1080 -a cube -u vagrant export -o /etc/init

cd /srv/log-dashboard
test -f .env || cp .env.example .env
npm install
sudo nf -p 3000 -a dashboard -u vagrant -j Procfile.debug export -o /etc/init

sudo service stunnel4 restart
sudo service cube restart
sudo service dashboard restart

SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.define "mongodb" do |mongodb|
    mongodb.vm.provider "docker" do |d|
      d.image = "robinvdvleuten/mongo"
    end
  end

  config.vm.define "collector" do |collector|
    collector.vm.provider "docker" do |d|
      d.build_dir = "cube"
      d.link "mongodb"
      d.env SERVICE: "collector", COLLECTOR_AUTHENTICATOR: "trusted_recording_signed_request"
    end
    collector.vm.network :forwarded_port, guest: 1080, host: 1080
    collector.vm.synced_folder "cube/", "/srv/cube"
  end

  config.vm.define "evaluator" do |evaluator|
    evaluator.vm.provider "docker" do |d|
      d.build_dir = "cube"
      d.link "mongodb"
      d.env SERVICE: "evaluator", EVALUATOR_AUTHENTICATOR: "allow_all"
    end
    evaluator.vm.network :forwarded_port, guest: 1080, host: 1181
    evaluator.vm.synced_folder "cube/", "/srv/cube"
  end


  # Ports for cube collector, evaluator, and log dashboard
#  config.vm.network :forwarded_port, guest: 3000, host: 3000
  # SSL ports
#  config.vm.network :forwarded_port, guest: 10800, host: 10800
#  config.vm.network :forwarded_port, guest: 11800, host: 11800
#  config.vm.network :forwarded_port, guest: 30000, host: 30000
#  config.vm.hostname = "cube.local"

#  config.vm.provider :virtualbox do |vb|
#    vb.customize ["modifyvm", :id, "--memory", 768]
#  end
#  config.vm.synced_folder "log-dashboard/", "/srv/log-dashboard"
#  config.vm.synced_folder "stunnel/", "/srv/stunnel"
#
#  config.vm.provision "shell", inline: $script, privileged: false

end
