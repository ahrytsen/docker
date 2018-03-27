FROM	debian

RUN		apt-get update && apt-get upgrade -y
RUN		apt-get install -y wget bzip2

RUN		adduser --disabled-login teamspeak
RUN		wget -O teamspeak_server.tar.bz2\
		 	 http://dl.4players.de/ts/releases/3.1.1/teamspeak3-server_linux_amd64-3.1.1.tar.bz2

RUN		mkdir teamspeak_server\
		&& tar xvf teamspeak_server.tar.bz2 -C teamspeak_server --strip-components=1
RUN		mv teamspeak_server/* /home/teamspeak && rm -rf teamspeak_server*
RUN 	chown -R teamspeak:teamspeak /home/teamspeak

USER	  teamspeak
WORKDIR	  /home/teamspeak
COPY 	  server.ini /home/teamspeak

EXPOSE 	  9987/udp
EXPOSE 	  30033
EXPOSE 	  10011
EXPOSE 	  41144

ENTRYPOINT	./ts3server_minimal_runscript.sh inifile=server.ini license_accepted=1
