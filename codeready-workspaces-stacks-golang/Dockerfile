# Copyright (c) 2018-2019 Red Hat, Inc.
# This program and the accompanying materials are made
# available under the terms of the Eclipse Public License 2.0
# which is available at https://www.eclipse.org/legal/epl-2.0/
#
# SPDX-License-Identifier: EPL-2.0
#
# Contributors:
#   Red Hat, Inc. - initial API and implementation
#

# https://access.redhat.com/containers/?tab=tags#/registry.access.redhat.com/rhel8/go-toolset
FROM rhel8/go-toolset:1.14.7-15

ENV SUMMARY="Red Hat CodeReady Workspaces - Golang Stack container" \
    DESCRIPTION="Red Hat CodeReady Workspaces - Golang Stack container" \
    PRODNAME="codeready-workspaces" \
    COMPNAME="stacks-golang-rhel8" \
    HOME=/home/jboss \
    PATH=/usr/bin:/usr/bin:/home/jboss/go/bin:$PATH \
    GOPATH=/projects/.che/gopath

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

USER root

# built in Brew, use get-sources-jenkins.sh to pull latest
COPY . /tmp/assets/

# NOTE: uncomment for local build. Must also set full registry path in FROM to registry.redhat.io
# COPY content_sets.repo /etc/yum.repos.d/

# For OpenShift Client 4.4 (oc): rhocp-4.6-for-rhel-8-x86_64-rpms
# For OCP Tools (helm): ocp-tools-4.4-for-rhel-8-x86_64-rpms
# must hard code a version because otherwise we fail with
## Failed component comparison for components: openshift-clients
# http://pulp.dist.prod.ext.phx2.redhat.com/content/dist/layered/rhel8/x86_64/rhocp/4.6/os/Packages/o/openshift-clients-4.6.0-202010081244.p0.git.3794.4743d24.el8.x86_64.rpm
# http://pulp.dist.prod.ext.phx2.redhat.com/content/dist/layered/rhel8/s390x/rhocp/4.6/os/Packages/o/openshift-clients-4.6.0-202010081244.p0.git.3794.4743d24.el8.s390x.rpm
# http://pulp.dist.prod.ext.phx2.redhat.com/content/dist/layered/rhel8/ppc64le/rhocp/4.6/os/Packages/o/openshift-clients-4.6.0-202010081244.p0.git.3794.4743d24.el8.ppc64le.rpm
RUN useradd -u 1000 -G wheel,root -d /home/jboss --shell /bin/bash -m jboss && \
    dnf remove -y nodejs && \
    yum install -y \
    # next line are c/c++ compiler stuff which we apparently need for new golangci-lint
    make cmake gcc gcc-c++ glibc-devel zlib-devel libstdc++ libstdc++-devel \
    sudo git wget curl nss_wrapper bzip2 npm openshift-clients-4.6.0-202010081244.p0.git.3794.4743d24.el8 && \
    yum update -y cryptsetup-libs gnutls systemd systemd-libs systemd-pam subscription-manager python3-syspurpose dnf-plugin-subscription-manager python3-subscription-manager-rhsm \
        subscription-manager-rhsm-certificates kernel-headers pango libnghttp2 systemd go-toolset-rhel8 gnutls && \
    if [[ $(uname -m) == "ppc64le" ]]; then yum update -y freetype freetype-devel; fi && \
    yum clean all && rm -rf /var/cache/yum && \
    mkdir -p /projects/.che/gopath /.cache && \
    cat /etc/passwd | \
    sed s#jboss:x.*#jboss:x:\${USER_ID}:\${GROUP_ID}::\${HOME}:/bin/bash#g \
    > /home/jboss/passwd.template && \
    cat /etc/group | \
    sed s#root:x:0:#root:x:0:0,\${USER_ID}:#g \
    > /home/jboss/group.template && \
    echo "jboss	ALL=(ALL)	NOPASSWD: ALL" >> /etc/sudoers && \
    echo -e "if [ ! -d "/projects/.che/gopath/src" ]; then\necho "Copying GO LS Deps"\nmkdir -p /projects/.che/gopath\n cp -R /tmp/go/* /projects/.che/gopath\nfi" \
    > ${HOME}/gopath.sh && \
    chmod +x ${HOME}/gopath.sh

ADD entrypoint.sh ${HOME}/entrypoint.sh
RUN mkdir -p ${HOME}/go && \
    tar -xvf /tmp/assets/codeready-workspaces-stacks-language-servers-dependencies-golang-$(uname -m).tar.gz -C /tmp && \
    rm -fr tmp/go /tmp/assets && \
    for f in "/home/jboss" "/etc/passwd" "/etc/group" "/projects" "/.cache" "/usr/share/gocode" "/opt/app-root/src/"; do \
        chgrp -R 0 ${f} && \
        chmod -R g+rwX ${f}; \
    done && \
    echo "Installed Packages" && rpm -qa | sort -V && echo "End Of Installed Packages"

USER jboss
ENTRYPOINT ["/home/jboss/entrypoint.sh"]
WORKDIR /projects
CMD ${HOME}/gopath.sh & tail -f /dev/null


# insert generated LABELs below this line
LABEL \
      git.commit.redhat-developer__codeready-workspaces-deprecated="https://github.com/redhat-developer/codeready-workspaces-deprecated/commit/5e50" \
      jenkins.build.url="https://codeready-workspaces-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/CRW_CI/view/Pipelines/job/crw-deprecated_2.5/7/" \
      jenkins.artifact.url="https://codeready-workspaces-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/CRW_CI/view/Pipelines/job/crw-deprecated_2.5/7/artifact/**/codeready-workspaces-stacks-language-servers-dependencies-golang-x86_64.tar.gz" \
      jenkins.artifact.url="https://codeready-workspaces-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/CRW_CI/view/Pipelines/job/crw-deprecated_2.5/7/artifact/**/codeready-workspaces-stacks-language-servers-dependencies-golang-s390x.tar.gz" \
      jenkins.artifact.url="https://codeready-workspaces-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/CRW_CI/view/Pipelines/job/crw-deprecated_2.5/7/artifact/**/codeready-workspaces-stacks-language-servers-dependencies-golang-ppc64le.tar.gz" \
      jenkins.build.number="7"
