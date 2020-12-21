# PoC

- show how to use Caddy to RP for multiple jupyter notebook collections prefixed by their own distinct paths

# run the sample

> clone the repo

> start the services

```sh
$ docker-compose up -d
```

> navigate to the static `index.html` file

- go to [https://localhost](https://localhost)
- you will need to accept the self-signed cert to proceed
  - `advanced -> accept and continue` in Firefox

> click the `COVID` and `Other` links

- open these in separate tabs

> log in with the test key

- key: `test`

> check each notebook collection

- `COVID` will have `covid_hello_world`
- `Other` will have `other_hello_world`

> note: in production (if your DNS records have been updated to point at the IP of the host machine) then Caddy will provision a proper cert signed by LetsEncrypt for that domain

- make sure to replace `localhost` with your domain name in the `Caddyfile`
- then Caddy can go through the ACME challenge dance to get the cert automatically

# Notes

## `Caddyfile`

```sh
# replace localhost with your public domain name
localhost

# no reason for a route block since we arent stripping /covid anymore
# proxies to the compose project-network DNS name (service name) for the service on its listening port

# covid notebooks only
reverse_proxy /covid* covid:8888
# other notebooks
reverse_proxy /other* other:8888

# file server will serve all unmatched paths as files from the root dir that you mounted as a volume
root * /usr/share/caddy
file_server
```

## Jupyter Notebooks

- each notebook collection runs as its own service and gets a corresponding `reverse_proxy` entry as seen above

  - each container should have its respective notebooks mounted into the `/home/jovyan/work` dir
  - they can have unique tokens etc since they are distinct container services

> lessons learned

- its fucking `jovyan` **NOT** `joyvan`
  - you can remember this with the classic expression: "`V before Y unless you want to die`"
- for future ref the docs you want to look for are for `jupyter/minimal-notebook` (yours specifically) OR `jupyter/base-notebook` (which it inherits from). the latter is much more prevalent for finding docs
  - docs on this relationship: https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html#jupyter-minimal-notebook
- you cant pass a `jupyter_notebook_config.py` file because its overwritten on startup by their custom one in the `Dockerfile`

  - instead you override the start command passing the config params as --ParamName.param=value format
  - [documentation on this procedure](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/common.html#notebook-options)

  - [list of options you can pass](https://jupyter-notebook.readthedocs.io/en/stable/config.html)
  - **note, no `c.` prefix like in `jupyter_notebook_config.py` file**
