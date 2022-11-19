FROM rabbitmq:3.6-management
RUN apt-get -qq update && apt-get -qq -y install curl \
    && rm -rf /var/lib/apt/lists/*
COPY ./rabbit_definitions.json /etc/rabbitmq/definitions.json
HEALTHCHECK CMD curl --fail http://localhost:15672/ || exit 1
RUN rabbitmq-plugins enable --offline rabbitmq_management