=================
Backend Commands
=================

``augur backend``
====================
The ``augur backend`` CLI group is for controlling Augur's API server & data collection workers. All commands are invoked like::

  $ augur backend <command name>

``start``
============
This command is for starting Augur's API server & (optionally) data collection workers. Example usages are shown below the parameters. After starting up, it will run indefinitely (but might not show any output, unless it's being queried or the data collection is working).

--disable-collection      Flag that turns off the data collection. Useful for testing the REST API or if you want to pause data collection without editing your config.

--skip-cleanup      Flag that disables the old process cleanup that runs before Augur starts. Useful for Python scripts where Augur needs to be run in the background: see the `test/api/runner.py` file for an example.

**To start the backend normally:**

.. code-block:: 

  uv run augur backend start

  # successful output looks like:
  >[43389] augur [INFO] Augur application initialized
  >[43389] augur [INFO] Booting manager
  >[43389] augur [INFO] Booting broker
  >[43389] augur.housekeeper [INFO] Booting housekeeper
  >[43389] augur.housekeeper [INFO] Preparing housekeeper jobs
  >[43389] augur.housekeeper [INFO] Scheduling update processes
  >[43389] augur [INFO] Booting facade_worker #1
  >[43389] augur [INFO] Booting github_worker #1
  >[43389] augur [INFO] Booting linux_badge_worker #1
  >[43389] augur [INFO] Booting pull_request_worker #1
  >[43389] augur [INFO] Booting repo_info_worker #1
  >[43389] augur [INFO] Booting contributor_worker #1
  >[43389] augur [INFO] Booting gitlab_issues_worker #1
  >[43389] augur [INFO] Booting release_worker #1
  >[43389] augur [INFO] Starting Gunicorn server in the background...
  >[43389] augur [INFO] Housekeeper update process logs will now take over.
  >[43645] augur.jobs.insights [INFO] Housekeeper spawned insights model updater process for repo group id 0
  >[43639] augur.jobs.issues [INFO] Housekeeper spawned issues model updater process for repo group id 0
  >[43646] augur.jobs.badges [INFO] Housekeeper spawned badges model updater process for repo group id 0
  >[43640] augur.jobs.pull_request_commits [INFO] Housekeeper spawned pull_request_commits model updater process for repo group id 0
  >[43642] augur.jobs.commits [INFO] Housekeeper spawned commits model updater process for repo group id 0
  >[43647] augur.jobs.value [INFO] Housekeeper spawned value model updater process for repo group id 0
  >[43644] augur.jobs.contributors [INFO] Housekeeper spawned contributors model updater process for repo group id 0
  >[43641] augur.jobs.repo_info [INFO] Housekeeper spawned repo_info model updater process for repo group id 0
  >[43643] augur.jobs.pull_requests [INFO] Housekeeper spawned pull_requests model updater process for repo group id 0
  >[43648] augur.jobs.pull_request_files [INFO] Housekeeper spawned pull_request_files model updater process for repo group id 0
  > ...
  > From this point on, the housekeeper and broker logs detailing the worker's progress will take over

To start the backend as a background process: 

.. code-block:: bash

  uv run nohup augur backend start >logs/base.log 2>logs/base.err &
  
Successful output looks like the generation of standard Augur logfiles in the logs/ directory.

To start the backend server without the housekeeper:

.. code-block:: bash

  uv run augur backend start --disable-housekeeper

Successful output looks like:

.. code-block:: bash

  > [14467] augur [INFO] Augur application initialized
  > [14467] augur [INFO] Using config file: /Users/carter/workspace/chaoss/augur/augur.config.json
  > [14467] augur [INFO] Starting Gunicorn webserver...
  > [14467] augur [INFO] Augur is running at: http://0.0.0.0:5000
  > [14467] augur [INFO] Gunicorn server logs & errors will be written to logs/gunicorn.log


``stop``
---------
**Gracefully** attempts to stop all currently running backend Augur processes, including any workers. Will only work in a virtual environment.

Example usage:

.. code-block:: bash

  uv run augur backend stop

Successful output looks like:

.. code-block:: bash

  > CLI: [backend.stop] [INFO] Stopping process 33607
  > CLI: [backend.stop] [INFO] Stopping process 33775
  > CLI: [backend.stop] [INFO] Stopping process 33776
  > CLI: [backend.stop] [INFO] Stopping process 33777

``kill``
---------
**Forcefully** terminates (using ``SIGKILL``) all currently running backend Augur processes, including any workers. Will only work in a virtual environment.
Should only be used when ``uv run augur backend stop`` is not working.

