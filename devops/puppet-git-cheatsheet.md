# Puppet git cheatsheet



SETUP Setup a puppet directory on your filesystem. under this directory create 4 directories: production, qua, staging, and test in each directory initialize a git repository \(git init\) Then run the following commands:

```text
git remote add origin git@gitlab.stillwaterinsurance.com:puppet/puppet.git

git fetch

git checkout [branch name]
```

WORKFLOW

```text
git add .      - adds all files to git

git commit -m “Commit Message”

git push  - deploy to central repository


/_fetch upstream without overriding changes_/    NOTE: this will likely have conflicts which you will need to resolve
git stash    git pull 
git stash pop
```

CONVENTIONS naming-convention - folder should be application name - metadata should state: SWIG ———————————————— For Production modules layout should be - class/manifests/ ————————————————install.pp - ensures packages are installed ————————————————config.pp - configuration file logic ————————————————service.pp - ensures service is running ————————————————init.pp - initializes other classes ————————————————params.pp - logic to change parameters data should NOT be in your production manifests. Data is managed by the ENC \(External Node Classifier\). PUPPET COMMANDS

```text
puppet resource ?  {package, user} 
puppet apply -e (puppet syntax)  ————————————————noop ————————————————debug    -  lists commands to perform selected task
```

