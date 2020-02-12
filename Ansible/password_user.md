Ansible Password User

Python passlib option is more cross platform compatible and easier to manage in scripts.
``
hosts: main
vars:
``
created with:
``
python -c “from passlib.hash import sha512_crypt; print sha512_crypt.encrypt(‘’)”
``
above command requires the PassLib library: sudo pip install passlib
``
password: ‘$6$rounds=100000$H/83rErWaObIruDw$DEX.DgAuZuuF.wOyCjGHnVqIetVt3qRDnTUvLJHBFKdYr29uVYbfXJeHg.IacaEQ08WaHo9xCsJQgfgZjqGZI0’
tasks:
user: name=spree password={{password}} groups=sudo,www-data shell=/bin/bash append=yes
sudo: yes
``
or
``
user:
pass: secret
name: fake
``
encrypt your secrets file with :
``
ansible-vault encrypt /path/to/credential.yml
``
Vault key usage:
via passing argument when running playbook.
—ask-vault-pass: secret
or you can save into file like password.txt and hide somewhere. (useful for CI users)
—vault-password-file=/path/to/file.txt
In your case : include vars yml and use your variables.
``
include_vars: /path/credential.yml
name: Add deployment user
action: user name={{user.name}} password={{user.pass}}
``
