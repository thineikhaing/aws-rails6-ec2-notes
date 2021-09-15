# Generate Random Passwords
curl 'https://www.random.org/passwords/?num=2&len=24&format=plain&rnd=new' 

# Create deploy user
sudo adduser deploy 
sudo adduser deploy sudo 
su deploy 
cd ../deploy/ 

# Generate ssh keys  
ssh-keygen -t rsa -b 4096 -C "[your-email-here]"
cat ~/.ssh/id_rsa.pub 
sudo nano .ssh/authorized_keys 
[Paste your laptops id_rsa.pub public key in authorized_keys]

# Add repositories
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo add-apt-repository ppa:chris-lea/redis-server

# Install dependencies
sudo apt-get update; sudo apt-get install apt-transport-https build-essential ca-certificates curl dirmngr g++ gcc gifsicle git-core gnupg jpegoptim libcurl4-openssl-dev libffi-dev libpq-dev libqt5webkit5-dev libreadline-dev libsqlite3-dev libssl-dev libxml2-dev libxslt1-dev libyaml-dev make nodejs optipng postgresql postgresql-contrib qt5-default redis-server redis-tools ruby-dev software-properties-common sqlite3 yarn zlib1g-dev -y;

# Install rbenv and ruby-build
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
git clone https://github.com/rbenv/rbenv-vars.git ~/.rbenv/plugins/rbenv-vars

# Configure rbenv and ruby-build
sudo nano ~/.bashrc
[paste the following in bashrc]
  export PATH="$HOME/.rbenv/bin:$PATH"
  eval "$(rbenv init -)"
  export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"

exec $SHELL

# Install ruby 
rbenv install 2.7.1
rbenv global 2.7.1

# Install Bundler
gem install bundler

# Install Passenger and NGINX 
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 561F9B9CAC40B2F7
sudo sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger focal main > /etc/apt/sources.list.d/passenger.list'
sudo apt-get update
sudo apt-get install -y nginx-extras libnginx-mod-http-passenger
if [ ! -f /etc/nginx/modules-enabled/50-mod-http-passenger.conf ]; then sudo ln -s /usr/share/nginx/modules-available/mod-http-passenger.load /etc/nginx/modules-enabled/50-mod-http-passenger.conf ; fi
sudo ls /etc/nginx/conf.d/mod-http-passenger.conf

# Configure Passenger
sudo nano /etc/nginx/conf.d/mod-http-passenger.conf
replace free ruby version with: passenger_ruby /home/deploy/.rbenv/shims/ruby;

# Configure NGINX
sudo nano /etc/nginx/sites-enabled/aws-rails.com
server {
  listen 80;
  listen [::]:80;

  server_name aws-rails.com;
  root /home/deploy/aws_rails/current/public;

  passenger_enabled on;
  passenger_app_env production;

  location /cable {
    passenger_app_group_name aws_rails_websocket;
    passenger_force_max_concurrent_requests_per_process 0;
  }

  client_max_body_size 200m;

  location ~ ^/(assets|packs) {
    expires max;
    gzip_static on;
  }
}

sudo service nginx start
