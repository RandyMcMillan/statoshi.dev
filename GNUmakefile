
# The repo may contain a Makefile generated by ./configure
# We include Makefile at the end of this GNUmakefile
# Make searches for this file first first per make default search hierarchy
ifneq ($(Makefile),)
	Makefile := defined
endif

#These are referenced here and docker-compose.yml
DOCKERFILE         := $(notdir $(PWD))
DOCKERFILE_SLIM    := $(notdir $(PWD)).slim
DOCKERFILE_GUI     := $(notdir $(PWD)).gui
DOCKERFILE_EXTRACT := $(notdir $(PWD)).extract

# If you see pwd_unknown showing up, this is why. Re-calibrate your system.
PWD ?= pwd_unknown

# Note the different service configs in  docker-compose.yml.
# We override this default for different build/run configs
SERVICE_TARGET := shell

ifeq ($(user),)
# We force root
HOST_USER := root
HOST_UID  := 0
## USER retrieved from env, UID from shell.
#HOST_USER ?= $(strip $(if $(USER),$(USER),nodummy))
#HOST_UID  ?=  $(strip $(if $(shell id -u),$(shell id -u),4000))
else
# We force root
HOST_USER := root
HOST_UID  := 0
## allow override by adding user= and/ or uid=  (lowercase!).
## uid= defaults to 0 if user= set (i.e. root).
#HOST_USER = $(user)
#HOST_UID = $(strip $(if $(uid),$(uid),0))
endif
# PROJECT_NAME defaults to name of the current directory.
# should not need to be changed if you follow GitOps operating procedures.
PROJECT_NAME := $(notdir $(PWD))

THIS_FILE := $(lastword $(MAKEFILE_LIST))
CMD_ARGUMENTS ?= $(cmd)

# export such that its passed to shell functions for Docker to pick up.
# control alpine version from here
VERSION := 3.11.6

export VERSION
export PROJECT_NAME
export HOST_USER
export HOST_UID

# all our targets are phony (no files to check).
.PHONY: help init shell build-shell rebuild-shell service login concat-all build-all run-all make-statoshi run-statoshi extract concat-slim build-slim rebuild-slim run-slim concat-gui build-gui rebuild-gui run-gui test-gui autogen depends config doc concat package-all package-gui package-slim

# suppress make's own output
#.SILENT:

help:
	@echo ''
	@echo ''
	@echo '	Docker: make [TARGET] [EXTRA_ARGUMENTS]'
	@echo '	Shell:'
	@echo '		make shell user=$(HOST_USER)'
	@echo ''
	@echo '	Targets:'
	@echo ''
	@echo '		build-all	complete build - no deploy'
	@echo '		run-all  	deploy build-all product'
	@echo ''
	@echo '		build-slim	build with signed precompiled statoshi binaries'
	@echo '		run-slim	deploy build-slim product'
	@echo ''
	@echo '	Extra: push a shell command to the container'
	@echo ''
	@echo '		cmd=:	    make shell cmd="whoami"'
	@echo ''
	@echo ''

#######################
# Shared volume for datadir persistance
# We avoid a user's bitcoin config when sharing $HOME with container
########################
init:
	@echo ''
	bash -c 'mkdir -p $(HOME)/.statoshi'
	bash -c 'install -v conf/bitcoin.conf $(HOME)/.statoshi'
	@echo ''

#######################
# Docker file creation...
########################
concat-all: init
	@echo ''
	bash -c '$(pwd) cat ./docker/header               > $(DOCKERFILE)'
	bash -c '$(pwd) cat ./docker/statoshi.all        >> $(DOCKERFILE)'
	bash -c '$(pwd) cat ./docker/footer              >> $(DOCKERFILE)'
	@echo ''
#######################
concat-slim:
	@echo ''
	bash -c '$(pwd) cat ./docker/header               > $(DOCKERFILE).slim'
	bash -c '$(pwd) cat ./docker/statoshi.slim       >> $(DOCKERFILE).slim'
	bash -c '$(pwd) cat ./docker/footer              >> $(DOCKERFILE).slim'
	@echo ''
#######################
concat-gui:
	@echo ''
	bash -c '$(pwd) cat ./docker/header.slim          > $(DOCKERFILE).gui'
	bash -c '$(pwd) cat ./docker/gui                 >> $(DOCKERFILE).gui'
	bash -c '$(pwd) cat ./docker/footer              >> $(DOCKERFILE).gui'
	@echo ''
