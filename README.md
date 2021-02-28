# How to run Strapi as a node.js application on Plesk Obsidian using yarn and MySQL

Based on [this article](https://raoulkramer.de/deploy-run-host-strapi-on-plesk-obsidian-as-node-application/).

## Requirements

- A plesk server obviously
- Node.js extension installed
- A domain
- SSH access as root

**For git setup, check original article.**

## Configure Strapi

Here is the basic structure of a strapi app (temporary files included) after using `yarn create strapi-app {app-name} --quickstart`:

```bash
.cache/
.tmp/
api/
build/
config/
exports/
extensions/
node_modules/
public/
.gitignore
.strapi-updater.json
favicon.ico
package.json
README.md
yarn.lock
```

First we need to add mysql using `yarn add mysql`.

Now that's done we can setup our databases, let's modify `config/database.js` such as:

```js
module.exports = ({ env }) => ({
  defaultConnection: env("DATABASE_CONNECTION_NAME", "sqlite"),
  connections: {
    sqlite: {
      connector: "bookshelf",
      settings: {
        client: "sqlite",
        filename: env("DATABASE_FILENAME", ".tmp/data.db"),
      },
      options: {
        useNullAsDefault: true,
      },
    },
    mysql: {
      connector: "bookshelf",
      settings: {
        client: "mysql",
        host: env("DATABASE_HOST"),
        port: env.int("DATABASE_PORT"),
        database: env("DATABASE_NAME"),
        username: env("DATABASE_USERNAME"),
        password: env("DATABASE_PASSWORD"),
        ssl: env.bool("DATABASE_SSL", false),
      },
      options: {},
    },
  },
});
```

Env variables can be stored in a .env or in Plesk.

Create a folder document.root including a .gitkeep file, this folder is needed for plesk node.js configuration later.

Finally, create `server.js` in your project root:

```js
// https://strapi.io/documentation/v3.x/getting-started/deployment.html#application-configuration
const strapi = require("strapi");

strapi(/* {...} */).start();
```

## Configure Plesk

Create a subdomain and create a database related to it. **Write down** the database name, username and password.
Check "Allow local connections only" and validate.

> **WARNING**
> Strapi only supports Mysql >5.6, while Plesk provides 5.5. You must update it via https://support.plesk.com/hc/en-us/articles/213403429--How-to-upgrade-MySQL-5-5-to-5-6-5-7-or-MariaDB-5-5-to-10-x-on-Linux-.

In the **File explorer**, delete `index.html` and upload your strapi files, but only:

```
admin/ (if you customized the admin panel)
api/
config/
document.root/
extensions/
public/
.env (if this is where you have setup your env variables)
favicon.ico
package.json
server.js
yarn.lock
```

Now, connect to your server using SSH. I personnaly use [Putty](https://www.putty.org/).
In **Host Name**, use your server url or IP adress (in this case use the right port).

```bash
login as: root
... passsword: {your password}
```

Now that you are connected, execute the following commands:

```bash
export PATH=/opt/plesk/node/12/bin:$PATH # Necessary each time you need to use yarn


### If you don't have Yarn installed ###

npm install -g yarn

########################################


cd /var/www/vhosts/{domain}/{folder} # Move to your domain folder

yarn # Install dependencies

NODE_ENV=production yarn build # Build the app
```

Now go to your subdomain dashboard and click on the Node.js extension.

- Document root: /{folder}/document.root
- Application mode: production
- Application root: /{folder}
- Application startup file: server.js

In _Custom environment variables_ (if not set in `.env` file), add the following:

- HOST: 0.0.0.0
- PORT: 1337
- DATABASE_HOST: localhost
- DATABSE_PORT: 3306
- DATABASE_NAME: ...
- DATABASE_USERNAME: ...
- DATABASE_PASSWORD: ...
- DATABASE_CONNECTION_NAME: mysql

Then press **Restart the app**.

And that's it!

## Update the app

- Upload your files
- Connect via SSH
- ```bash
    export PATH=/opt/plesk/node/12/bin:$PATH
    cd /var/www/vhosts/{domain}/{folder}
    yarn
    NODE_ENV=production yarn build
  ```
- On Node.js subdomain settings, press **Restart the app**.

### Feel free to open an issue if you have questions or improvements.
