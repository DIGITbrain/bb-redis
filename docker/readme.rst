=====
Redis
=====

About
=====
**Redis** [1]_ is an open-source, networked, in-memory, key-value data store with optional durability.

Version
-------
Redis version **7.x** deployed based on the official Docker Hub image: [2]_.

License
-------
**BSD 3 Clause** [3]_

Pre-requisites
==============
* *docker* installed
* access to DIGITbrain private docker repo (username, password) to pull the image:

  - ``docker login dbs-container-repo.emgora.eu``
  - ``docker pull dbs-container-repo.emgora.eu/redis:7``

Usage
=====
.. code-block:: bash

  docker run -d --rm \
      --name redis \
      -e REDIS_USER=myredisuser \
      -e REDIS_PASSWORD=myredispassword \
      -p 16379:6379 \
      redis:7

where REDIS_USER and REDIS_PASSWORD parameters create a new database user with the given username and password,
and standard Redis port 6379 is opened on as port 16379 the host.

.. warning::
  Always update the ``REDIS_USER`` and ``REDIS_PASSWORD`` parameters with the values of your choice
  prior to running this container.


Security
========
The image uses SSL/TLS traffic encryption and username-password authentication (*digitbrain/digitbrain*) by default.
The pre-built server certificate signed by DIGITbrain CA.

You can override all these settings with your own (certificates, CA file, configuration, password), see below.

Configuration
=============

Parameters
----------
.. list-table::
   :header-rows: 1

   * - Name
     - Example
     - Comment
   * - *username*
     - ``-e REDIS_USER=myredisuser``
     - Username for a newly created user
   * - *password*
     - ``-e REDIS_PASSWORD=myredispassword``
     - Password for a newly created user

Ports
-----
.. list-table::
  :header-rows: 1

  * - Container port
    - Host port bind example
    - Comment
  * - *Redis port*
    - ``-p 16379:6379``
    - Default Redis port 6379 is opened as port 16379 on the host

Volumes
-------
.. list-table::
  :header-rows: 1

  * - Name
    - Volume mount example
    - Comment
  * - *Data*
    - ``-v $PWD/redis-data:/data``
    - Redis data persisted on host's ./redis-data directory
  * - *Configuration*
    - ``-v $PWD/redis.conf:/home/redis/redis.conf``
    - Custom Redis configuration file. Refer to [4]_ for details.
  * - *Certificate Authority file*
    - ``-v $PWD/certificates/ca.crt:/home/redis/certs/ca.pem``
    - Custom Certificate Authority (CA) certificate
  * - *Server certificate*
    - ``-v $PWD/certificates/redis.crt:/home/redis/certs/server-cert.pem``
    - Custom server certificate
  * - *Server key*
    - ``-v $PWD/certificates/redis.key:/home/redis/certs/server-key.pem``
    - Custom server certificate
  * - *Users*
    - ``-v $PWD/certificates/redis.key:/home/redis/users.acl``
    - Usernames, passwords, ACLs. Refer to [5]_ for details.

References
==========
.. [1] https://redis.io/

.. [2] https://hub.docker.com/_/redis/

.. [3] https://github.com/redis/redis/blob/unstable/COPYING

.. [4] https://raw.githubusercontent.com/redis/redis/6.0/redis.conf

.. [5] https://redis.io/topics/acl

