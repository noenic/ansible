# ATELIER-07

## Initialisation de l'atelier
Après un `vagrant up`, on se connecte à la machine `ansible`

```console
[noenic@mint-XPS:~/formation-ansible/atelier-07] # vagrant ssh ansible

This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento

Use of this system is acceptance of the OS vendor EULA and License Agreements.
[vagrant@ansible ~]$ cd ansible/projets/ema
```

## Petit test pour s'assurer que tout fonctionne

```console
[vagrant@ansible ema]$ ansible rocky -m user -a "name=greg shell=/bin/bash"
rocky | CHANGED => {
    "changed": true,
    "comment": "",
    "create_home": true,
    "group": 1001,
    "home": "/home/greg",
    "name": "greg",
    "shell": "/bin/bash",
    "state": "present",
    "system": false,
    "uid": 1001
}

[vagrant@ansible ema]$ ansible rocky -m user -a "name=greg shell=/bin/bash"
rocky | SUCCESS => {
    "append": false,
    "changed": false,
    "comment": "",
    "group": 1001,
    "home": "/home/greg",
    "move_home": false,
    "name": "greg",
    "shell": "/bin/bash",
    "state": "present",
    "uid": 1001
}
```

## Installation des packages
```console
[vagrant@ansible ema]$ ansible all -m package -a "name=tree,git,nmap state=present"
.....

[vagrant@ansible ema]$ ansible all -m package -a "name=tree,git,nmap state=present"
debian | SUCCESS => {
    "cache_update_time": 1761245363,
    "cache_updated": false,
    "changed": false
}
suse | SUCCESS => {
    "changed": false,
    "name": [
        "tree",
        "git",
        "nmap"
    ],
    "rc": 0,
    "state": "present",
    "update_cache": false
}
rocky | SUCCESS => {
    "changed": false,
    "msg": "Nothing to do",
    "rc": 0,
    "results": []
}
```
On voit que les logs sont différents selon les distributions, mais en relançant la même commande, on voit que les modules sont idempotents, c'est à dire qu'ils ne font rien si l'état désiré est déjà atteint.

## Désinstallation des packages
```console
[vagrant@ansible ema]$ ansible all -m package -a "name=tree,git,nmap state=absent"
.....
[vagrant@ansible ema]$ ansible all -m package -a "name=tree,git,nmap state=absent"
debian | SUCCESS => {
    "changed": false
}
suse | SUCCESS => {
    "changed": false,
    "name": [
        "tree",
        "git",
        "nmap"
    ],
    "rc": 0,
    "state": "absent",
    "update_cache": false
}
rocky | SUCCESS => {
    "changed": false,
    "msg": "Nothing to do",
    "rc": 0,
    "results": []
}
```

