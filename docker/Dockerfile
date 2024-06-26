# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.

FROM --platform=linux/amd64 mambaorg/micromamba:1.5.5-jammy

# Create new user
ARG NEW_MAMBA_USER_ID=57440
ARG NEW_MAMBA_USER_GID=57440


USER root

RUN if grep -q '^ID=alpine$' /etc/os-release; then \
  # alpine does not have usermod/groupmod
  apk add --no-cache --virtual temp-packages shadow=4.13-r0; \
  fi
RUN if [ "$(id ${MAMBA_USER} -u)" != "$NEW_MAMBA_USER_ID" ]; then \
  usermod "-u ${NEW_MAMBA_USER_ID}" "${MAMBA_USER}"; \
  fi
RUN if [ "$(id ${MAMBA_USER} -g)" != "$NEW_MAMBA_USER_GID" ]; then \
  groupmod -o -g ${NEW_MAMBA_USER_GID} ${MAMBA_USER} && \
  usermod -g ${NEW_MAMBA_USER_GID} ${MAMBA_USER}; \
  fi
RUN if grep -q '^ID=alpine$' /etc/os-release; then \
  # remove the packages that were only needed for usermod/groupmod
  apk del temp-packages; \
  fi

USER $MAMBA_USER

WORKDIR /home/$MAMBA_USER/jupyterlab_cache

COPY --chown=$MAMBA_USER:$MAMBA_USER docker/environment.yml ./docker/environment.yml

RUN  micromamba install -n base -c conda-forge git rsync -y && micromamba install -y -n base -f ./docker/environment.yml && micromamba clean --all --yes

COPY --chown=$MAMBA_USER:$MAMBA_USER . .

RUN micromamba run jlpm install

WORKDIR /home/$MAMBA_USER/jupyterlab

RUN micromamba run rsync -ar /home/$MAMBA_USER/jupyterlab_cache/. /home/$MAMBA_USER/jupyterlab && micromamba run python -m pip install -e  ".[dev,docs,test]"

EXPOSE 8888

WORKDIR  /home/$MAMBA_USER/jupyterlab
