FROM ruby

RUN gem install rails
RUN apt-get update -y && apt-get upgrade && apt-get install -y nodejs
ONBUILD COPY . /opt/app

ONBUILD WORKDIR /opt/app

ONBUILD RUN bundle install
ONBUILD RUN rails db:migrate
ONBUILD RUN rails db:seed
