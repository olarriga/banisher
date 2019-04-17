# The Banisher

<p align="center">
  <img width="330" height="330" src="/etc/banisher.png">
</p>

The Banisher watch in real time your systemd journal and ban, via iptables, hosts based on yours rules.

Currently hosts (IP) are banished for 3 hours.

The Banisher keeps states of banished IPs in a key-value store ([badger](https://github.com/dgraph-io/badger))   


## Getting started

*WARNING The Banisher works onlys with logs handled by systemd journal.*

### Installing

Just download the lastest binary from the [releases section](https://github.com/toorop/banisher/releases)
 
### Rules

In the same directory than The Banisher binary, create a [YAML](https://en.wikipedia.org/wiki/YAML) file named `rules.yml`.
This file will contain your Banisher rules.

A rule has three poperties:
- *name*: is the name of the rule (whaoo amazing!)
- *match*: is a regular expression. If a log line matches this regex, The Banisher will ban IP address found in this line.
- *IPpos*: as some log line may have multiple IP, this properties will indicate which IP to ban. *Warning*: index start at 0, so if you want to ban the first IP found (left to right) IPpso must be 0.

And... that it.

Here is some samples:

##### SSH

A failed auth attempt, appears in log with this line:

```text
Failed password for invalid user mrpresidentmanu from XXX.XXX.XXX.XXX port 47092 ssh2
```

Here is the corresponding rule:

```yaml
- name: ssh
  match: Failed password.*ssh2
  IPpos: 0
```

##### Dovecot IMAP

Log line for [Dovecot](https://www.dovecot.org/) authentification failure looks like:

```text
imap-login: Disconnected (auth failed, 1 attempts in 3 secs): user=<tobe@rnotto.be>, method=PLAIN, rip=XXX.XXX.XXX.XXX, lip=YYY.YYY.YYY.YYY, TLS: Disconnected, session=<n48ImrmGRP6xth/K>

``` 

Here is the corresponding rule:

```yaml
- name: dovecot-imap
  match: .*imap-login:.*auth failed,.*
  IPpos: 0
```

Yes i know it seems to be too easy to be real.

#### Multiple rules ?

Of course you can have multiple rule in your rules.ym, you just have to not forget the `-` prepending the `name` property.

For example if you want those two rules, your `rules.yml` will be:

```yaml

- name: ssh
  match: Failed password.*ssh2
  IPpos: 0

- name: dovecot-imap
  match: .*imap-login:.*auth failed,.*
  IPpos: 0
```  

### Launch 

You have downloaded the Banisher binary ?  
You have set the exec flag (`chmod +x banisher`) ?  
You have set up your rules ?

Let's go !

Just run:

```bash
./banisher
2019/04/17 16:19:12 dovecot: 183.82.32.153 banned
2019/04/17 16:19:12 ssh: 104.236.246.16 banned
2019/04/17 16:19:13 dovecot: 178.150.194.243 banned
2019/04/17 16:19:15 ssh: 51.77.213.181 banned
2019/04/17 16:19:20 ssh: 193.169.39.254 banned
2019/04/17 16:19:20 ssh: 82.200.65.218 banned
2019/04/17 16:19:21 ssh: 178.128.84.246 banned
2019/04/17 16:19:21 ssh: 190.145.55.89 banned
2019/04/17 16:19:21 ssh: 211.21.154.4 banned
```

Of course you can configure systemd to handle banisher binay (doc is comming)

### And what can i do if somthing goes wrong !!!

An iptables rules will be automacicaly removed after 3 hours.

If you made a mistake, just:

- stop The Banisher
- remove badger files, the db.bdg folder.
- flush iptables `ìptables -F`
- add your own iptables rules (if needed)   
