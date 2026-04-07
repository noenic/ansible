[🏠 Sommaire ](/README.md)
# ATELIER-17 : Gestion des cibles hétérogènes (NTP/Chrony)

## Approche "Gros Sabots" (chrony-01.yml)

Ici, on multiplie les tâches avec des conditions when et on utilise les modules natifs (apt, dnf, zypper), c'est bourrin mais ça marche.

```yaml
# chrony-01.yml
---
- hosts: all
  tasks:
    - name: Install Chrony (Debian/Ubuntu)
      apt:
        name: chrony
        state: present
        update_cache: true
      when: ansible_os_family == "Debian"

    - name: Install Chrony (Rocky)
      dnf:
        name: chrony
        state: present
      when: ansible_distribution == "Rocky"

    - name: Install Chrony (SUSE)
      zypper:
        name: chrony
        state: present
      when: ansible_distribution == "openSUSE Leap"

    - name: Install custom configuration
      copy:
        dest: /etc/chrony.conf
        content: |
          server 0.fr.pool.ntp.org iburst
          server 1.fr.pool.ntp.org iburst
          server 2.fr.pool.ntp.org iburst
          server 3.fr.pool.ntp.org iburst
          driftfile /var/lib/chrony/drift
          makestep 1.0 3
          rtcsync
          logdir /var/log/chrony
      notify: "Restart Chrony Service"

    - name: Start Chrony (Debian/Ubuntu)
      service:
        name: chrony
        state: started
        enabled: true
      when: ansible_os_family == "Debian"

    - name: Start Chrony (Rocky)
      service:
        name: chronyd
        state: started
        enabled: true
      when: ansible_distribution == "Rocky"

    - name: Start Chrony (SUSE)
      service:
        name: chronyd
        state: started
        enabled: true
      when: ansible_distribution == "openSUSE Leap" 

  handlers:
    - name: Restart chrony on Debian
      service:
        name: chrony
        state: restarted
      when: ansible_os_family == "Debian"
      listen: "Restart Chrony Service"

    - name: Restart chronyd on Rocky
      service:
        name: chronyd
        state: restarted
      when: ansible_distribution == "Rocky"
      listen: "Restart Chrony Service"

    - name: Restart chronyd on SUSE
      service:
        name: chronyd
        state: restarted
      when: ansible_distribution == "openSUSE Leap"
      listen: "Restart Chrony Service"
...
```

```console
[vagrant@ansible ema]$  ansible-playbook playbooks/chrony-01.yml
...
PLAY [all] ***********************************************************************************

TASK [Gathering Facts] ***********************************************************************
ok: [suse]
ok: [rocky]
ok: [debian]
ok: [ubuntu]

TASK [Install Chrony (Debian/Ubuntu)] ********************************************************
skipping: [rocky]
skipping: [suse]
ok: [debian]
ok: [ubuntu]

TASK [Install Chrony (Rocky)] ****************************************************************
skipping: [debian]
skipping: [suse]
skipping: [ubuntu]
ok: [rocky]

TASK [Install Chrony (SUSE)] *****************************************************************
skipping: [rocky]
skipping: [debian]
skipping: [ubuntu]
ok: [suse]

TASK [Install custom configuration] **********************************************************
changed: [ubuntu]
changed: [rocky]
changed: [suse]
changed: [debian]

TASK [Start Chrony (Debian/Ubuntu)] **********************************************************
skipping: [rocky]
skipping: [suse]
ok: [ubuntu]
ok: [debian]

TASK [Start Chrony (Rocky)] ******************************************************************
skipping: [debian]
skipping: [suse]
skipping: [ubuntu]
ok: [rocky]

TASK [Start Chrony (SUSE)] *******************************************************************
skipping: [rocky]
skipping: [debian]
skipping: [ubuntu]
ok: [suse]

RUNNING HANDLER [Restart chrony on Debian] ***************************************************
skipping: [rocky]
skipping: [suse]
changed: [ubuntu]
changed: [debian]

RUNNING HANDLER [Restart chronyd on Rocky] ***************************************************
skipping: [debian]
skipping: [suse]
skipping: [ubuntu]
changed: [rocky]

RUNNING HANDLER [Restart chronyd on SUSE] ****************************************************
skipping: [rocky]
skipping: [debian]
skipping: [ubuntu]
changed: [suse]
...
```


