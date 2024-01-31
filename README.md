# Stubby on Raspberry Pi

This is a tutorial on how to build and install [Stubby](https://dnsprivacy.org/dns_privacy_daemon_-_stubby/) on a Raspberry Pi.

It took me a while to figure it out, so I thought I'd share it here. Some of these commands require root privileges, so you might want to run them with `sudo`.

## Build

### Clone source code

First, clone the source code from GitHub into `/usr/src`:

```sh
cd /usr/src
git clone https://github.com/getdnsapi/stubby.git
cd stubby
git checkout master
```

Make sure you're on the `master` branch.

### Install dependencies

Install the dependencies listed in the [Dockerfile](https://github.com/getdnsapi/stubby/blob/develop/contrib/docker/Dockerfile) under `# Build stage.`. At the time of writing, the following packages were required:

```sh
apt-get update && apt-get -y install autoconf build-essential ca-certificates cmake libgetdns-dev libidn2-0-dev libssl-dev libunbound-dev libyaml-dev
```

### Build stubby

Run the following commands to build stubby:

```sh
cmake .
make
```

## Install

### Install runtime dependencies

Install the dependencies listed in the [Dockerfile](https://github.com/getdnsapi/stubby/blob/develop/contrib/docker/Dockerfile) under `# Final image.`. At the time of writing, the following packages were required:

```sh
apt-get update && apt-get -y install ca-certificates libgetdns10 libidn2-0 libunbound8 libyaml-0-2 openssl
```

### Install stubby

To install stubby, we'll copy the binary to `/usr/bin`. You can probably also just create a symlink, but I haven't tried that.

```sh
cp stubby /usr/bin
```

If you prefer to install for your user only, you can try to copy it to `/usr/local/bin` instead (haven't tried that either).

### Install configuration file

```sh
cp stubby.yml.example /usr/local/etc/stubby/stubby.yml
```

Initially I tried to use `/etc/stubby/stubby.yml`, but stubby wouldn't find the configuration file on startup. There's no option to customize that path in the service file either. If anyone knows why, please let me know.

At this point open the configuration file with a text editor like `nano` and make sure it's configured to your liking. In my case I changed to listening port to 5353 and set my upstream DNS servers to Quad9.

Note: In my case I'm using stubby with Pi-hole so I don't care about external access. But, if you want to call stubby from another device, I think you have to add `0.0.0.0` to the `listen_addresses` list. I tried to run `dig` from my computer and it just wouldn't reach stubby (would say "connection timed out").

### Install service file

```sh
cp systemd/stubby.service /lib/systemd/system/stubby.service
systemctl enable stubby.service
```

Again if you want to use symlinks I'm pretty sure you can.

In my case, I had to comment out the lines `User=...` and `DynamicUser=...` because stubby would fail with exit code 1 otherwise. If someone figures it out, please let me know.

### Test

Reboot your Raspberry Pi and run `dig` to test if stubby is working:

```sh
dig @127.0.0.1 -p 5353 example.com
```

Run this command to check what protocol you're using:

```sh
dig +short txt proto.on.quad9.net.
```

For DNS-over-TLS it should say `dot`.
