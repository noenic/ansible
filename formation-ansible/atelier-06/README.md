# ATELIER-06

## Initialisation de l'atelier
Après un `vagrant up`, on se connecte à la machine `control`

```bash
[noenic@mint-XPS:~/formation-ansible/atelier-06] # vagrant ssh control


Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-160-generic x86_64)
vagrant@control:~$ 
```

## On ajoute les hôtes à /etc/hosts

```bash
vagrant@control:~$ sudo tee -a /etc/hosts << EOF
192.168.56.20 target1
192.168.56.30 target2
192.168.56.40 target3
EOF
```

## On configure ssh pour donner nos clés publiques aux cibles

```bash
vagrant@control:~$ ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ""
Generating public/private ed25519 key pair.....

vagrant@control:~$ ssh-keyscan -H target1 target2 target3 >> ~/.ssh/known_hosts


vagrant@control:~$ ssh-copy-id -i ~/.ssh/id_ed25519.pub target1
vagrant@control:~$ ssh-copy-id -i ~/.ssh/id_ed25519.pub target2
vagrant@control:~$ ssh-copy-id -i ~/.ssh/id_ed25519.pub target3
```

## On installe Ansible

```bash
vagrant@control:~$ sudo apt update && sudo apt install -y ansible
```

## On ping les cibles (sans configuration d'inventaire)

```bash
vagrant@control:~$ ansible all -i target1,target2,target3 -m ping

target2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
target1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
target3 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}   
```

## On crée une configuration
```bash
vagrant@control:~$ mkdir -p monprojet && cd monprojet
vagrant@control:~/monprojet$ touch ansible.cfg
vagrant@control:~/monprojet$ ansible --version | grep "config file"
  config file = /home/vagrant/monprojet/ansible.cfg
```
Le fichier de config sera pris en compte uniquement si on se trouve dans le dossier `monprojet`.

Donc on doit utiliser direnv pour charger la configuration d'ansible à chaque fois que l'on se trouve dans le dossier `monprojet`

```bash
vagrant@control:~/monprojet$ sudo apt install -y direnv
vagrant@control:~/monprojet$ echo 'eval "$(direnv hook bash)"' >> ~/.bashrc
vagrant@control:~/monprojet$ source ~/.bashrc
vagrant@control:~/monprojet$ echo 'export ANSIBLE_CONFIG=$(expand_path ansible.cfg)' > .envrc
vagrant@control:~/monprojet$ direnv allow

# Maintenant, on a le bon fichier de configuration d'ansible qui est pris en compte à chaque fois que l'on se trouve dans le dossier monprojet, ou dans un de ses sous-dossiers
vagrant@control:~$ cd monprojet/
direnv: loading ~/monprojet/.envrc
direnv: export +ANSIBLE_CONFIG

vagrant@control:~/monprojet$ ansible --version | grep "config file"
  config file = /home/vagrant/monprojet/ansible.cfg

vagrant@control:~/monprojet$ cd test/
vagrant@control:~/monprojet/test$ ansible --version | grep "config file"
  config file = /home/vagrant/monprojet/ansible.cfg
```

## Création d'un inventaire

```bash
vagrant@control:~/monprojet$ mkdir ~/journal
cat > ansible.cfg <<EOF
[defaults]
inventory = ./hosts
log_path = ~/journal/ansible.log
EOF

cat > hosts <<EOF
[testlab]
target1
target2
target3

[testlab:vars]
ansible_user=vagrant
ansible_become=yes
ansible_python_interpreter=/usr/bin/python3
EOF
```

## On ping les cibles (avec configuration d'inventaire)

```bash
vagrant@control:~/monprojet$ ansible all -m ping
target3 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
target2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
target1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```
## Affichage de /etc/shadows pour tester l'élévation de privilèges

```bash
vagrant@control:~/monprojet$ ansible all -a "head -n 1 /etc/shadow"
target1 | CHANGED | rc=0 >>
root:*:19977:0:99999:7:::
target2 | CHANGED | rc=0 >>
root:*:19977:0:99999:7:::
target3 | CHANGED | rc=0 >>
root:*:19977:0:99999:7:::
```

## On affiche le journal d'exécution d'ansible

```bash
vagrant@control:~/monprojet$ cat ~/journal/ansible.log 
2026-03-24 08:43:04,467 p=3251 u=vagrant n=ansible | target3 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
2026-03-24 08:43:04,470 p=3251 u=vagrant n=ansible | target2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
2026-03-24 08:43:04,481 p=3251 u=vagrant n=ansible | target1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
2026-03-24 08:44:10,443 p=3313 u=vagrant n=ansible | target1 | CHANGED | rc=0 >>
root:*:19977:0:99999:7:::

2026-03-24 08:44:10,445 p=3313 u=vagrant n=ansible | target2 | CHANGED | rc=0 >>
root:*:19977:0:99999:7:::

2026-03-24 08:44:10,445 p=3313 u=vagrant n=ansible | target3 | CHANGED | rc=0 >>
root:*:19977:0:99999:7:::
```