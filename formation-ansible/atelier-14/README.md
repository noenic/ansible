[🏠 Sommaire ](../README.md)
# ATELIER-14 : Les Variables Ansible

## Initialisation de l'atelier
Démarrage de l'environnement et connexion au `control` Host et on vérifie que tout fonctionne avec un ping sur tous les hôtes :

```console
[vagrant@control ema]$ ansible -m ping all
target01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
target02 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
target03 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

##  Playbook n°1 de variables

```yaml
---
- hosts: localhost
  gather_facts: false
  vars:
    mycar: Ford Fiesta 
    mybike: Kawasaki Z500
  tasks:
    - name: Display favorites
      debug:
        msg: "Ma voiture : {{ mycar }}, Ma moto : {{ mybike }}"
...
```

```console
[vagrant@control ema]$ ansible-playbook playbooks/myvars1.yaml 

PLAY [localhost] ***************************************************************************

TASK [Display favorites] *******************************************************************
ok: [localhost] => {
    "msg": "Ma voiture : Ford Fiesta, Ma moto : Kawasaki Z500"
}

PLAY RECAP *********************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
> [!TIP]
> On peut aussi surcharger les variables définies dans le playbook en utilisant l'option `-e` ou `--extra-vars` lors de l'exécution du playbook.

```console
[vagrant@control ema]$ ansible-playbook playbooks/myvars1.yaml -e mycar=Tesla

PLAY [localhost] ***************************************************************************

TASK [Display favorites] *******************************************************************
ok: [localhost] => {
    "msg": "Ma voiture : Tesla, Ma moto : Kawasaki Z500"
}

PLAY RECAP *********************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[vagrant@control ema]$ ansible-playbook playbooks/myvars1.yaml -e mycar=Ferrari -e mybike=Ducati

PLAY [localhost] ***************************************************************************

TASK [Display favorites] *******************************************************************
ok: [localhost] => {
    "msg": "Ma voiture : Ferrari, Ma moto : Ducati"
}

PLAY RECAP *********************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

## Playbook n°2 de variables

```yaml
---
- hosts: localhost
  gather_facts: false
  tasks:
    - name: Define variables dynamically
      set_fact:
        mycar: Renault Clio
        mybike: Honda CB500

    - name: Display favorites
      debug:
        msg: "Ma voiture : {{ mycar }}, Ma moto : {{ mybike }}"
...
```

```console
[vagrant@control ema]$ ansible-playbook playbooks/myvars2.yaml

PLAY [localhost] ***************************************************************************

TASK [Define variables dynamically] ********************************************************
ok: [localhost]

TASK [Display favorites] *******************************************************************
ok: [localhost] => {
    "msg": "Ma voiture : Renault, Ma moto : Honda"
}

PLAY RECAP *********************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

> [!NOTE]
> On peut aussi surcharger les variables définies dynamiquement :

```console
[vagrant@control ema]$ ansible-playbook playbooks/myvars2.yaml -e mycar=Ferrari -e mybike=Ducati

PLAY [localhost] ***************************************************************************

TASK [Define variables dynamically] ********************************************************
ok: [localhost]

TASK [Display favorites] *******************************************************************
ok: [localhost] => {
    "msg": "Ma voiture : Ferrari, Ma moto : Ducati"
}

PLAY RECAP *********************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

## Playbook n°3 de variables

```yaml
---
- hosts: all
  gather_facts: false
  tasks:
    - name: Display favorites from inventory
      debug:
        msg: "Ma voiture : {{ mycar }}, Ma moto : {{ mybike }}"
...
```
> [!WARNING]
> Si on l'exécute tel quel, on a une erreur car les variables ne sont pas définies mais on va les définir dans des variables de groupe.

```console
[vagrant@control ema]$ mkdir -p group_vars
cat << EOV > group_vars/all.yml
---
mycar: VW
mybike: BMW
...
EOV
````

```console
[vagrant@control ema]$ ansible-playbook playbooks/myvars3.yaml

PLAY [all] *********************************************************************************

TASK [Display favorites from inventory] ****************************************************
ok: [target01] => {
    "msg": "Ma voiture : VW, Ma moto : BMW"
}
ok: [target02] => {
    "msg": "Ma voiture : VW, Ma moto : BMW"
}
ok: [target03] => {
    "msg": "Ma voiture : VW, Ma moto : BMW"
}

PLAY RECAP *********************************************************************************
target01                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

## Surcharge pour l'hôte `target02`
On peut surcharger les variables pour un hôte spécifique en créant un fichier de variables d'hôte :
```console
[vagrant@control ema]$ mkdir -p host_vars
cat << EOV > host_vars/target02.yml
---
mycar: Mercedes
mybike: Honda
...
EOV
```

```console
[vagrant@control ema]$ ansible-playbook playbooks/myvars3.yaml

PLAY [all] *********************************************************************************

TASK [Display favorites from inventory] ****************************************************
ok: [target01] => {
    "msg": "Ma voiture : VW, Ma moto : BMW"
}
ok: [target02] => {
    "msg": "Ma voiture : Mercedes, Ma moto : Honda"
}
ok: [target03] => {
    "msg": "Ma voiture : VW, Ma moto : BMW"
}

PLAY RECAP *********************************************************************************
target01                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

> [!NOTE]
> On voit que les variables définies pour `target02` ont été utilisées à la place de celles définies par défaut pour tous les hôtes.


## Saisie interactive de variables
On peut aussi demander à l'utilisateur de saisir des variables au moment de l'exécution du playbook :

```yaml
---
- hosts: localhost
  gather_facts: false
  vars_prompt:
    - name: user
      prompt: "Utilisateur"
      default: microlinux
      private: false
    - name: password
      prompt: "Mot de passe"
      default: yatahongaga
      private: true
  tasks:
    - name: Display credentials
      debug:
        msg: "L'utilisateur est {{ user }} et son mot de passe est {{ password }}"
...
```

```console
[vagrant@control ema]$ ansible-playbook playbooks/display_user.yaml
Utilisateur [microlinux]: noenic
Mot de passe [yatahongaga]: 

PLAY [localhost] ***************************************************************************

TASK [Display credentials] *****************************************************************
ok: [localhost] => {
    "msg": "L'utilisateur est noenic et son mot de passe est supermotdepasse"
}

PLAY RECAP *********************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[vagrant@control ema]$ ansible-playbook playbooks/display_user.yaml
Utilisateur [microlinux]: 
Mot de passe [yatahongaga]: 

PLAY [localhost] ***************************************************************************

TASK [Display credentials] *****************************************************************
ok: [localhost] => {
    "msg": "L'utilisateur est microlinux et son mot de passe est yatahongaga"
}

PLAY RECAP *********************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

> [!NOTE] On voit que les valeurs par défaut sont utilisées si l'utilisateur ne saisit rien. et le mot de passe est masqué lors de la saisie.
