FROM ubuntu:14.04

MAINTAINER Omar Al-Ithawi <i@omardo.com>

# The latest from edx/master that worked with me
# change it to the commit you'd like
ENV EDX_VERSION 7a858b6e09afd96e68939e4074825d1459144a2e

# Use: git references to speed up clone
ENV EDX_REPO https://github.com/edx/edx-platform.git

RUN mkdir -p /edx/app/edxapp/
WORKDIR /edx/app/edxapp/

RUN apt-get update -y  \
    && apt-get install -y  \
            git  \
            python-pip  \
            ruby1.9.3  \
            software-properties-common

RUN git clone $EDX_REPO
WORKDIR /edx/app/edxapp/edx-platform
RUN git checkout $EDX_VERSION

RUN gem install bundler

# TODO: Add this using the apt-repos.txt file
RUN apt-add-repository ppa:chris-lea/node.js
RUN apt-get update -y

# Allows ubuntu apt cache
# RUN rm /etc/apt/apt.conf.d/no-cache
ADD apt-packages.gitpatch /tmp/
RUN git apply /tmp/apt-packages.gitpatch
RUN cat requirements/system/ubuntu/apt-packages.txt | xargs apt-get install -y


# Keep `pre.txt` in its own step to support options in base and other stuff
RUN pip install --exists-action w -r requirements/edx/pre.txt

# TODO: Use the following to reduce layers
    # for pckg in $REQUIREMENT_FILES; \
    #   do pip install -q --disable-pip-version-check --exists-action w -r requirements/edx/$pckg.txt; \
    # done
RUN pip install -q --disable-pip-version-check --exists-action w -r requirements/edx/github.txt
RUN pip install -q --disable-pip-version-check --exists-action w -r requirements/edx/local.txt
RUN pip install -q --disable-pip-version-check --exists-action w -r requirements/edx/base.txt
RUN pip install -q --disable-pip-version-check --exists-action w -r requirements/edx/post.txt
RUN pip install -q --disable-pip-version-check --exists-action w -r requirements/edx/paver.txt

RUN bundle install

# Fix hardcoded values
RUN mkdir -p /edx/app/edxapp/log \
    && mkdir -p /edx/var/edxapp/data \
    && mkdir -p /edx/app/edxapp/uploads


# TODO: Remove this, as we don't need them anymore
ADD env/*.json /edx/app/edxapp/

# TODO: Remove those, as we don't need them anymore
ADD lms_docker_env.py lms/envs/docker.py
ADD cms_docker_env.py cms/envs/docker.py

# TODO: Remove those, as we don't need them anymore
ADD docker_common.py lms/envs/
ADD docker_common.py cms/envs/

RUN paver install_prereqs


# TODO: Make a link of it, instead of ruining the docker layers by an `$ mv`
WORKDIR /edx/app/edxapp
RUN mv edx-platform edx-platform-buildcode
RUN ln -sd edx-platform-devcode edx-platform

RUN bash -c 'rm {lms,cms}.{env,auth}.json'
COPY envs/ buildenvs/
RUN bash -c 'ln -s devenvs/{lms,cms}.{env,auth}.json .'

COPY copy_code.sh copy_code.sh
RUN chmod +x copy_code.sh