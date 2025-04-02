# Balaxy (nom provisoire - sortie publique fin Avril) 

Interface web pour `ansible-playbook`, afin de mieux suivre l'exécution des playbooks :

![Application view](images/Run.png)

# Mise en place simple

Il suffit d’ajouter une collection contenant un callback plugin, puis d'ajouter deux lignes dans votre fichier de configuration ~/.ansible.cfg :

```bash
callback_plugins=/path/to/newly/installed/collections/callback_plugin/plugins/callback/balaxy 

stdout_callback=callbackName
```

Ensuite, exécutez vos playbooks Ansible normalement :

```bash
ansible-playbook -i ../test.inventory2.ini -i test.inventory3.ini live_demo.yml
```

![ansible-playbook running](images/RunLaunch.png)

Le plugin intercepte les événements générés par Ansible :

![python code](images/EventCallbackPlugin.png) 
(https://github.com/ansible/ansible/blob/devel/lib/ansible/plugins/callback/__init__.py)

Les interprète, puis envoie à un serveur distant :

![simplified workflow](images/basicWorkflow.png)

# Fonctionnalités existantes

## Centralisation des exécutions `ansible-playbook`

![Home page, list of run](images/runList.png)

## Affichage des tâches selon la hiérarchie d’origine :

*playbook → play → rôle → block → include / import…*

![Table of contents](images/hierarchie.png)

## Visualisation des rôles importés et de leurs dépendances :

![Role dependancy graph](images/Role.png)

## Représentation des `include` / `import` :

![Including task file representation](images/include-import.png)

## Représentation des `block` :

![Block representation](images/block.png)

## Le rendu classique toujours disponible dans le terminal :

![Default callback](images/NoHierarchy.png)

## Vue de l'inventaire Ansible :

![Inventory hosts and groups](images/Inventory.png)

## Correspondance entre hôtes et les cibles des plays :

![Host matched in selected plays](images/InventoryView.png)

## Codes visuels pour représenter l’état des hôtes (ok, changed, failed, unreachable…) :
    
![status colors and icon](images/inventoryStatusColors.png)

## Hiérarchie des groupes dans l’inventaire :

![inventory groups hierarchy](images/inventoryGroupsHierarchy.png)

# Variables 

## Suivi des variables et de leurs points d’entrée :

![Run vars, internal config](images/runVars.png)

## Origine des variables (fichier, script, groupe…) :

![Inventory file or script group vars](images/HostVars.png)

# Statistiques & Résultats

## Statistiques globales de la run :

![Run Stats](images/StatsRun.png)

## Vue par tâche et par hôte :

![Gatherings facts](images/Facts.png)

## Hôtes groupés par résultat :

![Host grouped by result](images/GroupedResult.png)

## (prochainement) Affichage spécifique selon le module (ex. gather_facts) :

![gatherings facts result using an special component](images/exempleOfFactsRender.png)

Plutôt que :

![gatherings facts result using default yaml for representation](images/FactsResult.png)

## Timeline par hôte :

![Timeline](images/TimelineFacts.png)

# Documentation intégrée

## Sur les actions des tâches :

![clicking on task action](images/ClickingOnTaskAction.png)
![Doc openning in a modal](images/DocModalOpen.png)

## Sur les variables :

![Playbook group_vars/*](images/DocInVars.png)

# Smart Vars Panel

Un panneau intelligent qui regroupe toutes les variables actives à un moment donné, par hôte :

![smart vars panel](images/smartVars.png)

***Sélectionner un hôte***

![selecting a host](images/smartvarspanelClickOnHost.png)

## Variables par origine (priorité, portée, source...) :

![list of vars](images/SmartVarsOrigins.png)

Chaque élément est cliquable pour afficher sa documentation et son origine directe (hôte, groupe, tâche, play…) :

![play vars](images/smartVarsOriginClickable.png)

## Addition finale des variables (ordre de priorité respecté) :


![vars summs](images/SmartVarsSumms.png)

## Accès rapide aux conflits détectés :

![vars summs conflicts](images/SmartVarsSummsConflict.png)
 
# Limitations

Les limitations actuelles proviennent principalement du plugin Python, notamment sur la manière de capter certaines informations ou de gérer certains cas.

## `ansible-playbook` + `inventory` + `playbooks`

La solution est conçue pour être utilisée avec la commande ansible-playbook, un ou plusieurs fichiers d’inventaire, et **un seul** playbook en paramètre. L'interface **ne fonctionne pas** dans les cas suivants :
- Inventaire dynamique ❌
- Utilisation de la commande ansible (au lieu de ansible-playbook) ❌

## Un seul playbook en argument

Il ne faut passer **qu’un seul** playbook dans la commande. Les `import_playbook` sont gérés à l’intérieur, mais plusieurs fichiers `.yml` en arguments ne sont pas supportés.

❌ Mauvais :
```
ansible-playbook live_demo.yml another_one.yml
```

✅ Correct :
```
ansible-playbook live_demo.yml
```

## Variables contenant des expressions Jinja2

Les variables définies à l’aide de Jinja2 (ex: {{ some_var }}) ne sont pas encore correctement interprétées par le plugin.
Cela **n’impacte pas l’exécution**, mais ces variables peuvent être absentes ou mal affichées dans l’interface.

## Propriétés de scope (connection, sudo, etc.)

Le bon traitement des propriétés de connexion, d’environnement ou d’élévation (sudo) dépend aussi de la capacité à interpréter Jinja2.
En attendant une solution, des **indications visuelles** sont fournies via la table des matières :

![other scope attribute](images/exempleOtherScopeAttr.png)

## Tags

Les tags sont bien enregistrés, mais pas encore affichés et non valorisés dans l'interface.

## Bugs possibles

Le plugin et l’application sont en phase **pré-alpha** :

- Non destinés à un usage en production
- Informations envoyées au serveur sans anonymisation, ce qui peut inclure IP, méthodes d’accès, mots de passe, clés SSH, et

## Rafraîchissement non dynamique

L’interface **n’est pas dynamique** : il faut rafraîchir manuellement la page pour voir les nouvelles informations, ou attendre la fin de l’exécution.

# Conclusion

Plusieurs pistes sont envisagées pour l’avenir de l’outil :
- Meilleur support des attributs cumulatifs (`connection`, `sudo`, `environment`, etc.)
- Une page "explorer" pour :
    - Naviguer dans les entités
    - Accéder à la documentation des modules 
    - Lister les résultats par module sur l’ensemble des runs
    - Trouver toutes les tâches avec un attribut donné (when, changed_when, etc.)
- Segmentation public / privé avec authentification.
- Fonctionnalités collaboratives :
    - Serveur IRC intégré avec salons : 
        - Global
        - Par run
        - Etc...
    - Vignetes collables (info, attention, danger) dans la table des matières.
    - Espace commentaires :
        - Par module
        - Par résultat
        - Par tâche
        - Par run
        - Etc...
- Rendu dynamique en temps réel.
- Des notifications quand une run commence ou se termine.
- L'ajout d'autre plugin *(inventaire / Cache)* à la collection et intégré à l'interface.
- Et plus encore selon les besoins et retours
