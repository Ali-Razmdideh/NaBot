# To-Do: Make Docker Builds Multi-Platform
# Base Stage - Buster-Slim is used as a base image
FROM python:3.8-slim AS base

# Update base image. It might be helpful for security reasons 
# Install curl to use in docker-compose healthcare
RUN apt-get update -qq && \
    apt-get upgrade -qq -y && \
    apt-get autoremove -y -qq && \
    apt-get autoclean -y -qq

# Build stage
FROM base AS build-stage
# Change current directory to build
WORKDIR /build
# Copy requirements list from the host
COPY requirements.txt /build
# Activate virtual environment. 
# Download python pacakges and compile some of them
# Install binaries. and the make things clear 
ENV PATH="/opt/venv/bin:$PATH"
RUN python -m venv /opt/venv && \
    . /opt/venv/bin/activate && \
    pip install --no-cache-dir --upgrade pip wheel && \
    pip wheel --requirement=requirements.txt --wheel-dir=wheels && \
    pip install --no-deps wheels/*.whl && \
    rm -rf wheels *.egg-info
  
# Runtime stage
FROM base AS runtime-stage
# Copy installed libraries from the build stage 
COPY --from=build-stage /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

WORKDIR /action-server
VOLUME /tmp
# Copt actions from the host
COPY actions/ ./actions
# Make logs directory for action server logs storing
RUN mkdir -p logs && \
    rasa telemetry disable
# Create logs. And set an action server enabled
ENTRYPOINT ["rasa", "run", "--log-file", "logs/action-server.log", "actions", "--actions"]
CMD [ "actions" ] 