## Approche "Intelligente" (chrony-02.yml)

Pour la meilleure approche, on sépare les différences en fichiers de variables, on va se retrouver avec cette structure :

```.
.
├── playbooks/
│   ├── chrony-02.yml
│   └── vars/
│       ├── chrony_debian.yml
|       ├── chrony_ubuntu.yml
│       ├── chrony_rocky.yml
│       └── chrony_opensuse-leap.yml
```

et les fichiers de variables contiendront les éléments spécifiques à chaque distribution, comme le nom du service ou du paquet.
Voici un exemple pour Debian :

```yaml
# vars/chrony_debian.yml
---
chrony_package: chrony
chrony_service: chrony
chrony_file : "/etc/chrony/chrony.conf"
...
```

Du coup maintenant le playbook devient plus fluide et plus facile à maintenir, on n'a plus de conditions dans les tâches, juste des variables qui changent en fonction de l'hôte.

```yaml
# chrony-02.yml
---
- hosts: all
  tasks:
    - name: Include distribution-specific variables
      include_vars: >
        vars/chrony_{{ansible_distribution|lower|replace(" ", "-")}}.yml

    - name: Update package information on Debian/Ubuntu
      apt:
        update_cache: true
        cache_valid_time: 3600
      when: ansible_os_family == "Debian"

    - name: Install Chrony
      package:
        name: "{{ chrony_package }}"

    - name: Install custom configuration
      copy:
        dest: "{{ chrony_file }}"
        content: |
          server 0.fr.pool.ntp.org iburst
          server 1.fr.pool.ntp.org iburst
          server 2.fr.pool.ntp.org iburst
          server 3.fr.pool.ntp.org iburst
          driftfile /var/lib/chrony/drift
          makestep 1.0 3
          rtcsync
          logdir /var/log/chrony
      notify: Restart Chrony

    - name: Start Chrony & enable it on boot
      service:
        name: "{{ chrony_service }}"
        state: started
        enabled: true

  handlers:
    - name: Restart Chrony
      service:
        name: "{{ chrony_service }}"
        state: restarted
...
```

```console
[vagrant@ansible ema]$  ansible-playbook playbooks/chrony-02.yml

PLAY [all] ***********************************************************************************

TASK [Gathering Facts] ***********************************************************************
ok: [suse]
ok: [rocky]
ok: [ubuntu]
ok: [debian]

TASK [Include distribution-specific variables] ***********************************************
ok: [rocky]
ok: [debian]
ok: [suse]
ok: [ubuntu]

TASK [Update package information on Debian/Ubuntu] *******************************************
skipping: [rocky]
skipping: [suse]
ok: [debian]
ok: [ubuntu]

TASK [Install Chrony] ************************************************************************
ok: [suse]
ok: [debian]
ok: [rocky]
ok: [ubuntu]

TASK [Install custom configuration] **********************************************************
changed: [ubuntu]
changed: [suse]
changed: [rocky]
changed: [debian]

TASK [Start Chrony & enable it on boot] ******************************************************
ok: [ubuntu]
ok: [rocky]
ok: [debian]
ok: [suse]

RUNNING HANDLER [Restart Chrony] *************************************************************
changed: [ubuntu]
changed: [debian]
changed: [rocky]
changed: [suse]
...
```

[⬅️ Atelier-16](/formation-ansible/atelier-16/README.md) | [Atelier-18➡️](/formation-ansible/atelier-18/README.md)