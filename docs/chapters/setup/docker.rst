.. _docker:


DockerSpawner
=============

JupyterHub provides a range of spawners (method of starting new Jupyter environments for individual users). One such package is called 
`DockerSpawner <https://github.com/jupyterhub/dockerspawner>`__ and that creates new Jupyter environments for each user in a separate Docker 
container.

Please ensure DockerSpawner is working for your JupyterHub.

Installing cdsdashboards
~~~~~~~~~~~~~~~~~~~~~~~~

Install the cdsdashboards package in the JupyterHub Python environment:

::

    pip install cdsdashboards

Not all dependencies are strictly required in both environments - work is underway to split these out into separate installation tracks.


Generic Changes to jupyterhub_config.py
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

    c.JupyterHub.allow_named_servers = True

    c.DockerSpawner.name_template = "{prefix}-{username}-{servername}"

    c.DockerSpawner.image = 'ideonate/jh-voila-oauth-scipy:latest'


The changes above are mostly for guidance - you may want to use your own Docker image, but it should follow the guidelines listed 
later in this document.

The allow_named_servers option is a standard JupyterHub option where every user gets more than just their single 'My Server' Jupyter environment. 
They can add extra environments by specifying a name. ContainDS Dashboards makes use of this by running the presentation servers as named servers - 
they are really servers just like the original Jupyter notebook servers, but running Voila or another system instead.

For 'named servers' to run correctly with DockerSpawner, you need to specify a name_template based on both username and servername - the example 
above is fine.

Enabling Dashboards in jupyterhub_config.py
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following options in your config file will install the extension. Please note if you have existing customisations you may need to merge these with your existing settings.

::

    from cdsdashboards.app import CDS_TEMPLATE_PATHS
    from cdsdashboards.hubextension import cds_extra_handlers

    c.JupyterHub.template_paths = CDS_TEMPLATE_PATHS
    c.JupyterHub.extra_handlers = cds_extra_handlers

    c.CDSDashboardsConfig.builder_class = 'cdsdashboards.builder.dockerbuilder.DockerBuilder'


If you are using The Littlest JupyterHub, you can copy any new settings into a new file called 
/opt/tljh/config/jupyterhub_config.d/cdsdashboards_config.py, although ideally you would set allow_named_servers and the DockerSpawner settings 
through tljh-config (but that doesn't matter if you're just trying it out).

Docker Image Requirements
~~~~~~~~~~~~~~~~~~~~~~~~~

The image you use for the single-user server should have `Voila <https://voila.readthedocs.io/en/stable/>`__ and 
`jhsingle-native-proxy <https://github.com/ideonate/jhsingle-native-proxy>`__ installed, plus a second 'entrypoint' at 
/opt/conda/bin/voila-entrypoint.sh to run the Voila server.

Please see `docker-singleuser-images <https://github.com/ideonate/cdsdashboards/tree/master/docker-singleuser-images>`__ for an example, 
or use the ones created for you already:

- `jh-voila-oauth-singleuser:latest <https://hub.docker.com/r/ideonate/jh-voila-oauth-singleuser>`__
- `jh-voila-oauth-scipy:latest <https://hub.docker.com/r/ideonate/jh-voila-oauth-scipy>`__
- `jh-voila-oauth-r:latest <https://hub.docker.com/r/ideonate/jh-voila-oauth-r>`__
- `jh-voila-oauth-datascience:latest <https://hub.docker.com/r/ideonate/jh-voila-oauth-datascience>`__

These are based on the similarly-named `Jupyter Docker Stacks <https://jupyter-docker-stacks.readthedocs.io/en/latest/>`__ images, just 
with extra support for Voila and ContainDS Dashboards. The ContainDS Dockerfile versions are 
on `GitHub here <https://github.com/ideonate/cdsdashboards-jupyter-docker>`__.

Options
~~~~~~~

Extra options to control behavior of Dashboards are available - see :ref:`options<options>`.