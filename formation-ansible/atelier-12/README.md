# ATELIER-12

## Initialisation de l'atelier

Démarrage de l'environnement et connexion au `control` Host et on vérifie que tout fonctionne avec un ping sur tous les hôtes :

```console
[vagrant@control ema]$ ansible -m ping all
target02 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
target01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
target03 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Playbook chrony 

L'objectif est d'installer chrony, de gérer son service et de ne redémarrer le service que si le fichier `/etc/chrony.conf` est modifié.

```yaml
---
- hosts: redhat
  tasks:
    - name: Install Chrony
      dnf:
        name: chrony
        state: present

    - name: Start and Enable Chrony service
      service:
        name: chronyd
        state: started
        enabled: true

    - name: Install custom configuration
      copy:
        dest: /etc/chrony.conf
        content: |
          # /etc/chrony.conf
          server 0.fr.pool.ntp.org iburst
          server 1.fr.pool.ntp.org iburst
          server 2.fr.pool.ntp.org iburst
          server 3.fr.pool.ntp.org iburst
          driftfile /var/lib/chrony/drift
          makestep 1.0 3
          rtcsync
          logdir /var/log/chrony
      notify: Restart Chrony

  handlers:
    - name: Restart Chrony
      service:
        name: chronyd
        state: restarted
...
```

On peut vérifier la syntaxe avec yaml-lint vu qu'il est déjà installé 

```console
[vagrant@control ema]$ yamllint -s playbooks/chrony.yaml 
[vagrant@control ema]$ echo $?
0
```


```console
[vagrant@control playbooks]$ ansible-playbook chrony.yaml 

PLAY [redhat] ******************************************************************************

TASK [Gathering Facts] *********************************************************************
ok: [target03]
ok: [target01]
ok: [target02]

TASK [Install Chrony] **********************************************************************
changed: [target03]
changed: [target02]
changed: [target01]

TASK [Start and Enable Chrony service] *****************************************************
changed: [target03]
changed: [target02]
changed: [target01]

TASK [Install custom configuration] ********************************************************
changed: [target03]
changed: [target01]
changed: [target02]

RUNNING HANDLER [Restart Chrony] ***********************************************************
changed: [target01]
changed: [target03]
changed: [target02]

PLAY RECAP *********************************************************************************
target01                   : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

En relançant le playbook, on voit que les tâches sont idempotentes et que le handler n'est pas déclenché car le fichier de configuration n'est pas modifié.

```console
[vagrant@control ema]$ ansible-playbook playbooks/chrony.yaml 

PLAY [redhat] ******************************************************************************

TASK [Gathering Facts] *********************************************************************
ok: [target03]
ok: [target01]
ok: [target02]

TASK [Install Chrony] **********************************************************************
ok: [target03]
ok: [target02]
ok: [target01]

TASK [Start and Enable Chrony service] *****************************************************
ok: [target02]
ok: [target03]
ok: [target01]

TASK [Install custom configuration] ********************************************************
ok: [target02]
ok: [target03]
ok: [target01]

PLAY RECAP *********************************************************************************
target01                   : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Mais si on modifie le contenu du fichier de configuration, on voit que le handler est déclenché et que le service est redémarré.

Disons que pour l'exemple, on ajoute une ligne dans le fichier de configuration pour changer le serveur NTP :

```yaml
server 0.fr.pool.ntp.org iburst
server 1.fr.pool.ntp.org iburst
server 2.fr.pool.ntp.org iburst
server 3.fr.pool.ntp.org iburst
server time.google.com iburst
```

```console
[vagrant@control ema]$ ansible-playbook playbooks/chrony.yaml 

PLAY [redhat] ******************************************************************************

TASK [Gathering Facts] *********************************************************************
ok: [target01]
ok: [target03]
ok: [target02]

TASK [Install Chrony] **********************************************************************
ok: [target03]
ok: [target02]
ok: [target01]

TASK [Start and Enable Chrony service] *****************************************************
ok: [target03]
ok: [target01]
ok: [target02]

TASK [Install custom configuration] ********************************************************
changed: [target01]
changed: [target02]
changed: [target03]

RUNNING HANDLER [Restart Chrony] ***********************************************************
changed: [target01]
changed: [target03]
changed: [target02]

PLAY RECAP *********************************************************************************
target01                   : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Le service est redémarré car le fichier de configuration a été modifié, et le module `copy` a détecté ce changement et a déclenché le handler pour redémarrer le service.



