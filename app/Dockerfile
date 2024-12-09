FROM ubuntu:22.04
ARG SQUID_VERSION="SQUID_6_11"

RUN apt-get update && apt-get install -y \
    libssl-dev libcppunit-dev libexpat1 libnetfilter-conntrack3 git libxml2 dnsutils vim net-tools \
    build-essential libssl-dev libcppunit-dev libtool libexpat1-dev libnetfilter-conntrack-dev libxml2-dev \
    && rm -rf /var/lib/apt/lists/*


RUN git clone https://github.com/squid-cache/squid.git && cd squid && git checkout ${SQUID_VERSION}
RUN cd squid && ./bootstrap.sh && \
    ./configure --prefix=/usr/local/squid --enable-ssl-crtd --with-openssl --enable-ssl

RUN cd squid && make clean && make -j$(nproc) && make install

# Verify ssl_crtd exists
RUN ls /usr/local/squid/libexec | grep 'security_file_certgen' 

#TODO: Verify squid version
RUN /usr/local/squid/sbin/squid -v

# Create necessary directories
RUN mkdir -p /var/cache/squid /var/log/squid /var/spool/squid && \
    chown -R nobody:nogroup /var/cache/squid /usr/local/squid/var/logs /var/log/squid /var/spool/squid
RUN chmod -R 755 /usr/local/squid/var/logs/


# Initialize the SSL DB
RUN mkdir -p /usr/local/squid/var/lib
RUN chown -R nobody:nogroup /usr/local/squid/var/lib
RUN chmod -R 750 /usr/local/squid/var/lib
RUN /usr/local/squid/libexec/security_file_certgen -c -s /var/cache/squid/ssl_db -M 4MB

# Set Squid configuration file
COPY squid.conf /usr/local/squid/etc/squid.conf

# TODO: Add SSL Certs signed by CA

# Expose the HTTP proxy port
EXPOSE 3128

# Command to start Squid
CMD ["/usr/local/squid/sbin/squid", "-N", "-X", "-d", "1"]
