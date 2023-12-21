---
title: 如何使用PM2部署Node.js应用
slug: deploy-nodejs-using-pm2
date: 2023-04-16
coverImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.5.2/img/gallery/u/giuseppe-murabito-DhtXQxSH2Jk-unsplash.jpg
thumbnailImagePosition: right
thumbnailImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.5.2/img/gallery/u/giuseppe-murabito-DhtXQxSH2Jk-unsplash.jpg
categories:
- dev
tags:
- pm2
- nodejs
---


PM2是一个用于管理和保持Node.js应用状态的进程管理器。它可以用于部署和监控你的Node.js应用程序。本文将介绍如何使用PM2部署Node.js应用。

<!--more-->

{{< toc >}}

## 为什么要使用PM2

使用PM2部署Node.js应用程序有几个好处:
1. 进程管理。PM2可以启动、停止、重启和守护你的Node.js应用程序，它将应用程序作为进程守护在后台运行。
2. 持久化。PM2可以在服务器重启后自动重启你的Node.js应用，这意味着你的应用将始终处于运行状态。
3. 负载均衡。PM2可以轻松制作一个本地的负载均衡器来平衡你的Node.js应用的多个实例之间的传入请求。
4. 监控。PM2提供一个简单的仪表板来监控你所有的Node.js应用程序(CPU、内存、进程数等)。
5. 日志管理。PM2聚合了所有应用程序的日志,可以轻松管理和查询日志。

## 使用PM2部署Node.js应用

安装PM2:

```bash
npm install pm2@latest -g
```

进入你的应用程序目录:

```bash 
cd your-app/
```

启动应用程序:
```bash
pm2 start npm --name "your-app-name" -- run start 
```

查看应用程序状态:
```bash
pm2 list
# 或 pm2 ls
# 或 pm2 status
```

你将看到你的应用程序名称以及相关信息(进程 ID、内存使用情况、重启次数等)：

```shell
┌────┬────────────┬─────────────┬─────────┬─────────┬──────────┬────────┬──────┬───────────┬──────────┬──────────┬──────────┬──────────┐
│ id │ name       │ namespace   │ version │ mode    │ pid      │ uptime │ ↺    │ status    │ cpu      │ mem      │ user     │ watching │
├────┼────────────┼─────────────┼─────────┼─────────┼──────────┼────────┼──────┼───────────┼──────────┼──────────┼──────────┼──────────┤
│ 0  │ app1       │ default     │ 2.16.3  │ cluster │ 18259    │ 37s    │ 0    │ online    │ 0%       │ 93.9mb   │ root     │ disabled │
│ 1  │ app2       │ default     │ 2.16.3  │ cluster │ 18289    │ 11s    │ 0    │ online    │ 0%       │ 114.5mb  │ root     │ disabled │
└────┴────────────┴─────────────┴─────────┴─────────┴──────────┴────────┴──────┴───────────┴──────────┴──────────┴──────────┴──────────┘
```

重启应用程序:

```bash
pm2 restart your-app-name
```

停止应用程序：
```bash 
pm2 stop your-app-name
```

PM2 也有一个简单的仪表板可以监控你的应用程序：
```bash
pm2 monit
```

最后，你可以通过运行以下命令，在服务器重启后使 PM2 自动启动：

```bash
pm2 startup
```

`pm2 startup`是PM2的一个命令，它的主要作用是创建一个系统服务，以便在系统重启时自动启动PM2进程管理器和已经被管理的应用程序。

具体来说，`pm2 startup`执行以下操作：

- 检测当前系统使用哪种初始化系统（例如，`systemd`、`upstart`、`SysV`等），并生成相应的启动脚本。
- 将这个启动脚本复制到系统服务目录中（例如，`/etc/systemd/system`目录），并设置服务启动时的用户、环境变量以及其他相关信息。
- 最后，它会生成一条命令，要求用户以管理员权限运行该命令，并将其复制到终端中执行。这个命令将启用刚刚创建的系统服务，并在系统重启时自动启动 PM2 进程管理器和已经被管理的应用程序。这样，用户就不必手动启动这些进程，从而提高了系统的可靠性和稳定性。

如果停用自动启动，运行命令：`pm2 unstartup`。

以下是执行完命令`pm2 startup`的输出信息：

```shell
[PM2] Init System found: systemd
Platform systemd
Template
[Unit]
Description=PM2 process manager
Documentation=https://pm2.keymetrics.io/
After=network.target

[Service]
Type=forking
User=root
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
Environment=PATH=/root/.nvm/versions/node/v18.15.0/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
Environment=PM2_HOME=/root/.pm2
PIDFile=/root/.pm2/pm2.pid
Restart=on-failure

ExecStart=/root/.nvm/versions/node/v18.15.0/lib/node_modules/pm2/bin/pm2 resurrect
ExecReload=/root/.nvm/versions/node/v18.15.0/lib/node_modules/pm2/bin/pm2 reload all
ExecStop=/root/.nvm/versions/node/v18.15.0/lib/node_modules/pm2/bin/pm2 kill

[Install]
WantedBy=multi-user.target

Target path
/etc/systemd/system/pm2-root.service
Command list
[ 'systemctl enable pm2-root' ]
[PM2] Writing init configuration in /etc/systemd/system/pm2-root.service
[PM2] Making script booting at startup...
[PM2] [-] Executing: systemctl enable pm2-root...
Created symlink /etc/systemd/system/multi-user.target.wants/pm2-root.service → /etc/systemd/system/pm2-root.service.
[PM2] [v] Command successfully executed.
+---------------------------------------+
[PM2] Freeze a process list on reboot via:
$ pm2 save

[PM2] Remove init script via:
$ pm2 unstartup systemd
```


