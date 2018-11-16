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
ENV REDIS_DOWNLOAD_URL http://download.redis.io/releases/redis-3.2.9.tar.gz
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

	make -C /usr/src/redis; \
	make -C /usr/src/redis install; \
	\
	rm -r /usr/src/redis; \
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
