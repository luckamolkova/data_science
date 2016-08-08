# AWS

## EC2

- Create new EC2 instance and note the connection details
- Add shortcut to your `~/.ssh/config` file to be able to access it easily:

```
$ subl ~/.ssh/config
```

- This is what goes into the `~/.ssh/config` file:

```
 Host ec2
  HostName 52.32.24.5
  user ubuntu
  IdentityFile ~/.ssh/capstone.pem
```

- SSH to your instance and install basic things:

```
$ ssh ec2 -t "tmux -CC attach"
ubuntu$ sudo apt-get update
ubuntu$ sudo apt-get install git
ubuntu$ wget http://repo.continuum.io/archive/Anaconda2-4.1.1-Linux-x86_64.sh
ubuntu$ bash Anaconda2-4.1.1-Linux-x86_64.sh
```

- Install specific python packages you need:

```
$ pip install pathos
```

- Clone your git repo:

```
$ git clone <repo>
```


- Optionally it might be worth installing nuclide server followwing the instructions here: https://nuclide.io/docs/features/remote/#nuclide-server. This will allow you to edit the code from Atom on your laptop and avoid vi!

## PostgreSQL

- Install PostgreSQL on the ec2 instance

```
ubuntu$ apt-get install postgresql
ubuntu$ sudo apt-get install libpq-dev
```

- Install specific packages needed

```
ubuntu$ pip install psycopg2
```

- Create user - you will be prompted for pasword:

```
ubuntu$ sudo su -postgres
postgres$ createuser --password ubuntu
```

- Create empty database

```
ubuntu$ sudo su -postgres
postgres$ psql
postgres=# CREATE DATABASE bugs;
postgres=# \q
# Ctrl+D to exit
```

**Move database from your laptop to EC2 instance**

- Create backp (dump):

```
$ pg_dump bugs > bugs3.sql
```

- Copy database dump to EC2 instance using SCP:

```
$ scp bugs3.sql ec2:/home/ubuntu
```

- From the EC2 instance load the dump:

```
ubuntu$ psql bugs < bugs3.sql
```

## Flask

- Make sure the app.py has debug False and threaded True:

```
app.run(host='0.0.0.0', port=5353, debug=False, threaded=True)
```

- AWS console > EC2 Security Groups: Add Inbound rule

```
HTTP 80
```

- Redirect port 80 to 5353 (the app port):

``` 
ubuntu$ sudo iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 5353
ubuntu$ sudo apt-get install  iptables-persistent
```

- AWS console > EC2: Elastic IPs

```
Create new
Actions > Associate with the instance
```

- At this point the instance will get this IP address - you may need to change it in your `~/.ssh/config` and reconnect as your connection will be lost.
- You can now access your app from a browser by typing the Elastic IP address.