FROM photon:2.0

ENV PYTHON_VERSION 3.6.5
ENV PYTHON_PIP_VERSION 18.0
ENV FWATCHDOG_VERSION 0.9.4

ARG ADDITIONAL_PACKAGE

RUN echo "Installing Python " \
    && yum install -y \
        python3-$PYTHON_VERSION \
        python3-xml \
        wget \
        ${ADDITIONAL_PACKAGE} \
    && echo "Installing PIP " \
    && wget -O get-pip.py 'https://bootstrap.pypa.io/get-pip.py' \
    && python3 get-pip.py \
        --disable-pip-version-check \
        --no-cache-dir \
        "pip==$PYTHON_PIP_VERSION" \
    && pip --version \
    && echo "Pulling watchdog binary from Github." \
    && curl -sSL https://github.com/openfaas/faas/releases/download/$FWATCHDOG_VERSION/fwatchdog > /usr/bin/fwatchdog \
    && chmod +x /usr/bin/fwatchdog \
    && echo "Cleaning up" \
    && yum remove -y wget \
    && rm -rf /var/cache/tdnf/* \
    && rm -f get-pip.py

# Add non root user
# RUN addgroup -S app && adduser app -S -G app
# RUN chown app /home/app

# USER app

ENV PATH=$PATH:/home/app/.local/bin

WORKDIR /home/app/

COPY index.py           .
COPY requirements.txt   .
RUN pip install -r requirements.txt

RUN mkdir -p function
RUN touch ./function/__init__.py
WORKDIR /home/app/function/
COPY function/requirements.txt	.
RUN pip install --user -r requirements.txt

WORKDIR /home/app/

# USER root
COPY function           function
# RUN chown -R app:app ./
# USER app

ENV fprocess="python3 index.py"
EXPOSE 8080

HEALTHCHECK --interval=1s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]
