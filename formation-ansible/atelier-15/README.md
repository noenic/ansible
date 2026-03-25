# ATELIER-15 : Variables Enregistrées (Register)

## Initialisation de l'atelier
Démarrage de l'environnement et connexion au `ansible` Host et on vérifie que tout fonctionne avec un ping sur tous les hôtes :

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

## Playbook pour les informations noyau
L'objectif est de récupérer les informations du noyau de chaque hôte et de les afficher à l'aide du module `debug`.

Pour simplifier la lecture, on a mis directement les deux manières d'afficher les résultats, avec `msg` et avec `var`. La première affiche une simple chaîne de caractères tandis que la seconde affiche une structure plus détaillée.

```yaml
---
- hosts: all
  gather_facts: false
  tasks:
    - name: Get kernel info
      command: uname -a
      register: kernel_res
      changed_when: false

    - name: Display with msg
      debug:
        msg: "Le noyau est : {{ kernel_res.stdout }}"

    - name: Display with var
      debug:
        var: kernel_res.stdout_lines
...
```

```console
[vagrant@ansible ema]$ ansible-playbook playbooks/kernel.yml

PLAY [all] *********************************************************************************

TASK [Get kernel info] *********************************************************************
ok: [suse]
ok: [rocky]
ok: [debian]

TASK [Display with msg] ********************************************************************
ok: [rocky] => {
    "msg": "Le noyau est : Linux rocky 5.14.0-570.52.1.el9_6.x86_64 #1 SMP PREEMPT_DYNAMIC Wed Oct 15 13:59:22 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux"
}
ok: [suse] => {
    "msg": "Le noyau est : Linux suse 6.4.0-150600.23.73-default #1 SMP PREEMPT_DYNAMIC Tue Oct  7 08:43:02 UTC 2025 (46f6a23) x86_64 x86_64 x86_64 GNU/Linux"
}
ok: [debian] => {
    "msg": "Le noyau est : Linux debian 6.1.0-40-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.153-1 (2025-09-20) x86_64 GNU/Linux"
}

TASK [Display with var] ********************************************************************
ok: [rocky] => {
    "kernel_res.stdout_lines": [
        "Linux rocky 5.14.0-570.52.1.el9_6.x86_64 #1 SMP PREEMPT_DYNAMIC Wed Oct 15 13:59:22 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux"
    ]
}
ok: [suse] => {
    "kernel_res.stdout_lines": [
        "Linux suse 6.4.0-150600.23.73-default #1 SMP PREEMPT_DYNAMIC Tue Oct  7 08:43:02 UTC 2025 (46f6a23) x86_64 x86_64 x86_64 GNU/Linux"
    ]
}
ok: [debian] => {
    "kernel_res.stdout_lines": [
        "Linux debian 6.1.0-40-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.153-1 (2025-09-20) x86_64 GNU/Linux"
    ]
}

PLAY RECAP *********************************************************************************
debian                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rocky                      : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
suse                       : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

## Playbook pour le comptage de paquets RPM
L'objectif est de compter le nombre de paquets installés sur les hôtes `rocky`et `suse` et de les afficher.

```yaml
---
- hosts: rocky, suse
  gather_facts: false
  tasks:
    - name: Count RPM packages
      shell: rpm -qa | wc -l
      register: rpm_count
      changed_when: false

    - name: Display count
      debug:
        msg: "Nombre de paquets installés : {{ rpm_count.stdout }}"
...
```

```console
[vagrant@ansible ema]$ ansible-playbook playbooks/packages.yml

PLAY [rocky, suse] *************************************************************************

TASK [Count RPM packages] ******************************************************************
ok: [rocky]
ok: [suse]

TASK [Display count] ***********************************************************************
ok: [rocky] => {
    "msg": "Nombre de paquets installés : 642"
}
ok: [suse] => {
    "msg": "Nombre de paquets installés : 504"
}

PLAY RECAP *********************************************************************************
rocky                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
suse                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

