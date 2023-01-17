# syntax=docker/dockerfile:experimental
#
# Copyright 2018-2022 Elyra Authors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

# Ubuntu 20.04.2 LTS (Focal Fossa)
# Repository: https://hub.docker.com/r/jupyterhub/k8s-singleuser-sample/tags
FROM jupyterhub/k8s-singleuser-sample:1.2.0

ARG TAG="dev"
ARG ELYRA_VERSION
ARG ELYRA_PACKAGE=elyra-"$ELYRA_VERSION"-py3-none-any.whl

COPY requirements.txt ${ELYRA_PACKAGE}* ./

# Install Elyra
RUN python3 -m pip install --disable-pip-version-check --quiet --no-cache-dir "$ELYRA_PACKAGE"[all] && rm $ELYRA_PACKAGE

# Install custom requirements
RUN python3 -m pip install -r requirements.txt && rm requirements.txt

# Install component examples catalogs
#  - this wail fail if the 'kfp-examples' pip extra is not installed
RUN elyra-metadata create component-catalogs \
    --schema_name=elyra-kfp-examples-catalog  \
    --display_name="Kubeflow Pipelines examples" \
    --runtime_type=KUBEFLOW_PIPELINES \
    --categories="['examples']"

RUN apt update && apt install openssh-client -y

RUN echo "$NB_USER:$NB_USER" | chpasswd && adduser $NB_USER sudo

USER $NB_USER

RUN conda remove --force -y terminado && \
    python -m pip install --upgrade pip && \
    python -m pip install --ignore-installed --upgrade setuptools pandas && \
    npm config set scripts-prepend-node-path auto && \
    npm config get prefix /home/jovyan/.npm-global && \
    mkdir -p /home/jovyan/.npm-global && \
    if [ "$TAG" == "dev" ] ; then \
       npm install -g yarn && \
       npm install -g npm && \
       cd /tmp && git clone https://github.com/elyra-ai/elyra.git && \
       cd /tmp/elyra && git checkout tags/v3.2.2 -b v3.2.2 && make UPGRADE_STRATEGY=eager install && rm -rf /tmp/elyra; \
    else \
        python3 -m pip install --quiet --no-cache-dir --use-deprecated=legacy-resolver elyra[all]=="$TAG" && \
        jupyter lab build; \
    fi

CMD ["/usr/local/bin/start-elyra.sh"]
