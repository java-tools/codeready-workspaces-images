# Copyright (c) 2020 Red Hat, Inc.
# This program and the accompanying materials are made
# available under the terms of the Eclipse Public License 2.0
# which is available at https://www.eclipse.org/legal/epl-2.0/
#
# SPDX-License-Identifier: EPL-2.0
#
# Contributors:
#   Red Hat, Inc. - initial API and implementation
#

# https://access.redhat.com/containers/?tab=tags#/registry.access.redhat.com/ubi8-minimal
FROM ubi8-minimal:8.2-349

USER root

ENV HOME=/home/jboss \
    NODEJS_VERSION=12 \
    MAVEN_VERSION="3.6.3" \
    JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk \
    PATH=$HOME/node_modules/.bin/:$HOME/.npm-global/bin/:/opt/app-root/src/.npm-global/bin/:/usr/lib/jvm/java-1.8.0-openjdk:/opt/apache-maven/bin:/usr/bin:$PATH \
    MANPATH="/usr/share/man:${MANPATH}" \
    JAVACONFDIRS="/etc/java${JAVACONFDIRS:+:}${JAVACONFDIRS:-}" \
    XDG_CONFIG_DIRS="/etc/xdg:${XDG_CONFIG_DIRS:-/etc/xdg}" \
    XDG_DATA_DIRS="/usr/share:${XDG_DATA_DIRS:-/usr/local/share:/usr/share}" \
    M2_HOME="/opt/apache-maven" 

# built in Brew, use get-sources-jenkins.sh to pull latest
COPY . /tmp/assets/

# NOTE: uncomment for local build. Must also set full registry path in FROM to registry.redhat.io
# COPY content_set*.repo /etc/yum.repos.d/

