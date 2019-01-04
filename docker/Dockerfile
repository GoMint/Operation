FROM adoptopenjdk/openjdk11:alpine AS jlink

# Create minimal JVM
RUN ["jlink", "--compress=2", "--strip-debug", "--module-path", "/opt/java/openjdk/jmods", \
  "--add-modules", "java.base", "--add-modules", "java.xml", "--add-modules", "java.logging", \
  "--add-modules", "java.management", "--add-modules", "java.naming", "--add-modules", "java.desktop", \
  "--add-modules", "java.scripting", "--add-modules", "java.sql", "--add-modules", "jdk.unsupported", \
  "--add-modules", "jdk.crypto.ec", "--output", "/jlinked"]

FROM alpine

# Build arguments
ARG buildNumber
ENV gomintBuildNumber=$buildNumber

# Copy over minimal JRE
COPY --from=jlink /jlinked /opt/java/

# Set env first to reduce layers in the container image
ENV JAVA_HOME=/opt/java/ \
    PATH="/opt/java/bin:$PATH" \
    ALPINE_MIRROR="http://dl-cdn.alpinelinux.org/alpine"

# This is the line from AdoptOpenJDK:
RUN apk --update add --no-cache ca-certificates curl openssl binutils xz \
    && GOMINT_URL="http://ci.gomint.io/job/GoMint/job/master/${gomintBuildNumber}/artifact/gomint-server/target/GoMint.jar" \
    && GLIBC_VER="2.28-r0" \
    && ALPINE_GLIBC_REPO="https://github.com/sgerrand/alpine-pkg-glibc/releases/download" \
    && GCC_LIBS_URL="https://archive.archlinux.org/packages/g/gcc-libs/gcc-libs-8.2.1%2B20180831-1-x86_64.pkg.tar.xz" \
    && GCC_LIBS_SHA256=e4b39fb1f5957c5aab5c2ce0c46e03d30426f3b94b9992b009d417ff2d56af4d \
    && ZLIB_URL="https://archive.archlinux.org/packages/z/zlib/zlib-1%3A1.2.9-1-x86_64.pkg.tar.xz" \
    && ZLIB_SHA256=bb0959c08c1735de27abf01440a6f8a17c5c51e61c3b4c707e988c906d3b7f67 \
    && curl -Ls https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub -o /etc/apk/keys/sgerrand.rsa.pub \
    && curl -Ls ${ALPINE_GLIBC_REPO}/${GLIBC_VER}/glibc-${GLIBC_VER}.apk > /tmp/${GLIBC_VER}.apk \
    && apk add /tmp/${GLIBC_VER}.apk \
    && curl -Ls ${GCC_LIBS_URL} -o /tmp/gcc-libs.tar.xz \
    && echo "${GCC_LIBS_SHA256}  /tmp/gcc-libs.tar.xz" | sha256sum -c - \
    && mkdir /tmp/gcc \
    && tar -xf /tmp/gcc-libs.tar.xz -C /tmp/gcc \
    && mv /tmp/gcc/usr/lib/libgcc* /tmp/gcc/usr/lib/libstdc++* /usr/glibc-compat/lib \
    && strip /usr/glibc-compat/lib/libgcc_s.so.* /usr/glibc-compat/lib/libstdc++.so* \
    && curl -Ls ${ZLIB_URL} -o /tmp/libz.tar.xz \
    && echo "${ZLIB_SHA256}  /tmp/libz.tar.xz" | sha256sum -c - \
    && mkdir /tmp/libz \
    && tar -xf /tmp/libz.tar.xz -C /tmp/libz \
    && mv /tmp/libz/usr/lib/libz.so* /usr/glibc-compat/lib \
    && apk --update add --virtual build-dependencies --no-cache perl make gcc musl-dev linux-headers \
    && OPENSSL_URL="https://www.openssl.org/source/openssl-1.1.1a.tar.gz" \
    && OPENSSL_SHA256="fc20130f8b7cbd2fb918b2f14e2f429e109c31ddd0fb38fc5d71d9ffed3f9f41" \
    && curl -Ls ${OPENSSL_URL} -o /tmp/openssl.tar.gz \
    && echo "${OPENSSL_SHA256}  /tmp/openssl.tar.gz" | sha256sum -c - \
    && mkdir /tmp/openssl \
    && tar -xf /tmp/openssl.tar.gz -C /tmp/openssl \
    && cd /tmp/openssl/openssl-1.1.1a && chmod +x config \
    && ./config && make install_sw \
    && mkdir /gomint/ && mkdir /gomint/plugins/ && mkdir /gomint/internal/ \
    && curl -Ls "${GOMINT_URL}" -o /gomint/GoMint.jar \
    && apk del binutils curl xz build-dependencies \
    && rm -rf /tmp/${GLIBC_VER}.apk /tmp/gcc /tmp/gcc-libs.tar.xz /tmp/libz /tmp/libz.tar.xz /var/cache/apk/* \
    && cd /gomint && java -XX:+UseAppCDS -XX:DumpLoadedClassList=internal/classes.lst -jar GoMint.jar --exit-after-boot \
    && java -XX:+UseAppCDS -Xshare:dump -XX:SharedClassListFile=internal/classes.lst -XX:SharedArchiveFile=internal/app-cds.jsa --class-path GoMint.jar \
    && rm /gomint/internal/classes.lst && rm /tmp/* -rf

VOLUME /gomint/plugins
WORKDIR /gomint

CMD ["java", "--add-opens", "java.base/jdk.internal.loader=ALL-UNNAMED", "-XX:TieredCompilation", "-Xshare:on", "-XX:SharedArchiveFile=internal/app-cds.jsa", "-jar", "GoMint.jar"]