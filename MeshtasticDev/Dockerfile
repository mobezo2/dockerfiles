# syntax=docker/dockerfile:1
FROM ubuntu:latest
COPY . /app
RUN apt-get update && apt-get install -y screen
RUN apt-get install -y xterm featherpad

ENV DISPLAY=host.docker.internal:0.0
#RUN make /app
#CMD python /app/app.py