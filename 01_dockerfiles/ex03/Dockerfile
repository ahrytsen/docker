FROM debian

RUN apt-get update -y && apt-get upgrade -y
RUN apt-get install -y sudo git-core wget build-essential zlib1g-dev libyaml-dev\
	libssl-dev libgdbm-dev libre2-dev libreadline-dev libncurses5-dev libffi-dev\
	openssh-server checkinstall libxml2-dev libxslt-dev libcurl4-openssl-dev libicu-dev\
	curl logrotate rsync python-docutils pkg-config cmake libkrb5-dev postfix nodejs yarn\
	postgresql postgresql-client libpq-dev postgresql-contrib redis-server nginx
RUN apt-get remove -y ruby1.8
RUN	mkdir /tmp/ruby && cd /tmp/ruby
RUN curl --remote-name --progress https://cache.ruby-lang.org/pub/ruby/2.3/ruby-2.3.6.tar.gz
RUN tar xzf ruby-2.3.6.tar.gz && cd ruby-2.3.6 && ./configure --disable-install-rdoc && make\
	&& make install
RUN	gem install bundler --no-ri --no-rdoc

# INSTALL GO
RUN curl --remote-name --progress https://storage.googleapis.com/golang/go1.8.3.linux-amd64.tar.gz
RUN	tar -C /usr/local -xzf go1.8.3.linux-amd64.tar.gz
RUN	ln -sf /usr/local/go/bin/{go,godoc,gofmt} /usr/local/bin/
RUN rm go1.8.3.linux-amd64.tar.gz
RUN adduser --disabled-login --gecos 'GitLab' git

RUN sudo -u postgres psql -d template1 -c "CREATE USER git CREATEDB;"
RUN sudo -u postgres psql -d template1 -c "CREATE EXTENSION IF NOT EXISTS pg_trgm;"
RUN sudo -u postgres psql -d template1 -c "CREATE DATABASE gitlabhq_production OWNER git;"

# We'll install GitLab into home directory of the user "git"
WORKDIR /home/git
# Clone GitLab repository
RUN sudo -u git -H git clone https://gitlab.com/gitlab-org/gitlab-ce.git -b master gitlab
# Go to GitLab installation folder
WORKDIR /home/git/gitlab
# Copy the example GitLab config
RUN sudo -u git -H cp config/gitlab.yml.example config/gitlab.yml
# Copy the example secrets file
RUN sudo -u git -H cp config/secrets.yml.example config/secrets.yml
RUN sudo -u git -H chmod 0600 config/secrets.yml
# Make sure GitLab can write to the log/ and tmp/ directories
RUN sudo chown -R git log/
RUN sudo chown -R git tmp/
RUN sudo chmod -R u+rwX,go-w log/
RUN sudo chmod -R u+rwX tmp/
# Make sure GitLab can write to the tmp/pids/ and tmp/sockets/ directories
RUN sudo chmod -R u+rwX tmp/pids/
RUN sudo chmod -R u+rwX tmp/sockets/
# Create the public/uploads/ directory
RUN sudo -u git -H mkdir public/uploads/
# Make sure only the GitLab user has access to the public/uploads/ directory
# now that files in public/uploads are served by gitlab-workhorse
RUN sudo chmod 0700 public/uploads
# Change the permissions of the directory where CI job traces are stored
RUN sudo chmod -R u+rwX builds/
# Change the permissions of the directory where CI artifacts are stored
RUN sudo chmod -R u+rwX shared/artifacts/
# Change the permissions of the directory where GitLab Pages are stored
RUN sudo chmod -R ug+rwX shared/pages/
# Copy the example Unicorn config
RUN sudo -u git -H cp config/unicorn.rb.example config/unicorn.rb
# Copy the example Rack attack config
RUN sudo -u git -H cp config/initializers/rack_attack.rb.example config/initializers/rack_attack.rb
# Configure Git global settings for git user
# 'autocrlf' is needed for the web editor
RUN sudo -u git -H git config --global core.autocrlf input
# Disable 'git gc --auto' because GitLab already runs 'git gc' when needed
RUN sudo -u git -H git config --global gc.auto 0
# Enable packfile bitmaps
RUN sudo -u git -H git config --global repack.writeBitmaps true
# Enable push options
RUN sudo -u git -H git config --global receive.advertisePushOptions true
# Configure Redis connection settings
RUN sudo -u git -H cp config/resque.yml.example config/resque.yml
# PostgreSQL only:
RUN sudo -u git cp config/database.yml.postgresql config/database.yml
# For PostgreSQL (note, the option says "without ... mysql")
RUN sudo -u git -H bundle install --deployment --without development test mysql aws kerberos
# Run the installation task for gitlab-shell (replace `REDIS_URL` if needed):
RUN sudo -u git -H bundle exec rake gitlab:shell:install\
	REDIS_URL=unix:/var/run/redis/redis.sock RAILS_ENV=production SKIP_STORAGE_VALIDATION=true
RUN sudo -u git -H bundle exec rake "gitlab:workhorse:install[/home/git/gitlab-workhorse]"\
	RAILS_ENV=production
RUN sudo -u git -H bundle exec rake gitlab:setup RAILS_ENV=production
RUN sudo cp lib/support/init.d/gitlab /etc/init.d/gitlab
RUN sudo update-rc.d gitlab defaults 21
RUN sudo -u git -H bundle exec rake "gitlab:gitaly:install[/home/git/gitaly]" RAILS_ENV=production
# Restrict Gitaly socket access
RUN sudo chmod 0700 /home/git/gitlab/tmp/sockets/private
RUN sudo chown git /home/git/gitlab/tmp/sockets/private
RUN sudo cp lib/support/logrotate/gitlab /etc/logrotate.d/gitlab
RUN sudo -u git -H bundle exec rake gettext:compile RAILS_ENV=production
RUN sudo -u git -H yarn install --production --pure-lockfile
RUN sudo -u git -H bundle exec rake gitlab:assets:compile RAILS_ENV=production NODE_ENV=production
RUN sudo cp lib/support/nginx/gitlab /etc/nginx/sites-available/gitlab
RUN sudo ln -s /etc/nginx/sites-available/gitlab /etc/nginx/sites-enabled/gitlab
RUN sudo nginx -t
RUN sudo service nginx restart
EXPOSE 443 80 22
# Define data volumes
VOLUME ["/etc/gitlab", "/var/opt/gitlab", "/var/log/gitlab"]
ENTRYPOINT sudo service gitlab restart
