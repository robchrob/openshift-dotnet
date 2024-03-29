FROM dotnet/dotnet-22-runtime-centos7
# This image provides a .NET Core 2.2 environment you can use to run your .NET
# applications.

ENV PATH=/opt/app-root/src/.local/bin:/opt/app-root/src/bin:/opt/app-root/node_modules/.bin:${PATH} \
    STI_SCRIPTS_PATH=/usr/libexec/s2i

LABEL io.k8s.description="Platform for building and running .NET Core 2.2 applications" \
      io.openshift.tags="builder,.net,dotnet,dotnetcore,rh-dotnet22"

# Labels consumed by Red Hat build service
LABEL name="dotnet/dotnet-22-centos7" \
      com.redhat.component="rh-dotnet22-container" \
      version="2.2" \
      release="1" \
      architecture="x86_64"

# Labels consumed by Eclipse JBoss OpenShift plugin
LABEL com.redhat.dev-mode="DEV_MODE:false" \
      com.redhat.deployments-dir="/opt/app-root/src"

# Switch to root for package installs
USER 0

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH.
COPY ./s2i/bin/ /usr/libexec/s2i

RUN yum install -y centos-release-dotnet centos-release-scl-rh && \
    INSTALL_PKGS="rh-nodejs8-npm rh-nodejs8-nodejs-nodemon rh-dotnet22-dotnet-sdk-2.2 rsync" && \
    yum install -y --setopt=tsflags=nodocs $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum clean all -y && \
# yum cache files may still exist (and quite large in size)
    rm -rf /var/cache/yum/*

# Directory with the sources is set as the working directory.
RUN mkdir /opt/app-root/src
WORKDIR /opt/app-root/src

# Since $HOME is set to /opt/app-root, the yum install may have created config
# directories (such as ~/.pki/nssdb) there. These will be owned by root and can
# cause actions that work on all of /opt/app-root to fail. So we need to fix
# the permissions on those too.
RUN chown -R 1001:0 /opt/app-root && fix-permissions /opt/app-root

ENV ENABLED_COLLECTIONS="$ENABLED_COLLECTIONS rh-nodejs8"

# For backwards compatibility, s2i builds default to the oldest sdk in the image.
# We keep the patch at 'x00', the latest patch version is automatically picked up.
ENV DOTNET_SDK_BASE_VERSION=2.2.100 \
# Needed for the `dotnet watch` to detect changes in a container.
    DOTNET_USE_POLLING_FILE_WATCHER=true \
# Make all sdks aware of the latest ASP.NET Core version
    LatestPatchVersionForAspNetCoreApp2_2=2.2.2 \
    LatestPatchVersionForAspNetCoreAll2_2=2.2.2

# Run container by default as user with id 1001 (default)
USER 1001

# Set the default CMD to print the usage of the language image.
CMD /usr/libexec/s2i/usage
