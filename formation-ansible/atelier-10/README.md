[🏠 Sommaire ](../README.md)
# ATELIER-10 : Un serveur web

## Initialisation de l'atelier

On démarre les VM avec `vagrant up` et on se connecte au `control` host. Grâce à direnv, la configuration Ansible est automatiquement chargée.

Pour s'assurer que tout fonctionne, on fait un ping sur tous les hôtes :

```console
[vagrant@ansible ema]$ ansible -m ping all
rocky | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
suse | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
debian | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Déploiement de Debian 

On crée notre premier playbook `apache-debian.yml` pour installer le serveur web Apache avec une page d'accueil personnalisée.

> [!WARNING]
> Vu que c'est pas les mêmes nom de package et de service entre les distributions, on va pas utiliser le module `package` mais les modules spécifiques à chaque distribution.

```yaml
---
- hosts: debian
  tasks:
    - name: Update apt cache
      apt:
        update_cache: true
        cache_valid_time: 3600

    - name: Install Apache
      apt:
        name: apache2
        state: present

    - name: Start Apache
      service:
        name: apache2
        state: started
        enabled: true

    - name: Custom page
      copy:
        dest: /var/www/html/index.html
        content: "Apache web server running on Debian Linux\n"
...
```
> [!TIP]
> Mention honorable pour les `"---"` et `"..."` qui sont pas necessaires en temps normal mais qui après 8965 heures à debuguer des dry-run kubernetes c'est devenu une habitude d'entreprise.

```console
[vagrant@ansible playbooks]$ ansible-playbook apache-debian.yml

PLAY [debian] ******************************************************************************

TASK [Gathering Facts] *********************************************************************
ok: [debian]

TASK [Update apt cache] ********************************************************************
changed: [debian]

TASK [Install Apache] **********************************************************************
changed: [debian]

TASK [Start Apache] ************************************************************************
changed: [debian]

TASK [Custom page] *************************************************************************
changed: [debian]

PLAY RECAP *********************************************************************************
debian                     : ok=4    changed=4    unreachable=0    failed=0

[vagrant@ansible playbooks]$ curl debian
Apache web server running on Debian Linux
```


## Déploiement de Rocky Linux

```yaml
---
- hosts: rocky
  tasks:
    - name: Install Apache
      dnf:
        name: httpd
        state: present

    - name: Start Apache
      service:
        name: httpd
        state: started
        enabled: true

    - name: Custom page
      copy:
        dest: /var/www/html/index.html
        content: "Apache web server running on Rocky Linux\n"
...
```
```console
[vagrant@ansible playbooks]$ ansible-playbook apache-debian.yml


[vagrant@ansible playbooks]$ ansible-playbook apache-rocky.yml 

PLAY [rocky] *******************************************************************************

TASK [Gathering Facts] *********************************************************************
ok: [rocky]

TASK [Install Apache] **********************************************************************
changed: [rocky]

TASK [Start Apache] ************************************************************************
changed: [rocky]

TASK [Custom page] *************************************************************************
changed: [rocky]

PLAY RECAP *********************************************************************************
rocky                      : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[vagrant@ansible playbooks]$ curl rocky 
Apache web server running on Rocky Linux
```

## Déploiement de SUSE

```yaml
---
- hosts: suse
  tasks:
    - name: Install Apache
      zypper:
        name: apache2
        state: present

    - name: Start Apache
      service:
        name: apache2
        state: started
        enabled: true

    - name: Custom page
      copy:
        dest: /srv/www/htdocs/index.html
        content: "Apache web server running on SUSE Linux\n"
...
```
```console
[vagrant@ansible playbooks]$ ansible-playbook apache-suse.yml
```console
[vagrant@ansible playbooks]$ ansible-playbook apache-suse.yml

PLAY [suse] *******************************************************************************

TASK [Gathering Facts] *********************************************************************
ok: [suse]

TASK [Install Apache] **********************************************************************
changed: [suse]

TASK [Start Apache] ************************************************************************
changed: [suse]

TASK [Custom page] *************************************************************************
changed: [suse]

PLAY RECAP *********************************************************************************
suse                       : ok=4    changed=3    unreachable=0    failed=0

[vagrant@ansible playbooks]$ curl suse
Apache web server running on SUSE Linux
```
