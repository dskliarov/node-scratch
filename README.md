# node.js from scratch image extreme small

Only **/node** inside the container static compiled node.js

if you need npm or a alpine subsys try [mhart/alpine-node](https://hub.docker.com/r/mhart/alpine-node/)

## Images

All images follows `mrhein/node-scratch:<version tag>`

`latest` = v7

mrhein/node-scratch:v4 compressed size **8m**

```bash
○ → docker run mrhein/node-scratch:v4 -v
v4.7.2
```

mrhein/node-scratch:v6 compressed size **12m**

```bash
○ → docker run mrhein/node-scratch:v6 -v
v6.9.4
```

mrhein/node-scratch:v7 compressed size **13m**

```bash
○ → docker run mrhein/node-scratch:v7 -v
v7.4.0
```

## Examples

### Example Dockerfile

```Dockerfile
FROM mrhein/node-scratch:v4

ADD deploy.tar.gz /

EXPOSE 3000

ENTRYPOINT ["/node", "start.js", "--max_old_space_size=256"]

```

### Example start.js

```js
'use strict';

const fs = require('fs');
const spawn = require('child_process').spawn;
const out = fs.createWriteStream('logs/out.log', {flags: 'a'});
const error = fs.createWriteStream('logs/err.log', {flags: 'a'});
const combined = fs.createWriteStream('logs/combined.outerr.log', {flags: 'a'});
let suse = spawn('/node', ['app.js', process.argv], {shell: false});

suse.stdout.pipe(out);
suse.stderr.pipe(error);
suse.stdout.pipe(combined);
suse.stderr.pipe(combined);

suse.stdout.on('data', function (data) {
  process.stdout.write(data);
});

suse.stderr.on('data', function (data) {
  process.stderr.write(data);
});

suse.on('close', (code) => {
  process.stdout.write('child process exited with code ' + code);
});
```

### Example app.js used in combination with the start.js

```js
'use strict';

require('console-stamp')(console, {
    pattern : 'yyyy-mm-dd HH:MM:ss.l Z',
    colors: {
        stamp: "yellow",
        label: "white"
    }
});
const cluster = require('cluster');
const numCPUs = require('os').cpus().length;
const config = require('./config');
const env = config.env;
const debug = require('debug')('app');

if (cluster.isMaster && env !== 'development') {
  debug(`Master ${process.pid} is running`);
  for (let i = 0; i < numCPUs; i = i + 1) {
    cluster.fork();
  }
  cluster.on('exit', (worker, code, signal) => {
    debug(`worker ${worker.process.pid} died signal:` + signal);
    cluster.fork();
  });
} else {
  debug(`Worker ${process.pid} started`);
  
  // app code
}

```
