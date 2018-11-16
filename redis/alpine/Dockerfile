# Copyright 2017 Ismail KABOUBI
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM alpine:3.8

# Add kubectl to enable sentinels to change labels during promotion
ADD https://storage.googleapis.com/kubernetes-release/release/v1.11.2/bin/linux/amd64/kubectl /usr/local/bin/kubectl
RUN chmod +x /usr/local/bin/kubectl
#old install redis
#RUN apk add --no-cache bash sed curl redis=3.2.9 ca-certificates
#RUN rm -rf /var/cache/apk/*

#new install redis-----------------------------
RUN addgroup -S redis && adduser -S -G redis redis

RUN apk add --no-cache \
# grab su-exec for easy step-down from root
		'su-exec>=0.2' \
# add tzdata for https://github.com/docker-library/redis/issues/138
		tzdata

ENV REDIS_VERSION 3.2.9
ENV REDIS_DOWNLOAD_URL http://download.redis.io/releases/redis-3.2.9.tar.gz

# for redis-sentinel see: http://redis.io/topics/sentinel
RUN set -ex; \
	\
	apk add --no-cache --virtual .build-deps \
		coreutils \
		gcc \
		jemalloc-dev \
		linux-headers \
		make \
		musl-dev \
	; \
	\
	wget -O redis.tar.gz "$REDIS_DOWNLOAD_URL"; \
	mkdir -p /usr/src/redis; \
	tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1; \
	rm redis.tar.gz; \
	\
# disable Redis protected mode [1] as it is unnecessary in context of Docker
# (ports are not automatically exposed when running inside Docker, but rather explicitly by specifying -p / -P)
# [1]: https://github.com/antirez/redis/commit/edd4d555df57dc84265fdfb4ef59a4678832f6da
	grep -q '^#define CONFIG_DEFAULT_PROTECTED_MODE 1$' /usr/src/redis/src/server.h; \
	sed -ri 's!^(#define CONFIG_DEFAULT_PROTECTED_MODE) 1$!\1 0!' /usr/src/redis/src/server.h; \
	grep -q '^#define CONFIG_DEFAULT_PROTECTED_MODE 0$' /usr/src/redis/src/server.h; \
# for future reference, we modify this directly in the source instead of just supplying a default configuration flag because apparently "if you specify any argument to redis-server, [it assumes] you are going to specify everything"
# see also https://github.com/docker-library/redis/issues/4#issuecomment-50780840
# (more exactly, this makes sure the default behavior of "save on SIGTERM" stays functional by default)
	\
	make -C /usr/src/redis -j "$(nproc)"; \
	make -C /usr/src/redis install; \
	\
	rm -r /usr/src/redis; \
	\
	runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
			| tr ',' '\n' \
			| sort -u \
			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)"; \
	apk add --virtual .redis-rundeps $runDeps; \
	apk del .build-deps; \
	\
	redis-server --version
#new install redis end----------------------------------------------

COPY redis-master.conf /etc/redis/master.conf
COPY redis-slave.conf /etc/redis/slave.conf
COPY redis-sentinel.conf /etc/redis/sentinel.conf
COPY label-updater.sh /usr/local/bin/label-updater.sh
COPY redis-launcher.sh /usr/local/bin/redis-launcher.sh
COPY promote.sh /usr/local/bin/promote.sh

CMD [ "touch /var/log/redis/runner.txt; chmod +w /var/log/redis/runner.txt; chmod +x /usr/local/bin/redis-launcher.sh" ]
CMD [ "/usr/local/bin/redis-launcher.sh | tee /var/log/redis/runner.txt" ]

ENTRYPOINT [ "bash", "-c" ]