{{< alert info >}}
[`systemd`](https://en.wikipedia.org/wiki/Systemd)是一个Linux系统初始化系统和服务管理器，是目前主流Linux发行版（如Ubuntu、Debian、CentOS、Red Hat等）的默认初始化系统。它负责启动系统服务、管理进程、处理日志、控制系统休眠和唤醒等任务。

`systemd`的主要目标是提高系统启动速度和效率，统一系统管理，简化配置文件格式，提高可靠性和可维护性。它采用一种事件驱动的方式管理系统，所有的系统服务都以单独的进程运行，并由`systemd`统一管理。
{{< /alert >}}


## 使用`ecosystem.config.js`管理应用程序

PM2可以使用`ecosystem.config.js`文件来管理多个应用程序，这比手动启动每个应用程序更加结构化和可维护。`ecosystem.config.js`是一个模块，它导出用于定义应用程序环境的配置。

下面这个简单示例用来启动Nuxt应用程序的两个进程：`app1`和`app2`，分别使用端口`3000`和`3001`。同时在Nginx中配置了`upstream server`指向这两个端口做负载均衡，以方便应用升级的时候可以不影响后台服务。

```js
module.exports = {
    apps: [
      {
        name: 'app1',
        exec_mode: 'cluster',
        port: 3000,
        instances: '4', // Or a number of instances
        script: './node_modules/nuxt/bin/nuxt.js',
        args: 'start'
      },
      {
        name: 'app2',
        exec_mode: 'cluster',
        port: 3001,
        instances: '4', // Or a number of instances
        script: './node_modules/nuxt/bin/nuxt.js',
        args: 'start'
      }
    ]
  }
```

要启动此配置，运行:
```bash
pm2 start ecosystem.config.js 
```

或者可以单独启动某一个应用进程：
```bash
pm2 start app1
```

如果我们需要升级更新该Nuxt应用，我们只需将代码更新到服务器中，依次运行下面的命令，即可做到在升级的时候后台服务不需间断。

```shell
npm run build
pm2 stop app1
pm2 start app1
pm2 stop app2
pm2 start app2
```

## PM2的进程执行模式

在上面的示例中，我们看到一个配置`exec_mode: 'cluster'`，这表示PM2的进程执行模式选择`cluster`模式。

PM2有两种进程执行模式：
1. `fork`模式是默认的执行模式，它使用单个主进程来管理所有的子进程。在这种模式下，每个应用程序都会被复制多次，以创建多个独立的子进程。每个子进程都可以独立地处理请求，但它们之间不会共享状态或内存。
2. `cluster`模式是一种高级模式，它使用多个主进程来管理所有的子进程。在这种模式下，每个应用程序只会被复制一次，并由多个子进程共享。这些子进程可以共享状态和内存，并且可以使用IPC通信来协调工作。

这两种模式的主要区别在于PM2使用Node.js的`child_process.fork api`或`cluster api`：
- 在`fork`模式下，PM2为你的应用程序的每个实例创建一个单独的Node.js进程。每个进程独立运行，不与其他进程共享任何资源。这意味着，如果一个进程崩溃，它不会影响其他进程。在这个模式下，允许改变`exec_interpreter`（用来启动子进程的 "命令"），所以你可以用PM2运行一个PHP或[Python服务器](https://pm2.io/blog/2018/09/19/Manage-Python-Processes) 。默认情况下，PM2会使用`node`，所以`pm2 start server.js`类似于：
    ```
    require('child_process').spwn('node', ['server.js'])
    ```
    这种模式非常有用，因为它可以实现很多可能性。例如，你可以在预先确定的端口上启动多个服务器，然后由HAProxy或Nginx进行负载均衡。
- `cluster`只在`node`作为执行解释器的情况下工作，`cluster`模式使用内置的Node.js `cluster`模块来创建一个进程集群，这些进程都共享同一个服务器端口。这使Node.js能够利用多个CPU核心，并使需要处理大量并发请求的应用程序具有更好的性能和可扩展性。在`cluster`模式下，PM2创建一个主进程和几个工作进程。主进程管理工作进程，并将传入的请求分配给它们。
- 一般来说，`fork`模式适用于中低流量的简单应用，而`cluster`模式更适合于需要更好性能和可扩展性的高流量应用。然而，需要注意的是，`cluster`模式比`fork`模式需要更多的内存和CPU资源，所以它可能不适合所有的应用。

要切换模式，在部署应用程序时使用`--exec-mode`参数：

```bash
pm2 start app.js --exec-mode=cluster
```

或在`ecosystem.config.js`文件中设置：

```js
module.exports = {
  apps : [{
    exec_mode: "cluster"
  }]
}
```

- [PM2 Documentation](https://pm2.io/docs/plus/overview/)
- [Ecosystem file](https://pm2.keymetrics.io/docs/usage/application-declaration/)
- [PM2 Setup and Deployment with Ecosystem Configuration](https://pm2.io/blog/2018/05/31/PM2-Setup-and-Deployment-with-Ecosystem-Configuration)
- [PM2 Blog](https://pm2.io/blog)
- [Cluster and Fork mode difference in PM2](https://stackoverflow.com/questions/34682035/cluster-and-fork-mode-difference-in-pm2)