# For OpenShift Client 4.4 (oc): rhocp-4.6-for-rhel-8-x86_64-rpms
# For OCP Tools (helm): ocp-tools-4.4-for-rhel-8-x86_64-rpms
# must hard code a version because otherwise we fail with
## Failed component comparison for components: openshift-clients
# http://pulp.dist.prod.ext.phx2.redhat.com/content/dist/layered/rhel8/x86_64/rhocp/4.6/os/Packages/o/openshift-clients-4.6.0-202010081244.p0.git.3794.4743d24.el8.x86_64.rpm
# http://pulp.dist.prod.ext.phx2.redhat.com/content/dist/layered/rhel8/s390x/rhocp/4.6/os/Packages/o/openshift-clients-4.6.0-202010081244.p0.git.3794.4743d24.el8.s390x.rpm
# http://pulp.dist.prod.ext.phx2.redhat.com/content/dist/layered/rhel8/ppc64le/rhocp/4.6/os/Packages/o/openshift-clients-4.6.0-202010081244.p0.git.3794.4743d24.el8.ppc64le.rpm
RUN microdnf install -y yum bash tar gzip unzip which shadow-utils findutils wget curl openshift-clients-4.6.0-202010081244.p0.git.3794.4743d24.el8 \
    java-1.8.0-openjdk \
    java-1.8.0-openjdk-devel \
    java-1.8.0-openjdk-headless \
    # CRW-919 java8/mvn + python/pip + node/npm: combine forces and make one all-in-one useful image
    python36 python3-six python3-pip platform-python-pip \
    sudo git procps-ng bzip2 && \
    # BEGIN copy from https://catalog.redhat.com/software/containers/ubi8/nodejs-12/5d3fff015a13461f5fb8635a?container-tabs=dockerfile
        yum -y module reset nodejs && yum -y module enable nodejs:$NODEJS_VERSION && \
        INSTALL_PKGS="nodejs npm nodejs-nodemon nss_wrapper" && \
        ln -s /usr/lib/node_modules/nodemon/bin/nodemon.js /usr/bin/nodemon && \
        yum remove -y $INSTALL_PKGS && \
        yum install -y --setopt=tsflags=nodocs $INSTALL_PKGS && \
        rpm -V $INSTALL_PKGS && \
        yum -y clean all --enablerepo='*' && \
    # END copy from https://catalog.redhat.com/software/containers/ubi8/nodejs-12/5d3fff015a13461f5fb8635a?container-tabs=dockerfile
    # CVE updates
    microdnf update -y freetype zlib gnutls systemd-libs systemd pango libnghttp2 sqlite libarchive && \
    microdnf clean all && rm -rf /var/cache/yum && \
    # TODO: why do we need this jboss user?
    useradd -u 1000 -G wheel,root -d ${HOME} --shell /bin/bash -m jboss && \
    mkdir -p ${HOME}/che /projects && \
    for f in "${HOME}" "/etc/passwd" "/etc/group" "/projects"; do \
        chgrp -R 0 ${f} && \
        chmod -R g+rwX ${f}; \
    done && \
    # Generate passwd.template \
    cat /etc/passwd | \
    sed s#jboss:x.*#jboss:x:\${USER_ID}:\${GROUP_ID}::\${HOME}:/bin/bash#g \
    > ${HOME}/passwd.template && \
    # Generate group.template \
    cat /etc/group | \
    sed s#root:x:0:#root:x:0:0,\${USER_ID}:#g \
    > ${HOME}/group.template && \
    echo "jboss	ALL=(ALL)	NOPASSWD: ALL" >> /etc/sudoers && \
    \
    # maven stuff \
    mkdir -p /opt; \
    tar xzf /tmp/assets/apache-maven-${MAVEN_VERSION}-bin.tar.gz -C /opt && \
    mv /opt/apache-maven-${MAVEN_VERSION} /opt/apache-maven && \
    # fix permissions in bin/* files \
    for d in $(find /opt/apache-maven -name bin -type d); do echo $d; chmod +x $d/*; done && \
    \
    # node stuff \
    mkdir -p ${HOME}/lang-server /opt/app-root/src/.npm-global/bin && \
    ln -s /usr/bin/node /usr/bin/nodejs && \
    if [[ -f /tmp/assets/codeready-workspaces-stacks-language-servers-dependencies-node10-$(uname -m).tar.gz ]]; then \
        tar xzf /tmp/assets/codeready-workspaces-stacks-language-servers-dependencies-node10-$(uname -m).tar.gz -C ${HOME}/lang-server/ && \
        ln -s ${HOME}/lang-server/node_modules/typescript-language-server/lib/cli.js /opt/app-root/src/.npm-global/bin/typescript-language-server && \
        ln -s ${HOME}/lang-server/node_modules/nodemon/bin/nodemon.js /opt/app-root/src/.npm-global/bin/nodemon && \
        ln -s ${HOME}/lang-server/node_modules/typescript/bin/tsc /opt/app-root/src/.npm-global/bin/tsc && \
        ln -s ${HOME}/lang-server/node_modules/typescript/bin/tsserver /opt/app-root/src/.npm-global/bin/tsserver && \
        for f in "/opt/app-root/src/.npm-global"; do chgrp -R 0 ${f}; chmod -R g+rwX ${f}; done; \
    else \
        echo "[WARNING] Node lang server dependency tarball not found. Node support may be more limited on $(uname -m)"; \
    fi
RUN \
    # python stuff
    ln -s /usr/bin/python3.6 /usr/bin/python && \
    ln -s /usr/bin/pip3.6 /usr/bin/pip && \
    if [[ -f /tmp/assets/codeready-workspaces-stacks-language-servers-dependencies-python-$(uname -m).tar.gz ]]; then \
        mkdir -p /tmp/py-unpack /opt/app-root/bin /opt/app-root/lib /opt/app-root/lib64 && \
        tar -xf /tmp/assets/codeready-workspaces-stacks-language-servers-dependencies-python-$(uname -m).tar.gz -C /tmp/py-unpack && \
        cp -R /tmp/py-unpack/bin/* /opt/app-root/bin && \
        cp -R /tmp/py-unpack/lib/* /opt/app-root/lib && \
        cp -R /tmp/py-unpack/lib64/* /opt/app-root/lib64 && \
        for f in /opt/app-root/; do chgrp -R 0 ${f}; chmod -R g+rwX ${f}; done; \
    else \
        echo "[WARNING] Python lang server dependency tarball not found. Python support may be more limited on $(uname -m)"; \
    fi
RUN \
    # cleanup and summaries
    rm -fr /tmp/assets/ && \
    echo "Installed Packages" && rpm -qa | sort -V && echo "End Of Installed Packages" && \
    echo "========" && \
    java -version && \
    mvn -version && \
    echo "========" && \
    echo -n "Node "; node --version && \
    echo -n "npm ";  npm --version && \
    echo "========" && \
    python -V && \
    pip -V && \
    echo "========"

ADD entrypoint.sh ${HOME}/
RUN chmod +x ${HOME}/*.sh

USER jboss
ENTRYPOINT ["/home/jboss/entrypoint.sh"]
WORKDIR /projects
CMD tail -f /dev/null

# append Brew metadata here
ENV SUMMARY="Red Hat CodeReady Workspaces - Java 8 + Node 12 + Python 3 plugin container" \
    DESCRIPTION="Red Hat CodeReady Workspaces - Java 8 + Node 12 + Python 3 plugin container" \
    PRODNAME="codeready-workspaces" \
    COMPNAME="plugin-java8-rhel8"

LABEL summary="$SUMMARY" \
      description="$DESCRIPTION" \
      io.k8s.description="$DESCRIPTION" \
      io.k8s.display-name="$DESCRIPTION" \
      io.openshift.tags="$PRODNAME,$COMPNAME" \
      com.redhat.component="$PRODNAME-$COMPNAME-container" \
      name="$PRODNAME/$COMPNAME" \
      version="2.5" \
      license="EPLv2" \
      maintainer="Nick Boldt <nboldt@redhat.com>" \
      io.openshift.expose-services="" \
      usage=""

# insert generated LABELs below this line
LABEL \
	  git.commit.redhat-developer__codeready-workspaces-deprecated="https://github.com/redhat-developer/codeready-workspaces-deprecated/commit/5e50" \
	  jenkins.build.url="https://codeready-workspaces-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/CRW_CI/view/Pipelines/job/crw-deprecated_2.5/7/" \
	  jenkins.artifact.url="https://codeready-workspaces-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/CRW_CI/view/Pipelines/job/crw-deprecated_2.5/7/artifact/**/codeready-workspaces-stacks-language-servers-dependencies-node10-x86_64.tar.gz" \
	  jenkins.artifact.url="https://codeready-workspaces-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/CRW_CI/view/Pipelines/job/crw-deprecated_2.5/7/artifact/**/codeready-workspaces-stacks-language-servers-dependencies-python-x86_64.tar.gz" \
	  jenkins.build.number="7"
