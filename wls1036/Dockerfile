# Dockerfile for weblogic server 10.3.6 + jdk1.6_45
# Based on ref: https://github.com/gibaholms/docker/blob/master/weblogic/10.3.6/Dockerfile

# IMPORTANT
# -----------------------------------------
# The resulting image of this Dockerfile DOES NOT contain a WLS Domain.
# For that, you must to create a domain on a new inherited image.
#
# REQUIRED FILES TO BUILD THIS IMAGE
# -----------------------------------------
# (1) wls1036_generic.jar (Oracle WebLogic Server 10.3.6 Generic Installer)#
# (2) jdk-6u45-linux-x64.bin (Oracle Java SE Development Kit 6 for Linux x64 RPM)
#
# HOW TO BUILD THIS IMAGE
# -----------------------------------------
# Put all downloaded files in the same directory as this Dockerfile
# Run: 
#      $ sudo docker build -t yourusername/weblogic:10.3.6 . 

FROM oraclelinux:6
MAINTAINER Yamil Medina 

# -------------------------------------
# PRE-Build Setup
# -------------------------------------
# Environment build and system vars
ENV WLS_JAR wls1036_generic.jar
ENV JAVA_BIN jdk-6u45-linux-x64.bin
ENV JVM_BASE /usr/lib/jvm
ENV JAVA_HOME $JVM_BASE/jdk1.6.0_45

# Setup required packages (unzip), filesystem, and oracle user
RUN mkdir /u01 && \
    chmod a+xr /u01 && \
    useradd -b /u01 -m -s /bin/bash oracle 

# Setup JAVA_HOME directory
RUN mkdir $JVM_BASE && \
	chmod a+xr $JVM_BASE

# Copy packages JDK and WLS files
COPY $JAVA_BIN $JVM_BASE/
COPY $WLS_JAR /u01/
COPY wls-silent.xml /u01/

# -------------------------------------
# Install and configure Oracle JDK
# -------------------------------------
WORKDIR $JVM_BASE 
RUN $JVM_BASE/$JAVA_BIN && \ 
    rm $JVM_BASE/$JAVA_BIN
WORKDIR /

ENV PATH $PATH:$JAVA_HOME/bin

# -------------------------------------
# Tunning
# -------------------------------------
# Change the open file limits in /etc/security/limits.conf
RUN sed -i '/.*EOF/d' /etc/security/limits.conf && \
    echo "* soft nofile 16384" >> /etc/security/limits.conf && \ 
    echo "* hard nofile 16384" >> /etc/security/limits.conf && \ 
    echo "# EOF"  >> /etc/security/limits.conf

# Change the kernel parameters that need changing.
RUN echo "net.core.rmem_max=4192608" > /u01/oracle/.sysctl.conf && \
    echo "net.core.wmem_max=4192608" >> /u01/oracle/.sysctl.conf && \ 
    sysctl -e -p /u01/oracle/.sysctl.conf

# Adjust file permissions, go to /u01 as user 'oracle' to proceed with WLS installation
RUN chown oracle:oracle -R /u01
WORKDIR /u01
USER oracle

# -------------------------------------
# Installation of WebLogic Server 10.3.6
# -------------------------------------
RUN java -jar $WLS_JAR -mode=silent -silent_xml=/u01/wls-silent.xml && \ 
	rm $WLS_JAR /u01/wls-silent.xml 

WORKDIR /u01/oracle/

ENV PATH $PATH:/u01/oracle/weblogic/oracle_common/common/bin

# Define default command to start bash. 
CMD ["bash"] 
