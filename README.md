# Instructions

### GitHub Actions and Digital Ocean simple CI/CD

#### <mark style="background-color:blue;">Continuous Integration</mark> <a href="#heading-continuous-deployment-to-digitalocean" id="heading-continuous-deployment-to-digitalocean"></a>

#### Create a NestJS App

`git clone https://github.com/nestjs/typescript-starter.git netjs-sample` \
`cd netjs-sample npm install`

#### Create GitHub Repo

`git remote rm origin`\
`git remote add origin https://github.com/<username>/netjs-sample.git`\
`git branch -M master git push -u origin master`

#### Add Integration Workflow

`mkdir -p .github/workflows`\
`cd .github/workflows`\
`vi ci.yml`

```yaml
name: Continuous Integration

on:
  pull_request:
    branches:
      - master

jobs:
  merge_pull_request:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout master branch
        uses: actions/checkout@v2

      - name: Use Node.js
        uses: actions/setup-node@v1
        with:
          node-version: '14.x'

      - name: Install dependencies
        run: npm ci

      - name: Test application
        run: npm test

      - name: Build application
        run: npm run build
```

#### Create feature-test branch

`git checkout -b feature-test`

#### Edit `Hello World!` in `feature-test` branch

`#src > app.controller.spec`

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { AppController } from './app.controller';
import { AppService } from './app.service';

describe('AppController', () => {
  let app: TestingModule;

  beforeAll(async () => {
    app = await Test.createTestingModule({
      controllers: [AppController],
      providers: [AppService],
    }).compile();
  });

  describe('getHello', () => {
    it('should return "Hello World!"', () => {
      const appController = app.get<AppController>(AppController);
      expect(appController.getHello()).toBe('Hello World!');
    });
  });
});
```

`#src > app.service.ts`

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello World!';
  }
}
```

Check if Continuos Integration works like in this example:\
[https://github.com/angpab/netjs-sample/actions/workflows/ci.yml](https://github.com/angpab/netjs-sample/actions/workflows/ci.yml)

#### <mark style="background-color:green;">Continuous Deployment</mark> <a href="#heading-continuous-deployment-to-digitalocean" id="heading-continuous-deployment-to-digitalocean"></a>

#### Create a DO Droplet

This is a direct link for a droplet creation.
https://cloud.digitalocean.com/droplets/new?distro=ubuntu-20-04-x64


#### Create self-hosted runner

`Settings > Actions > Runners > New self-hosted runner`

![](<.gitbook/assets/Screen Shot 2022-04-21 at 19.55.06.png>)

#### Select Runner image, Architecture and follow the given instructions.

![](<.gitbook/assets/Screen Shot 2022-04-21 at 19.57.17.png>)

#### Login

`ssh root@public_ipv4`

**Update packs list and upgrade installed packages**

`apt-get update`\
`apt-get upgrade`

#### Install sudo

`apt-get install sudo`

#### Create user and login as

`useradd -g sudo -d /home/ghact -m ghact`\
`passwd ghact`\
`su - ghact`

#### Configure the runner and start the runner as a Linux service.

`./config.sh --url https://github.com//repo_name --token your_token`\
`sudo ./svc.sh install`\
`sudo ./svc.sh start`

#### Set up the project dependencies (Node.js and [PM2 Process Manager](https://pm2.keymetrics.io)).

`cd ~`\
`curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -`\
`sudo apt-get install -y nodejs`\
`sudo npm install pm2 -g`

#### Start App with PM2

`cd /home/ghact/actions-runner/_work/netjs-sample`\
`pm2 start dist/main.js --name MyApp`

**Add Deployment Workflow**

`cd .github/workflows`\
`vi cd.yml`

```yaml
name: Continuous Deployment

on:
  pull_request:
    types: [ closed ]
    branches:
      - master

jobs:
  deployment:
    runs-on: self-hosted
    steps:
      - name: Checkout master branch
        uses: actions/checkout@v2

      - name: Setup Node.js
        uses: actions/setup-node@v1
        with:
          node-version: '14.x'

      - name: Install dependencies
        run: npm ci

      - name: Test application
        run: npm test

      - name: Build application
        run: npm run build

      - name: Restart server application
        run: pm2 restart MyApp
```

#### Visit  Server IP Address and Port though a browser

![http://ip\_address:3000](<.gitbook/assets/Screen Shot 2022-04-21 at 21.18.51.png>)

#### Short shell log

```bash
➜  ~ ssh root@mypublicip
root@simple-netjs:~# apt-get update
root@simple-netjs:~# apt-get upgrade
root@simple-netjs:~# apt-get install sudo
root@simple-netjs:~# useradd -g sudo -d /home/ghact -m ghact
root@simple-netjs:~# passwd ghact
root@simple-netjs:~# su - ghact
$ mkdir actions-runner && cd actions-runner
$ curl -o actions-runner-linux-x64-2.290.1.tar.gz -L https://github.com/actions/runner/releases/download/v2.290.1/actions-runner-linux-x64-2.290.1.tar.gz
$ echo "2b97bd3f4639a5df6223d7ce728a611a4cbddea9622c1837967c83c86ebb2baa  
$ tar xzf ./actions-runner-linux-x64-2.290.1.tar.gz
$ ./config.sh --url https://github.com/angpab/netjs-sample --token MYTOKENGOESHEREBLABLABLABLABL
$ sudo ./svc.sh install
$ sudo ./svc.sh start
$ cd ~
$ curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
$ sudo apt-get install -y nodejs
$ sudo npm install pm2 -g
$ cd /home/ghact/actions-runner/_work/netjs-sample
$ pm2 start dist/main.js --name MyApp
[PM2] Starting /home/ghact/actions-runner/_work/netjs-sample/netjs-sample/dist/main.js in fork_mode (1 instance)
[PM2] Done.
┌────┬────────────────────┬──────────┬──────┬───────────┬──────────┬──────────┐
│ id │ name               │ mode     │ ↺    │ status    │ cpu      │ memory   │
├────┼────────────────────┼──────────┼──────┼───────────┼──────────┼──────────┤
│ 0  │ MyApp              │ fork     │ 0    │ online    │ 0%       │ 30.4mb   │
└────┴────────────────────┴──────────┴──────┴───────────┴──────────┴──────────┘
$
```