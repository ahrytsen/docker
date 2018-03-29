FROM	debian

RUN		apt-get update -y && apt-get upgrade -y
RUN		apt-get install -y postgresql postgresql-contrib libpq-dev redis-server\
		curl libicu-dev cmake g++ libkrb5-dev libre2-dev pkg-config\
		build-essential zlib1g-dev libyaml-dev libssl1.0-dev libgdbm-dev\
		libreadline-dev libncurses5-dev libffi-dev  openssh-server checkinstall\
		libxml2-dev libxslt-dev logrotate rsync ed\
		python-docutils libsqlite3-dev dh-autoreconf libcurl4-gnutls-dev libexpat1-dev \
  		gettext libz-dev
#libcurl4-openssl-dev
#GOLANG
RUN		curl --remote-name --progress\
		https://storage.googleapis.com/golang/go1.8.3.linux-amd64.tar.gz
RUN		tar -C /usr/local -xzf go1.8.3.linux-amd64.tar.gz
RUN		ln -sf /usr/local/go/bin/{go,godoc,gofmt} /usr/local/bin/
RUN		rm go1.8.3.linux-amd64.tar.gz
#GIT
RUN		apt-get remove -y git
RUN		mkdir /tmp/git
WORKDIR	/tmp/git
RUN		curl -OL --remote-name --progress https://www.kernel.org/pub/software/scm/git/git-2.14.3.tar.gz
RUN		tar -xzf git-2.14.3.tar.gz
WORKDIR /tmp/git/git-2.14.3
RUN		./configure
RUN		make prefix=/usr/local all
RUN		make prefix=/usr/local install
WORKDIR	/
RUN		rm -rf /tmp/git
#NODEjs
RUN		curl --location https://deb.nodesource.com/setup_8.x | bash -
RUN		apt-get install -y nodejs npm
#YARN
RUN		curl --silent --show-error https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN		echo "deb https://dl.yarnpkg.com/debian/ stable main"\
		| tee /etc/apt/sources.list.d/yarn.list
RUN		apt-get install -y apt-transport-https
RUN		apt-get update -y && apt-get install -y yarn

#RUBY
RUN		apt-get remove -y ruby
RUN		adduser --disabled-login --gecos 'GitLab' git
USER	git
WORKDIR	/home/git
RUN		git clone https://github.com/rbenv/rbenv.git .rbenv
WORKDIR	/home/git/.rbenv
RUN		src/configure && make -C src
ENV		PATH $PATH:/usr/local/bin/git:/usr/local/go/bin:/home/git/.rbenv/versions/2.3.6/bin:/home/git/.rbenv/bin:/home/git/.rbenv/shims
RUN		mkdir -p "$(rbenv root)"/plugins
RUN		git clone https://github.com/rbenv/ruby-build.git "$(rbenv root)"/plugins/ruby-build
RUN		eval "$(rbenv init -)"
RUN		rbenv install 2.3.6 && rbenv global 2.3.6
WORKDIR	/home/git
RUN		gem install bundler --no-ri --no-rdoc
RUN		gem install sqlite3 -v '1.3.13'
RUN		gem install gitlab-development-kit
RUN		gdk init
WORKDIR	/home/git/gitlab-development-kit
RUN		ln -s . gitlab-development-kit
RUN		gdk install
EXPOSE	3000 443 80 22 2222
ENTRYPOINT gdk run