#######################
concat: concat-all concat-slim concat-gui
	@echo ''
	bash -c ' install -v ./docker/docker-compose.yml .'
	bash -c ' install -v ./docker/shell .'
	@echo ''
#######################
build-shell: concat
	docker-compose build shell
#######################
rebuild-shell: concat
	docker-compose build --no-cache shell
#######################
shell: build-shell
ifeq ($(CMD_ARGUMENTS),)
	# no command is given, default to shell
	docker-compose -p $(PROJECT_NAME)_$(HOST_UID) run --rm shell sh
else
	# run the command
	docker-compose -p $(PROJECT_NAME)_$(HOST_UID) run --rm shell sh -c "$(CMD_ARGUMENTS)"
endif
#######################
autogen: concat
	# here it is useful to add your own customised tests
	docker-compose -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) run --rm statoshi sh -c "cd /home/root/stats.bitcoincore.dev  && ./autogen.sh && exit"
#######################
config: autogen
	docker-compose -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) run --rm statoshi sh -c "cd /home/root/stats.bitcoincore.dev  && ./configure --disable-wallet --disable-tests --disable-hardening --disable-man --enable-util-cli --enable-util-tx --with-gui=no --without-miniupnpc --disable-bench && exit"
#######################
depends: config
	docker-compose -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) run --rm statoshi sh -c "apk add coreutils && exit"
	docker-compose -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) run --rm statoshi sh -c "make -j $(nproc) download -C /home/root/stats.bitcoincore.dev/depends && exit"
#######################
test:
	docker-compose -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) run -d --rm shell sh -c '\
	echo "I am `whoami`. My uid is `id -u`." && echo "Docker runs!"' \
	&& echo success
#######################
make-statoshi: depends
	docker-compose -p $(PROJECT_NAME)_$(HOST_UID) run --rm statoshi sh -c "cd /home/root/stats.bitcoincore.dev && make install && exit"
#######################
run-statoshi: make-statoshi
	docker-compose build statoshi
ifeq ($(CMD_ARGUMENTS),)
	docker-compose -p $(PROJECT_NAME)_$(HOST_UID) run -d --rm statoshi sh
else
	docker-compose -p $(PROJECT_NAME)_$(HOST_UID) run -d --rm statoshi sh -c "$(CMD_ARGUMENTS)"
endif
#######################
service:
	# run as a (background) service
	docker-compose -p $(PROJECT_NAME)_$(HOST_UID) up -d shell
#######################
login: service
	# run as a service and attach to it
	docker exec -it $(PROJECT_NAME)_$(HOST_UID) sh
########################
build-all: concat
	docker-compose -f docker-compose.yml build statoshi
#######################
rebuild-all: concat
	docker-compose -f docker-compose.yml build --no-cache statoshi
#######################
run-all: build-all
ifeq ($(CMD_ARGUMENTS),)
	docker-compose -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) run -d --publish 3000:3000 --publish 8080:8080 --publish 8125:8125 --publish 8126:8126 --rm statoshi sh
else
	docker-compose -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) run -d --publish 3000:3000 --publish 8080:8080 --publish 8125:8125 --publish 8126:8126 --rm statoshi sh -c "$(CMD_ARGUMENTS)"
endif
#######################
extract: concat
	#extract TODO CREATE PACKAGE for distribution
	sed '$d' $(DOCKERFILE) | sed '$d' | sed '$d' > $(DOCKERFILE_EXTRACT)
	docker build -f $(DOCKERFILE_EXTRACT) --rm -t $(DOCKERFILE_EXTRACT) .
	docker run --name $(DOCKERFILE_EXTRACT) $(DOCKERFILE_EXTRACT) /bin/true
	docker rm $(DOCKERFILE_EXTRACT)
	rm -f  $(DOCKERFILE_EXTRACT)
#######################
build-slim: concat
	docker-compose -f docker-compose.yml -p stats.bitcoincore.dev build statoshi-slim
#######################
rebuild-slim: concat
	docker-compose -f docker-compose.yml -p stats.bitcoincore.dev build --no-cache statoshi-slim
#######################
run-slim: build-slim
ifeq ($(CMD_ARGUMENTS),)
	#docker-compose -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) run -d --publish 3000:3000 --publish 8080:8080 --publish 8125:8125 --publish 8126:8126 --rm statoshi-slim sh
	docker-compose -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) run --publish 3000:3000 --publish 8080:8080 --publish 8125:8125 --publish 8126:8126 --rm statoshi-slim sh
	#docker-compose up -d -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) --publish 3000:3000 --publish 8080:8080 --publish 8125:8125 --publish 8126:8126 --rm statoshi-slim