## Copie des fichiers fstab
```console
ansible all -m copy -a "src=/etc/fstab dest=/tmp/test3.txt"

debian | CHANGED => {
    "changed": true,
    "checksum": "0fe1d6fcaf1695fb3ef9d8a42d45d04e5e0c11c2",
    "dest": "/tmp/test3.txt",
    "gid": 0,
    "group": "root",
    "md5sum": "31623c38118cbe8247061a6bd3f239a4",
    "mode": "0644",
    "owner": "root",
    "size": 679,
    "src": "/home/vagrant/.ansible/tmp/ansible-tmp-1774349135.1299706-6391-111132752420510/source",
    "state": "file",
    "uid": 0
}
suse | CHANGED => {
    "changed": true,
    "checksum": "0fe1d6fcaf1695fb3ef9d8a42d45d04e5e0c11c2",
    "dest": "/tmp/test3.txt",
    "gid": 0,
    "group": "root",
    "md5sum": "31623c38118cbe8247061a6bd3f239a4",
    "mode": "0644",
    "owner": "root",
    "size": 679,
    "src": "/home/vagrant/.ansible/tmp/ansible-tmp-1774349135.2212005-6392-111507708812783/source",
    "state": "file",
    "uid": 0
}
rocky | CHANGED => {
    "changed": true,
    "checksum": "0fe1d6fcaf1695fb3ef9d8a42d45d04e5e0c11c2",
    "dest": "/tmp/test3.txt",
    "gid": 0,
    "group": "root",
    "md5sum": "31623c38118cbe8247061a6bd3f239a4",
    "mode": "0644",
    "owner": "root",
    "secontext": "unconfined_u:object_r:user_home_t:s0",
    "size": 679,
    "src": "/home/vagrant/.ansible/tmp/ansible-tmp-1774349135.2533913-6390-197481327493730/source",
    "state": "file",
    "uid": 0
}


[vagrant@ansible ema]$ ansible all -m copy -a "src=/etc/fstab dest=/tmp/test3.txt"
debian | SUCCESS => {
    "changed": false,
    "checksum": "0fe1d6fcaf1695fb3ef9d8a42d45d04e5e0c11c2",
    "dest": "/tmp/test3.txt",
    "gid": 0,
    "group": "root",
    "mode": "0644",
    "owner": "root",
    "path": "/tmp/test3.txt",
    "size": 679,
    "state": "file",
    "uid": 0
}
suse | SUCCESS => {
    "changed": false,
    "checksum": "0fe1d6fcaf1695fb3ef9d8a42d45d04e5e0c11c2",
    "dest": "/tmp/test3.txt",
    "gid": 0,
    "group": "root",
    "mode": "0644",
    "owner": "root",
    "path": "/tmp/test3.txt",
    "size": 679,
    "state": "file",
    "uid": 0
}
rocky | SUCCESS => {
    "changed": false,
    "checksum": "0fe1d6fcaf1695fb3ef9d8a42d45d04e5e0c11c2",
    "dest": "/tmp/test3.txt",
    "gid": 0,
    "group": "root",
    "mode": "0644",
    "owner": "root",
    "path": "/tmp/test3.txt",
    "secontext": "unconfined_u:object_r:user_home_t:s0",
    "size": 679,
    "state": "file",
    "uid": 0
}
```


## Suppression des fichiers
```console
ansible all -m file -a "path=/tmp/test3.txt state=absent"
debian | CHANGED => {
    "changed": true,
    "path": "/tmp/test3.txt",
    "state": "absent"
}
rocky | CHANGED => {
    "changed": true,
    "path": "/tmp/test3.txt",
    "state": "absent"
}
suse | CHANGED => {
    "changed": true,
    "path": "/tmp/test3.txt",
    "state": "absent"
}

[vagrant@ansible ema]$ ansible all -m file -a "path=/tmp/test3.txt state=absent"
debian | SUCCESS => {
    "changed": false,
    "path": "/tmp/test3.txt",
    "state": "absent"
}
rocky | SUCCESS => {
    "changed": false,
    "path": "/tmp/test3.txt",
    "state": "absent"
}
suse | SUCCESS => {
    "changed": false,
    "path": "/tmp/test3.txt",
    "state": "absent"
}
```

## Espace disque
```console
[vagrant@ansible ema]$ ansible all -a "df -h /"
debian | CHANGED | rc=0 >>
Filesystem                       Size  Used Avail Use% Mounted on
/dev/mapper/debian--12--vg-root   62G  1.7G   57G   3% /
suse | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3        64G  2.4G   58G   4% /
rocky | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        61G  2.1G   59G   4% /
```

On remarque que le résultat est toujours "CHANGED" même si la commande ne fait rien sur le système, c'est parce que les modules `command` et `shell` ne sont pas idempotents, ils ne peuvent pas vérifier l'état d'une ressource, donc ils affichent toujours "CHANGED" même si la commande ne fait rien, C'est pour cela qu'on préfère toujours utiliser un module spécifique (comme user ou package) plutôt qu'une commande brute quand c'est possible.
