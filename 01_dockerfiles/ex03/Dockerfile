FROM	debian

RUN		apt-get update -y && apt-get upgrade -y
RUN		apt-get install -y postgresql postgresql-contrib libpq-dev redis-server\
		curl libicu-dev cmake g++ libkrb5-dev libre2-dev ed pkg-config\
		build-essential zlib1g-dev libyaml-dev libssl1.0-dev libgdbm-dev libre2-dev\
		libreadline-dev libncurses5-dev libffi-dev  openssh-server checkinstall\
		libxml2-dev libxslt-dev libcurl4-openssl-dev logrotate rsync git-core\
		python-docutils pkg-config libsqlite3-dev
#GOLANG
RUN		curl --remote-name --progress\
		https://storage.googleapis.com/golang/go1.8.3.linux-amd64.tar.gz
RUN		tar -C /usr/local -xzf go1.8.3.linux-amd64.tar.gz
RUN		ln -sf /usr/local/go/bin/{go,godoc,gofmt} /usr/local/bin/
RUN		rm go1.8.3.linux-amd64.tar.gz
#NODEjs
RUN		curl --location https://deb.nodesource.com/setup_8.x | bash -
RUN		apt-get install -y nodejs
#YARN
RUN		curl --silent --show-error https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN		echo "deb https://dl.yarnpkg.com/debian/ stable main"\
		| tee /etc/apt/sources.list.d/yarn.list
RUN		apt-get install -y apt-transport-https
RUN		apt-get update -y && apt-get install -y yarn

RUN		apt-get remove -y ruby
#RUBY
RUN		adduser --disabled-login --gecos 'GitLab' git
USER	git
WORKDIR	/home/git
RUN		git clone https://github.com/rbenv/rbenv.git .rbenv
WORKDIR	/home/git/.rbenv
RUN		src/configure && make -C src
ENV		PATH $PATH:/usr/local/go/bin:/home/git/.gem/ruby/2.3.6/bin:/home/git/.rbenv/bin:/home/git/.rbenv/shims
RUN		mkdir -p "$(rbenv root)"/plugins
RUN		git clone https://github.com/rbenv/ruby-build.git "$(rbenv root)"/plugins/ruby-build
RUN		eval "$(rbenv init -)"
RUN		rbenv install 2.3.6 && ~/.rbenv/bin/rbenv global 2.3.6
WORKDIR	/home/git
RUN		gem install bundler --no-ri --no-rdoc
RUN		gem install sqlite3 -v '1.3.13'
RUN 	gem install rails
RUN		rails db:migrate
RUN		rails db:seed
RUN		gem install gitlab-development-kit
RUN		gdk init
WORKDIR	/home/git/gitlab-development-kit
RUN		gdk install

EXPOSE	3000 443 80 22
ENTRYPOINT gdk run db && gdk run app
