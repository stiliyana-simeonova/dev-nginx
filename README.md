# dev-nginx

Tools to configure a local development nginx setup to proxy our applications and services with SSL using [mkcert](https://github.com/FiloSottile/mkcert).

This typically allows accessing servers via
`service.local.dev-gutools.co.uk`, rather than a `localhost:PORT` URL,
which among other things makes it possible to share cookies for the [pan-domain authentication](https://github.com/guardian/pan-domain-authentication).

## Installation
### Via Homebrew

```bash
brew tap guardian/homebrew-devtools
brew install guardian/devtools/dev-nginx

# update
brew upgrade dev-nginx
```

### Manually
- Clone this repository:

```bash
git clone git@github.com:guardian/dev-nginx.git

# update
git pull && brew bundle
```

- Install dependencies listed in the [Brewfile](./Brewfile):

```bash
brew bundle
```

- Add [`bin`](./bin) to your PATH

## Usage
`dev-nginx` has a few commands available. Find them by passing no arguments:

```console
$ dev-nginx

dev-nginx COMMAND <OPTIONS>
Available commands:
- locate-nginx
- restart-nginx
- setup-app
- setup-cert
```

### Commands
#### `locate-nginx`
```bash
dev-nginx locate-nginx
```

Locates the directory nginx is installed.

#### `restart-nginx`
```bash
dev-nginx restart-nginx
```

Stops, if running, and starts nginx.

#### `setup-cert`
```bash
dev-nginx setup-cert demo-frontend.foobar.co.uk
```

Uses `mkcert` to issue a certificate for a domain, writing it to `~/.gu/mkcert` and symlinking it into the directory nginx is installed.

#### `setup-app`
```bash
dev-nginx setup-app /path/to/nginx-mapping.yml
```

Generates config for nginx proxy site(s) from a config file, issues the certificate(s) and restarts nginx. 

##### Config format
Your application's configuration is provided as a YAML file in the following format.

**Example:**

```yaml
name: demo
domain-root: foobar.co.uk
mappings:
- port: 9001
  prefix: demo-frontend
- port: 8800
  prefix: demo-api
```

In general, the format is as follows:

```yaml
name: <name of the project, used as its filename>
mappings:
  <list of server mappings>
ssl: <optional, defaults to `true` (you are unlikely to need to change this)>
domain-root: <optional, defaults to `local.dev-gutools.co.uk`>
```

Each mapping supports the following fields:

###### prefix

**required**

This is the domain prefix used for the service and will be prepended to the domain.
The default domain is `local.dev-gutools.co.uk`
but this can be overridden using the `domain-root` key at the top level (to apply to all mappings) or in a mapping (to set the domain root for just that mapping).

###### port

**required**

This sets the port that will be proxied - i.e. the upstream backend service. These are commonly in the 9XXXs or 8XXXs.

###### websocket

This optionally sets the path for a websocket. If present, nginx will be configured to serve websocket traffic at that location.

###### client_max_body_size

Optionally instructs nginx to set a max body size on incoming requests.

###### domain-root

The domain under which the service should run, which defaults to `local.dev-gutools.co.uk`.
This can also be overriden for all mappings by specifying a `domain-root` key at the top level.

##### Hosts file
You'll need to ensure your hosts file has entries for your new domains, so that they resolve:

```
127.0.0.1   demo-frontend.foobar.co.uk
127.0.0.1   demo-api.foobar.co.uk
```
