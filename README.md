@"
# TP 4 Kubernetes - Compte rendu

Cluster kind ``tp-k8s`` : 1 control-plane + 2 workers (3 conteneurs Docker).
Note : sous PowerShell 5.1, pod.yaml a ete genere avec ``| Set-Content -Encoding ascii`` car ``>`` ecrit en UTF-16.

## Q1. Role des Pods de kube-system
- **etcd** : base cle-valeur qui stocke tout l'etat du cluster.
- **kube-apiserver** : point d'entree unique (API REST) ; kubectl et tous les composants passent par lui.
- **kube-scheduler** : choisit le noeud sur lequel placer chaque nouveau Pod.
- **kube-controller-manager** : execute les boucles de controle (Deployment, ReplicaSet, Node...) qui ramenent l'etat reel vers l'etat desire.
- **kube-proxy** : present sur chaque noeud, programme les regles reseau qui implementent les Services.
- **CoreDNS** : DNS interne du cluster, resout les noms des Services.
- **kindnet** : plugin reseau (CNI) de kind, assure la communication entre Pods de noeuds differents.

## Q2. Pourquoi le Pod web n'est pas recree ?
``kubectl run`` cree un Pod seul, sans controleur : personne ne surveille son existence. Les Pods du Deployment appartiennent a un ReplicaSet dont la boucle de controle maintient le nombre de replicas voulu et recree tout Pod supprime.

## Q3. Deployment, ReplicaSet, Pod
Le Deployment gere des ReplicaSets (un par version du template de Pod), et chaque ReplicaSet gere ses Pods. A la fin il existe 3 ReplicaSets : v1.10 (0 Pod), v9.99 (0 Pod) et v1.11 (6 Pods, reutilise par le rollback). Les anciens sont conserves pour permettre le rollback (revisionHistoryLimit, 10 par defaut). Historique : revisions 1, 3 et 4.

## Q4. Comment le Service trouve ses Pods ?
Le Service utilise un selecteur de labels (``app=whoami``). Le controleur EndpointSlice maintient la liste des IP des Pods correspondants et prets (``kubectl get endpointslices`` : ``whoami-tmdxw`` liste 6 adresses, une par Pod), et kube-proxy s'en sert pour repartir le trafic.

## Q5. Disponibilite pendant l'image cassee
Oui, l'application est restee disponible (la boucle curl n'a pas ete interrompue). La strategie RollingUpdate par defaut a maxUnavailable = 25 % et maxSurge = 25 %. Avec 6 replicas, maxUnavailable = 1 : Kubernetes n'a retire qu'un seul ancien Pod, les nouveaux Pods restant en ImagePullBackOff (jamais Ready). Au moins 5 Pods sains ont continue de servir et le rollout s'est bloque sans degrader le service.

## Defi DNS : whoami.default.svc.cluster.local
- ``whoami`` : nom du Service
- ``default`` : namespace
- ``svc`` : type d'objet (Service)
- ``cluster.local`` : domaine du cluster

Test depuis un Pod busybox temporaire : ``wget -qO- http://whoami.default.svc.cluster.local`` a repondu avec ``Hostname: whoami-c66799bf6-24zjb``.
"@ | Set-Content -Encoding utf8 README.md
