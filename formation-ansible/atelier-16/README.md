# ATELIER-16 : Facts et Variables Implicites

Le jeu sur cet atelier est juste de se balader dans les facts qu'on peut récupérer sur les hôtes avec `ansible -m setup (target)`.

## Gestionnaire de paquets (pkg-info.yml)
L'objectif est d'identifier l'outil de gestion des paquets via les facts.

```yaml
---
- hosts: all
  tasks:
    - name: Display package manager
      debug:
        msg: "{{ inventory_hostname }} utilise {{ ansible_pkg_mgr }}"
...
```

```console
[vagrant@ansible ema]$ ansible-playbook playbooks/pkg-info.yml
...
TASK [Gathering Facts] *********************************************************
ok: [suse]
ok: [rocky]
ok: [debian]

TASK [Display package manager] *************************************************
ok: [rocky] => {
    "msg": "rocky utilise dnf"
}
ok: [debian] => {
    "msg": "debian utilise apt"
}
ok: [suse] => {
    "msg": "suse utilise zypper"
}
...

```


## Version de Python (python-info.yml)
Vu que Ansible a besoin de Python sur la cible, il peut nous dire quelle version il utilise.

```yaml
---
- hosts: all
  tasks:
    - name: Display Python version
      debug:
        msg: "Version Python sur {{ inventory_hostname }} : {{ ansible_python_version }}"
...
```

```console
[vagrant@ansible ema]$ ansible-playbook playbooks/python-info.yml
...
TASK [Gathering Facts] *********************************************************
ok: [suse]
ok: [rocky]
ok: [debian]

TASK [Display Python version] **************************************************
ok: [rocky] => {
    "msg": "Version Python sur rocky : 3.9.21"
}
ok: [debian] => {
    "msg": "Version Python sur debian : 3.11.2"
}
ok: [suse] => {
    "msg": "Version Python sur suse : 3.6.15"
}
...

```


## Serveurs DNS (dns-info.yml)
On récupère les serveurs DNS configurés sur les hôtes. 
```console
[vagrant@ansible ema]$ ansible debian -m  setup  -a "filter=*dns*"
debian | SUCCESS => {
    "ansible_facts": {
        "ansible_dns": {
            "nameservers": [
                "10.0.2.3"
            ]
        }
    },
    "changed": false
}
```

```yaml
---
- hosts: all
  tasks:
    - name: Display DNS servers
      debug:
        msg: "Serveurs DNS pour {{ inventory_hostname }} : {{ ansible_dns.nameservers | join(', ') }}"
        # join, c'est juste pour s'amuser à faire du formatage de variable, c'est plus joli que d'avoir une liste brute
...
```

```console
[vagrant@ansible ema]$ ansible-playbook playbooks/dns-info.yml
...
TASK [Gathering Facts] *********************************************************
ok: [suse]
ok: [rocky]
ok: [debian]

TASK [Display DNS servers] *****************************************************
ok: [suse] => {
    "msg": "Serveurs DNS pour suse : 10.0.2.3"
}
ok: [rocky] => {
    "msg": "Serveurs DNS pour rocky : 10.0.2.3"
}
ok: [debian] => {
    "msg": "Serveurs DNS pour debian : 10.0.2.3"
}
...
```

