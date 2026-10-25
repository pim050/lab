FROM ubuntu:24.04

ARG NAME
ARG GREETING

ENV NAME=${NAME}
ENV GREETING=${GREETING}

WORKDIR /app

COPY greet .

RUN chmod +x greet

CMD ["./greet"]
