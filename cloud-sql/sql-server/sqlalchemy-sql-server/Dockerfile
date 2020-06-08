# Copyright 2020 Google, LLC.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Use the official Python image.
# https://hub.docker.com/_/python
FROM python:3.8-buster

#Download the desired package(s) for Microsoft ODBC 17 Driver
RUN (curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -) && \
        curl https://packages.microsoft.com/config/debian/10/prod.list > /etc/apt/sources.list.d/mssql-release.list && \
        apt-get update && \
        ACCEPT_EULA=Y apt-get install -y msodbcsql17  unixodbc-dev && \
        ACCEPT_EULA=Y apt-get install -y mssql-tools && \
        echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile && \
        echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc

# In the app logs in GAE, we are getting:
# File "/usr/local/lib/python3.8/site-packages/sqlalchemy/engine/default.py", line 490, in connect
# return self.dbapi.connect(*cargs, **cparams) pyodbc.InterfaceError:
# ('IM002', '[IM002] [unixODBC][Driver Manager]Data source name not found and no default driver specified (0) (SQLDriverConnect)')

# Copy application dependency manifests to the container image.
# Copying this separately prevents re-running pip install on every code change.
COPY requirements.txt ./

# Install production dependencies.
RUN pip install -r requirements.txt && \
         pip install gunicorn

# Copy local code to the container image.
ENV APP_HOME /app
WORKDIR $APP_HOME
COPY . ./


# Run the web service on container startup. Here we use the gunicorn
# webserver, with one worker process and 8 threads.
# For environments with multiple CPU cores, increase the number of workers
# to be equal to the cores available.
CMD exec gunicorn --bind :$PORT --workers 1 --threads 8 main:app
