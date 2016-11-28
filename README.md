#Setup git deploy for AWS ec2 Ubuntu instance

The following instructions are for setting up git deployment on an AWS ec2 ubuntu instance (or any ubuntu server for that matter). Also included are instructions for deploying to the remote server and github simultaneously.

##Git deploy setup:

1 - copy your public key to your ec2 instance:

```
cat ~/.ssh/id_rsa.pub | ssh -i ~/.ssh/your_pemfile.pem ubuntu@your_ip_addr "cat>> .ssh/authorized_keys"
```

2 - on remote server: create bare git directory
```
$ cd ~
$ mkdir ProjectDir.git && cd ProjectDir.git
$ git init --bare
```

3 - on remote server: create post-receive hook

```
$ cat > hooks/post-receive
#!/bin/sh
export GIT_WORK_TREE=/var/www
git checkout -f
sudo chown -Rf ubuntu:ubuntu /var/www
cd /var/www

if [ ! -f .env ]; then
	mv .env.example .env
	composer install
	./artisan key:generate
	composer exec "Illuminate\\Foundation\\ComposerScripts::postInstall"
else
	composer update
	composer exec "Illuminate\\Foundation\\ComposerScripts::postUpdate"
fi

chmod -Rf +w storage bootstrap/cache

./artisan optimize
./artisan migrate
./artisan route:clear
./artisan route:cache
```

After create post-receive file, make him as executable.

```
$ chmod +x hooks/post-receive
```

4 - on local machine: init repo and add remote repository

```
git init
git remote add ec2 ssh://ubuntu@your_ip_addr/home/ubuntu/ProjectDir.git
git push ec2 +master:refs/heads/master
```

`note:` only have to use “+master:refs/heads/master for 1st push

##To push to remote repo in future:
```
$ git push ec2 master
```
Push to multiple remote repos with one command

1 - add to .git/config in local repo

```
[remote "all"]
url = https://github.com/YourGitAccount/ProjectDir.git
url = ssh://ubuntu@your_ip_addr/home/ubuntu/projects/ProjectDir.git
```

2 - push to both repos simultaneously

```
$ git push all master
```
