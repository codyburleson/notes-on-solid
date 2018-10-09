# Notes on local Solid server with Docker

**WORK IN PROGRESS**

Here, I am exploring use of the node-solid-server in a Docker container. Please be aware that this study is not yet 
complete and the notes represent a stream-of-conscious workflow. I'll go back and clean up the notes after I've found 
that I've actually learned something to share more formally.

## References

- https://github.com/solid/node-solid-server#use-docker

## Notes

Checkout (clone) the master branch of node-solid-server:
https://github.com/solid/node-solid-server.git

cd into the local repo.

`docker build -t node-solid-server .`

Run with:

`docker run -p 8443:8443 --name solid node-solid-server`

Point your web browser to verify:
https://localhost:8443

Quick acid test. Let's register  user through the web UI and see what happens.

- Register user: solid | password: solid
- Try to edit profile and get:

```
Web error: 401 (Unauthorized) on PATCH of <https://cburleson.localhost:8443/profile/card>
```

Obviously more is required than just spinning up the container. So, now I will refer to the following steps to see what 
the config looks like.

Modify the config as follows:

- Copy the config to the current directory with: `docker cp solid:/usr/src/app/config.json .`
- Edit the `config.json` file
- Copy the file back with `docker cp config.json solid:/usr/src/app/`
- Restart the server with `docker restart solid`

I used the first step to copy config.json (to my repos root) and initially, it looks like this:

```
{
  "root": "./data",
  "port": "8443",
  "serverUri": "https://localhost:8443",
  "webid": true,
  "mount": "/",
  "configPath": "./config",
  "dbPath": "./.db",
  "sslKey": "./cert.key",
  "sslCert": "./cert.pem",
  "multiuser": true,
  "corsProxy": "/proxy"
}
```

Lets now look at the file system in the container. We get shell access into the container with the following command:

`docker exec -it solid bash`

Then we can `ls -la` to see...

```
drwxr-xr-x   1 root root   4096 Oct  6 06:38 .
drwxr-xr-x   1 root root   4096 Jun  6 20:14 ..
drwxr-xr-x   3 root root   4096 Oct  6 06:38 .db
drwxr-xr-x   7 root root   4096 Oct  6 05:53 .git
-rwxr-xr-x   1 root root    248 Oct  6 05:53 .gitignore
-rwxr-xr-x   1 root root    310 Oct  6 05:53 .npmignore
-rwxr-xr-x   1 root root   1551 Oct  6 05:53 .snyk
-rwxr-xr-x   1 root root    612 Oct  6 05:53 .travis.yml
-rwxr-xr-x   1 root root   5640 Oct  6 05:53 CHANGELOG.md
-rwxr-xr-x   1 root root   3775 Oct  6 05:53 CONTRIBUTING.md
-rwxr-xr-x   1 root root    314 Oct  6 05:53 Dockerfile
-rwxr-xr-x   1 root root   2175 Oct  6 05:53 EXAMPLES.md
-rwxr-xr-x   1 root root   1089 Oct  6 05:53 LICENSE
-rwxr-xr-x   1 root root  16759 Oct  6 05:53 README.md
drwxr-xr-x   3 root root   4096 Oct  6 05:53 bin
-rw-r--r--   1 root root   3272 Oct  6 06:37 cert.key
-rw-r--r--   1 root root   1984 Oct  6 06:37 cert.pem
drwxr-xr-x   4 root root   4096 Oct  6 05:53 common
drwxr-xr-x   1 root root   4096 Oct  6 06:38 config
-rwxr-xr-x   1 root root    276 Oct  6 05:53 config.json
-rwxr-xr-x   1 root root    276 Oct  6 05:53 config.json-default
drwxr-xr-x   1 root root   4096 Oct  6 06:54 data
drwxr-xr-x   5 root root   4096 Oct  6 05:53 default-templates
drwxr-xr-x   4 root root   4096 Oct  6 05:53 default-views
drwxr-xr-x   2 root root   4096 Oct  6 05:53 examples
-rwxr-xr-x   1 root root    160 Oct  6 05:53 index.js
drwxr-xr-x   6 root root   4096 Oct  6 05:53 lib
drwxr-xr-x 732 root root  28672 Oct  6 06:37 node_modules
-rwxr-xr-x   1 root root 368479 Oct  6 05:53 package-lock.json
-rwxr-xr-x   1 root root   3251 Oct  6 05:53 package.json
drwxr-xr-x   2 root root   4096 Oct  6 05:53 static
drwxr-xr-x   6 root root   4096 Oct  6 05:53 test
```

It looks like `root` and `dbPath` in `config.json` point to two directories where data is written. I'm looking at this 
because I want to map volumes instead of having my data directly in the volatile container. I'll swing back around to 
this later and probably create a `docker-compose.yml` file that does the volume mappings. For now, however, I need 
to figure out how to more properly configure the instance so that it actually works without errors.

I now refer to the resource, [INSTALLING AND RUNNING NODE SOLID SERVER](https://solid.inrupt.com/docs/installing-running-nss), 
which details how to setup Solid Server on Debian GNU/Linux. I think it will provide some clues to the configuration 
options.

Right now I'm stuck editing my profile; getting error:

`Web error: 500 (Internal Server Error) on PATCH of <https://solid.localhost:8443/profile/card>`

Posting question in the `node-solid-server` Gitter channel.`

To be continued...

