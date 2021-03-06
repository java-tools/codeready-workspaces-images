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

# https://access.redhat.com/containers/?tab=tags#/registry.access.redhat.com/rhel8/llvm-toolset
FROM rhel8/llvm-toolset:9.0.1-49

ENV SUMMARY="Red Hat CodeReady Workspaces - C/CPP Stack container" \
    DESCRIPTION="Red Hat CodeReady Workspaces - C/CPP Stack container" \
    PRODNAME="codeready-workspaces" \
    COMPNAME="stacks-cpp-rhel8" \
    HOME=/home/jboss \
    PATH="/usr/bin:/usr/sbin${PATH:+:${PATH}}" \
    LD_LIBRARY_PATH="/usr/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}" \
    MANPATH="/usr/share/man:${MANPATH:-}" \
    PKG_CONFIG_PATH="/usr/lib64/pkgconfig${PKG_CONFIG_PATH:+:${PKG_CONFIG_PATH}}" \
    PYTHONPATH="/usr/lib/python3.6/site-packages${PYTHONPATH:+:${PYTHONPATH}}"

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
    yum install -y llvm-toolset clang clang-libs clang-tools-extra git-clang-format gdb wget bzip2 sudo make cmake gcc gcc-c++ \
    openshift-clients-4.6.0-202010081244.p0.git.3794.4743d24.el8 && \
    # to see what requires kernel-headers, use in line above: yum install --exclude=kernel*
    # or query: rpm -q --whatrequires kernel-headers && rpm -q --whatrequires glibc-headers && rpm -q --whatrequires glibc-devel && rpm -q --whatrequires gcc
    yum update -y freetype freetype-devel cryptsetup-libs gnutls systemd systemd-libs systemd-pam subscription-manager python3-syspurpose dnf-plugin-subscription-manager python3-subscription-manager-rhsm \
        subscription-manager-rhsm-certificates kernel-headers pango libnghttp2 systemd git git-core perl-Git git-core-doc && \
    yum clean all && rm -rf /var/cache/yum && \
    cat /etc/passwd | \
    sed s#jboss:x.*#jboss:x:\${USER_ID}:\${GROUP_ID}::\${HOME}:/bin/bash#g \
    > /home/jboss/passwd.template && \
    cat /etc/group | \
    sed s#root:x:0:#root:x:0:0,\${USER_ID}:#g \
    > /home/jboss/group.template && \
    echo "jboss	ALL=(ALL)	NOPASSWD: ALL" >> /etc/sudoers && \
    mkdir -p /projects ${HOME}/che/ls-clangd && \
    echo "clangd -pretty" > ${HOME}/che/ls-clangd/launch.sh && \
    chmod +x ${HOME}/che/ls-clangd/launch.sh && \
    for f in "/home/jboss" "/etc/passwd" "/etc/group" "/projects"; do \
        chgrp -R 0 ${f} && \
        chmod -R g+rwX ${f}; \
    done && \
    echo "Installed Packages" && rpm -qa | sort -V && echo "End Of Installed Packages"
USER jboss
ADD entrypoint.sh ${HOME}/entrypoint.sh

ENTRYPOINT ["/home/jboss/entrypoint.sh"]
WORKDIR /projects
CMD tail -f /dev/null