else
	#docker-compose -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) run -d --publish 3000:3000 --publish 8080:8080 --publish 8125:8125 --publish 8126:8126 --rm statoshi-slim sh -c "$(CMD_ARGUMENTS)"
	docker-compose -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) run --publish 3000:3000 --publish 8080:8080 --publish 8125:8125 --publish 8126:8126 --rm statoshi-slim sh -c "$(CMD_ARGUMENTS)"
	#docker-compose up -d -f docker-compose.yml up -d -p $(PROJECT_NAME)_$(HOST_UID) --publish 3000:3000 --publish 8080:8080 --publish 8125:8125 --publish 8126:8126 --rm statoshi-slim run sh -c "$(CMD_ARGUMENTS)"
endif
#######################
build-gui: concat
	docker-compose -f docker-compose.yml -p $(PROJECT_NAME) build gui
	@echo ''
#######################
rebuild-gui: concat
	docker-compose -f docker-compose.yml -p $(PROJECT_NAME) build --no-cache gui
	@echo ''
#######################
run-gui: build-gui
ifeq ($(CMD_ARGUMENTS),)
	docker-compose -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) run -d --publish 80:3000 --rm gui sh
	@echo ''
else
	docker-compose -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) run -d --publish 80:3000 --rm gui sh -c "$(CMD_ARGUMENTS)"
	@echo ''
endif
#######################
test-gui: build-gui
	docker-compose -f docker-compose.yml -p $(PROJECT_NAME)_$(HOST_UID) run -d --publish 3333:3000 --rm gui sh -c '\
		echo "I am `whoami`. My uid is `id -u`." && echo "Docker runs!"' \
	&& echo success
#######################
clean:
	@docker-compose -p $(PROJECT_NAME)_$(HOST_UID) down --remove-orphans --rmi all 2>/dev/null \
	&& echo 'Image(s) for "$(PROJECT_NAME):$(HOST_USER)" removed.' \
	|| echo 'Image(s) for "$(PROJECT_NAME):$(HOST_USER)" already removed.'
	rm -f $(DOCKERFILE)*
	rm -f shell
	rm -f gui
	rm -f statoshi
	rm -f stats.build.*
#######################
prune:
	docker system prune -af
#######################
doc:
	export HOST_USER=root
	export HOST_UID=0
	bash -c '$(pwd) make user=root'
	bash -c 'cat README > README.md'
	bash -c 'cat ./docker/Docker.md >> README.md'
	bash -c '$(pwd) make user=root  >> README.md'
#######################
package-all: clean build-all
	bash -c 'cat ~/GH_TOKEN.txt | docker login docker.pkg.github.com -u RandyMcMillan --password-stdin'
	bash -c 'docker tag $(PROJECT_NAME):$(HOST_USER) docker.pkg.github.com/bitcoincore-dev/stats.bitcoincore.dev/$(notdir $(PWD)):$(VERSION)'
	bash -c 'docker push docker.pkg.github.com/bitcoincore-dev/stats.bitcoincore.dev/$(notdir $(PWD)):$(VERSION)'
#######################
package-gui: clean build-gui
	bash -c 'cat ~/GH_TOKEN.txt | docker login docker.pkg.github.com -u RandyMcMillan --password-stdin'
	bash -c 'docker tag $(PROJECT_NAME):$(HOST_USER) docker.pkg.github.com/bitcoincore-dev/stats.bitcoincore.dev/$(notdir $(PWD)).gui:$(VERSION)'
	bash -c 'docker push docker.pkg.github.com/bitcoincore-dev/stats.bitcoincore.dev/$(notdir $(PWD)).gui:$(VERSION)'
#######################
package-slim: clean build-slim
	bash -c 'cat ~/GH_TOKEN.txt | docker login docker.pkg.github.com -u RandyMcMillan --password-stdin'
	bash -c 'docker tag $(PROJECT_NAME):$(HOST_USER) docker.pkg.github.com/bitcoincore-dev/stats.bitcoincore.dev/$(notdir $(PWD)).slim:$(VERSION)'
	bash -c 'docker push docker.pkg.github.com/bitcoincore-dev/stats.bitcoincore.dev/$(notdir $(PWD)).slim:$(VERSION)'
#######################
-include Makefile