Example usage:

.. code-block:: bash

  uv run augur backend kill

  # successful output looks like:
  > CLI: [backend.kill] [INFO] Killing process 87340
  > CLI: [backend.kill] [INFO] Killing process 87573
  > CLI: [backend.kill] [INFO] Killing process 87574
  > CLI: [backend.kill] [INFO] Killing process 87575
  > CLI: [backend.kill] [INFO] Killing process 87576


``processes``
--------------
Outputs the process ID (PID) of all currently running backend Augur processes, including any workers. Will only work in a virtual environment.

Example usage:

.. code-block:: bash

  uv run augur backend processes

Successful output looks like:

.. code-block:: bash

  > CLI: [backend.processes] [INFO] Found process 14467
  > CLI: [backend.processes] [INFO] Found process 14725


To enable log parsing for errors, you need to install `Elasticsearch <https://www.elastic.co/downloads/elasticsearch>`_ and `Logstash <https://www.elastic.co/downloads/past-releases/logstash-6-8-10>`_.

.. warning::
   Please note, that Logstash v7.0 and above has unresolved issues that affect this functionality.
   In order to use it in the near future, please download v6.8.
   If you use a package manager, it defaults to v7+, so we recommend downloading `binary <https://www.elastic.co/downloads/past-releases/logstash-6-8-10>`_.
   This change is tested with Elasticserach v7.8.0_2 and Logstash v6.8.10.

Set ``ELASTIC_SEARCH_PATH`` and ``LOGSTASH_PATH`` variables to point to elasticsearch and logstash binaries. For example:

.. code-block:: bash

  # If not specified, defaults to /usr/local/bin/elasticsearch
  $ export ELASTIC_SEARCH_PATH=<path_to_elastic_search_binary>

  # If not specified, defaults to /usr/local/bin/logstash
  $ export LOGSTASH_PATH=<path_to_logstash_binary>

  $ export ROOT_AUGUR_DIRECTORY=<path_to_augur>

Start the http server with::

  $ cd $ROOT_AUGUR_DIRECTORY/log_analysis/http
  $ python http_server.py

Then start Augur with ``logstash`` flag::

  $ uv run augur backend start --logstash

If you'd like to clean all previously collected errors, run::

  $ uv run augur backend start --logstash-with-cleanup

Open http://localhost:8003 and select workers to check for errors.


``export-env``
---------------
Exports your GitHub key and database credentials to 2 files. The first is ``augur_export_env.sh`` which is an executable shell script that can be used to initialize environment variables for some of your credentials. The second is ``docker_env.txt`` which specifies each credential in a key/value pair format that is used to configure the backend Docker containers.

Example usage:

.. code-block:: bash

  # to export your environment
  $ uv run augur util export-env

Successful output looks like:

.. code-block:: bash

  > CLI: [util.export_env] [INFO] Exporting AUGUR_GITHUB_API_KEY
  > CLI: [util.export_env] [INFO] Exporting AUGUR_DB_HOST
  > CLI: [util.export_env] [INFO] Exporting AUGUR_DB_NAME
  > CLI: [util.export_env] [INFO] Exporting AUGUR_DB_PORT
  > CLI: [util.export_env] [INFO] Exporting AUGUR_DB_USER
  > CLI: [util.export_env] [INFO] Exporting AUGUR_DB_PASSWORD

  # contents of augur_export_env.sh
  #!/bin/bash
  export AUGUR_GITHUB_API_KEY="your_key_here"
  export AUGUR_DB_HOST="your_host"
  export AUGUR_DB_NAME="your_db_name"
  export AUGUR_DB_PORT="your_db_port"
  export AUGUR_DB_USER="your_db_user"
  export AUGUR_DB_PASSWORD="your_db_password"

  # contents of docker_env.txt
  AUGUR_GITHUB_API_KEY="your_key_here"
  AUGUR_DB_HOST="your_host"
  AUGUR_DB_NAME="your_db_name"
  AUGUR_DB_PORT="your_db_port"
  AUGUR_DB_USER="your_db_user"
  AUGUR_DB_PASSWORD="your_db_password"


``repo-reset``
---------------
Refresh repo collection to force data collection. Mostly for debugging.

Example usage:

.. code-block:: bash

  # to reset the repo collection status to "New"
  $ uv run augur util repo-reset

  # successful output looks like:
  > CLI: [util.repo_reset] [INFO] Repos successfully reset
