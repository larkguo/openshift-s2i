
# CPP

## Install s2i tool

### See [CPP/install-s2i-tool.log](https://github.com/larkguo/openshift-s2i/blob/master/CPP/install-s2i-tool.log) for details: 
    
    $ wget https://github.com/openshift/source-to-image/releases/download/v1.1.9a/source-to-image-v1.1.9a-40ad911d-linux-amd64.tar.gz
    
    $ tar -xvzf source-to-image-v1.1.9a-40ad911d-linux-amd64.tar.gz -C /usr/bin
    
## Creating your own base image

### See [CPP/build-cpp-image.log](https://github.com/larkguo/openshift-s2i/blob/master/CPP/build-cpp-image.log) for details: 
    
    $ s2i create openshift/base-centos7 s2i-cpp
    
    $ cd s2i-cpp/
    
    $ cat Dockerfile 
    # openshift/base-centos7
    FROM openshift/base-centos7

    # TODO: Put the maintainer name in the image metadata
    # LABEL maintainer="Your Name <your@email.com>"

    # TODO: Rename the builder environment variable to inform users about application you provide them
    # ENV BUILDER_VERSION 1.0

    # TODO: Set labels used in OpenShift to describe the builder image
    #LABEL io.k8s.description="Platform for building xyz" \
    #      io.k8s.display-name="builder x.y.z" \
    #      io.openshift.expose-services="8080:http" \
    #      io.openshift.tags="builder,x.y.z,etc."

    # Install required packages here:
    RUN yum install -y epel-release
    RUN yum install -y net-tools
    RUN yum clean all -y

    # TODO (optional): Copy the builder files into /opt/app-root
    # COPY ./<builder_folder>/ /opt/app-root/

    # TODO: Copy the S2I scripts to /usr/libexec/s2i, since openshift/base-centos7 image
    # sets io.openshift.s2i.scripts-url label that way, or update that label
    COPY ./s2i/bin/ /usr/libexec/s2i

    # Copy the configuration file
    #COPY ./etc/ /opt/app-root/etc

    # TODO: Drop the root user and make the content of /opt/app-root owned by user 1001
    # RUN chown -R 1001:1001 /opt/app-root

    # This default user is created in the openshift/base-centos7 image
    USER 1001

    # TODO: Set the default port for applications built using this image
    EXPOSE 8000

    # Set the default CMD for the image
    CMD ["/usr/libexec/s2i/usage"]

    $ make
    $ docker images
    REPOSITORY                                                      TAG                 IMAGE ID            CREATED             SIZE
    s2i-cpp                                                         latest              11d7e16c642a        24 minutes ago      653 MB
    docker.io/openshift/base-centos7                                latest              4842f0bd3d61        15 months ago       383 MB
    
    $ cat s2i/bin/assemble 
    #!/bin/bash -e
    #
    # S2I assemble script for the 'openshift/base-centos7' image.
    # The 'assemble' script builds your application source so that it is ready to run.
    #
    # For more information refer to the documentation:
    #       https://github.com/openshift/source-to-image/blob/master/docs/builder_image.md
    #

    # If the 'openshift/base-centos7' assemble script is executed with the '-h' flag, print the usage.
    if [[ "$1" == "-h" ]]; then
            exec /usr/libexec/s2i/usage
    fi

    # Restore artifacts from the previous build (if they exist).
    #
    if [ "$(ls /tmp/artifacts/ 2>/dev/null)" ]; then
      echo "---> Restoring build artifacts..."
      mv /tmp/artifacts/. ./
    fi

    echo "---> Installing application source..."
    cp -Rf /tmp/src/. ./

    echo "---> Building application from source..."
    # Add build steps for your application, eg npm install, bundle install, pip install, etc.
    gcc socket/chat.c -o chat

    $ cat s2i/bin/run 
    #!/bin/bash -e
    #
    # S2I run script for the 'openshift/base-centos7' image.
    # The run script executes the server that runs your application.
    #
    # For more information see the documentation:
    #       https://github.com/openshift/source-to-image/blob/master/docs/builder_image.md
    #
    exec ./chat
    
    $ cat s2i/bin/usage 
    #!/bin/bash -e
    cat <<EOF
    This is the openshift/base-centos7 S2I image:
    To use it, install S2I: https://github.com/openshift/source-to-image

    Sample invocation:
    s2i build https://github.com/larkguo/tests.git s2i-cpp s2i-test 

    You can then run the resulting image via:
    docker run  -itd -p 8000:8000  s2i-test
    EOF

    $ s2i build https://github.com/larkguo/tests.git s2i-cpp s2i-test 
    
    $ docker images
    REPOSITORY                                                      TAG                 IMAGE ID            CREATED             SIZE
    s2i-test                                                        latest              0d710293ed22        23 minutes ago      663 MB
    s2i-cpp                                                         latest              11d7e16c642a        24 minutes ago      653 MB
    docker.io/openshift/base-centos7                                latest              4842f0bd3d61        15 months ago       383 MB
            
    $ docker run  -itd -p 8000:8000  s2i-test
    
    
