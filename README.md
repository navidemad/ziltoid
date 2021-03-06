# ziltoid

Cron based monitoring system.

There are many software applications that aim to watch processes, and keep them alive and clean. Some of them are well known :
 - god (http://godrb.com)
 - monit (http://mmonit.com/monit/)
 - bluepill (https://github.com/bluepill-rb/bluepill)
 - ... (https://www.ruby-toolbox.com/categories/server_monitoring)

All have good and bad sides.
One of the bad sides is that each alternative is based on a deamon that computes data and then sleeps for a while. Who is monitoring this particular deamon ? What if this process suddenly stops ?
Also, you often need root rights to run those tools. On some hosting environments (mainly in shared hosting), this is an issue.

Ziltoid is an attempt to solve those issues using the crontab system, which comes with many good sides :
 - it's on every system
 - it launches a task periodically then waits for an amount of time
 - it doesn't need monitoring
 - it can send emails to warn of an error
 - and it can run any script.

## Usage

The principle of Ziltoid is to use the crontab system. You need to create a script that monitors your processes and then configure it in your crontab to run as frequently as possible (every minute) :

    * * * * * /bin/bash -l -c 'ruby /path_to_ziltoid.rb'

Ziltoid is here to help you build this script.

### Process

It encapsulates a process and everything Ziltoid might need to monitor it :
 - start, stop en restart commands
 - pid file path
 - RAM and CPU limits
 
You can define one like so :

```ruby
    Ziltoid::Process.new("Lighty", {
      :pid_file => "/home/ror/http/tmp/lighttpd.pid",
      :commands => {
        :start => "/home/ror/bin/lighty start",
        :stop => "/home/ror/bin/lighty stop"
      },
      :limit => {
        :ram => 256,
        :cpu => 10
      }
    })
```

Processes are seen as `watchables` by Ziltoid. You can build your own watchable objects and pass them to Ziltoid, as long you can call the following methods on them :
 - `start`
 - `stop`
 - `restart`
 - `watch`

### Email Notifier

This is a wrapper for email notifications. Every log message of a level higher than Logger::INFO will be sent by email.

You can configure a mail like so :

```ruby
    Ziltoid::EmailNotifier.new(
      :via_options => {
        :address        => 'smtp.ziltoid.com',
        :port           => '25',
        :domain         => "ziltoid.com"
      },
      :subject => "[Balloonz] Ziltoid message",
      :to => ['developers@sociabliz.com'],
      :from => 'ziltoidd@ziltoid.com'
    )
```

The `via_options` accepts all options supported by the [pony](https://github.com/benprew/pony) library.

You can build your own notifiers (for IRC, XMPP, BaseCamp, Redmine, ....) as long as it responds to the `send` method. This method receives a `message` parameter which is the message to be sent.

### Watcher

This is the main object of Ziltoid. You can think of it as a container. With this container, you plug in a Logger, a Notifier and as many watchables as you want. You then run the commands of your choice.

Commands are :
 - `start`: to start watchable items
 - `stop`: to stop watchable items
 - `restart`: to restart watchable items
 - `watch`: to watch watchable items. IE start them if dead, restart them if of limits

You can see a sample Ziltoid script in the `examples` folder.

## Drawbacks

Ziltoid is not yet complete. We are currently working on a `grace time` system, where we don't check the RAM and CPU usages at the start of a process, and where we wait for a certain number of off-limit measurements to restart a process (and allow burst).

One of the bad side of the crontab system is that it cannot run more frequently than every minute. However, we think that if our system can't handle a dead process for a minute, there must be an issue somewhere (in replication, sizing, ...).

## About Ziltoid

It comes from Devin Townsen famous album "Ziltoid the omniscient".
Ziltoid is omniscient, watches everything and controls everything. Even processes.

Ziltoid is given to you by [The Social Client](http://www.thesocialclient.com), a digital and CRM consultant agency.

## Licencing

Ziltoid is a [Beerware](http://en.wikipedia.org/wiki/Beerware) and is licensed under the terms of the [WTFPL](http://www.wtfpl.net), see the included WTFPL-LICENSE file.

If we meet some day, and you think this stuff is worth it, you can buy us a beer in return.