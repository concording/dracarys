## [SSH Tunnel - Local and Remote Port Forwarding Explained With Examples](https://blog.trackets.com/2014/05/17/ssh-tunnel-local-and-remote-port-forwarding-explained-with-examples.html)

There are two ways to create an SSH tunnel, local and remote port forwarding (there’s also dynamic forwarding, but we won’t cover that here). The best way to understand these is by an example, let’s start with local port forwarding.

Imagine you’re on a private network which doesn’t allow connections to a specific server. Let’s say you’re at work and imgur.com is being blocked. To get around this we can create a tunnel through a server which isn’t on our network and thus can access Imgur.

```
$ ssh -L 9000:imgur.com:80 user@example.com

```

The key here is `-L` which says we’re doing local port forwarding. Then it says we’re forwarding our local port `9000` to `imgur.com:80`, which is the default port for HTTP. Now open your browser and go to [http://localhost:9000](http://localhost:9000/).

The awesome thing about SSH tunnels is that they are encrypted. Nobody is going to see what sites you’re visiting, they’ll only see an SSH connection to your server.

## Connecting to a database behind a firewall

Another good example is if you need to access a port on your server which can only be accessed from `localhost` and not remotely.

An example here is when you need to connect to a database console, which only allows local connection for security reasons. Let’s say you’re running PostgreSQL on your server, which by default listens on the port `5432`.

```
$ ssh -L 9000:localhost:5432 user@example.com

```

The part that changed here is the `localhost:5432`, which says to forward connections from your local port `9000` to `localhost:5432` on your server. Now we can simply connect to our database.

```
$ psql -h localhost -p 9000

```

Now let’s stop here for a little bit an explain what is actually going on. In the first example the `9000:imgur.com:80` is actually saying `forward my local port 9000 to imgur.com at port 80`. You can imagine SSH on your server actually making a connection (a tunnel) between those two ports, one on your local machine, and one on the target destination.

If we however say something like `9000:localhost:5432`, it means `localhost` from the server’s perspective, not localhost on your machine. This means `forward my local port 9000 to port 5432 on the server`, because when you’re on the server, `localhost` means the server itself.

This might be a bit confusing, but it is important to understand what the syntax actually means here.

## Remote port forwarding

Now comes the second part of this tutorial, which is remote port forwarding. This is again best to explain with an example.

Say that you’re developing a Rails application on your local machine, and you’d like to show it to a friend. Unfortunately your ISP didn’t provide you with a public IP address, so it’s not possible to connect to your machine directly via the internet.

Sometimes this can be solved by configuring NAT (Network Address Translation) on your router, but this doesn’t always work, and it requires you to change the configuration on your router, which isn’t always desirable. This solution also doesn’t work when you don’t have admin access on your network.

To fix this problem you need to have another computer, which is publicly accessible and have SSH access to it. It can be any server on the internet, as long as you can connect to it. We’ll tell SSH to make a tunnel that opens up a new port on the server, and connects it to a local port on your machine.

```
$ ssh -R 9000:localhost:3000 user@example.com

```

The syntax here is very similar to local port forwarding, with a single change of `-L` for `-R`. But as with local port forwarding, the syntax remains the same.

First you need to specify the port on which th remote server will listen, which in this case is `9000`, and next follows `localhost` for your local machine, and the local port, which in this case is `3000`.

There is one more thing you need to do to enable this. SSH doesn’t by default allow remote hosts to forwarded ports. To enable this open `/etc/ssh/sshd_config` and add the following line somewhere in that config file.

```
GatewayPorts yes

```

Make sure you add it only once!

```
$ sudo vim /etc/ssh/sshd_config

```

And restart SSH

```
$ sudo service ssh restart

```

After this you should be able to connect to the server remotely, even from your local machine. The way this would work is that you would first create an SSH tunnel that forwards traffic from the server on port `9000` to your local machine on port `3000`. This means that if you connect to the server on port `9000` from your local machine, you’ll actually make a request to your machine through the SSH tunnel.

## A few closing tips

You might have noticed that every time we create a tunnel you also SSH into the server and get a shell. This isn’t usually necessary, as you’re just trying to create a tunnel. To avoid this we can run SSH with the `-nNT` flags, such as the following, which will cause SSH to not allocate a tty and only do the port forwarding.

```
$ ssh -nNT -L 9000:imgur.com:80 user@example.com

```

SSH has a huge number of features, so I’d recommend you to checkout the manual page at `man ssh`, which contains even more tips.

There’s also an amazing talk called [The Black Magic of SSH / SSH Can Do That?](http://vimeo.com/54505525), which I really recommend you to watch.

* * *

