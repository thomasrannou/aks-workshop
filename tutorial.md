---
title: AZURE KUBERNETES SERVICES
---

--sep--
---
title: Introduction
---

# Introduction

Ce repository contient le code ainsi que les instructions vous permettant de 
réaliser le workshop **déployer votre premier cluster kubernetes dans Azure**.

## Pré-requis

Afin de réaliser ce workshop, vous aurez besoin: 

- D'un PC (ou Mac) de développement, sur lequel il faudra installer un certain nombre d'outils,
- D'un abonnement Azure (d'essai, payant ou MSDN)

--sep--
---
title: Objectif du workshop
---

# Objectif du workshop

Ce workshop, accessible à **tous les développeurs même sans connaissance de Kubernetes ou d'Azure **, vous permettra de découvrir le déploiement de cluster Kubernetes dans Azure ainsi que le déploiement d'application conteneurisée dans cet environnement.

![Logo du projet Kubernetes](media/kubernetes-logo.png)

--sep--
---
title: Préparez votre machine de dev
---

# Préparer sa machine de dev

Afin de pouvoir provisionner votre cluster et y déployer l'application vous aurez besoin de plusieurs outils (gratuits) : 

- [.NET 5](https://dotnet.microsoft.com/download)
- [Visual Studio Code](https://code.visualstudio.com/?wt.mc_id=WTMCID)
- [Docker](https://www.docker.com/products/docker-desktop)
- [Azure CLI](https://docs.microsoft.com/fr-fr/cli/azure/install-azure-cli)
- [Powershell 7](https://github.com/PowerShell/PowerShell/releases/tag/v7.1.2)
--sep--
---
title: Préparez votre environnement Azure
---

# Préparer son environnement Azure

Afin de réaliser cet atelier, vous aurez besoin d'une souscription Azure. Il y a plusieurs moyens d'en obtenir une: 

- (**Obligation**) Si vous lisez cet atelier durant le Roadshow, vous pouvez utiliser l'Azure Pass que nous vous fournissons,
- Ou si vous êtes abonnés MSDN, utiliser les crédits offerts par votre abonnement.
- Ou créer un [abonnement d'essai](https://azure.microsoft.com/en-us/free/?wt.mc_id=WTMCID).

## Utiliser votre Azure Pass

1. Rendez-vous sur [microsoftazurepass.com](https://www.microsoftazurepass.com/?wt.mc_id=WTMCID) et cliquez sur **Start**,
![Démarrer l'utilisation du pass](media/redeempass-1.jpg)
2. Connectez vous avec un compte Microsoft Live **Vous devez utiliser un compte Microsoft qui n'est associé à aucune
 autre souscription Azure**
3. Vérifiez l'email du compte utilisé et cliquez sur **Confirm Microsoft Account**
![Confirmer le compte](media/redeempass-2.jpg)
4. Entrez le code que nous vous avons communiqués, puis cliquez sur **Claim Promo Code** (et non, le code présent sur la
 capture d'écran n'est pas valide ;) ),
![Indiquer son code](media/redeempass-3.jpg)
5. Nous validons votre compte, cela prend quelques secondes
![Validation du code](media/redeempass-4.jpg)
6. Nous serez ensuite redirigé vers une dernière page d'inscrption. Remplissez les informations, puis cliquez sur **Suivant**
![Entrer les informations](media/redeempass-5.jpg)
7. Il ne vous restera plus que la partie légale: accepter les différents contrats et déclarations. Cochez les cases que 
vous acceptez, et si c'est possible, cliquez sur le bouton **Inscription**
![Accepter les conditions légales](media/redeempass-6.jpg)

Encore quelques minutes d'attente, et voilà, votre compte est créé ! Prenez quelques minutes afin d'effectuer la 
visite et de vous familiariser avec l'interface du portail Azure.

![Accueil du portail Azure](media/redeempass-7.jpg)

--sep--
---
title: AKS : le service managé k8s dans Azure
---

# AKS : Le service managé K8S dans Azure

Avant de présenter Azure Kubernetes Service en tant que produit Azure, il faut s’intéresser à l’outil Kubernetes lui même.

Kubernetes est une plateforme open source d’orchestration de containers créée par Google, puis offert à la Cloud Native Computing Foundation en 2015.
Kubernetes permet d’automatiser le déploiement et la gestion d’applications conteneurisées. Il gère le cycle de vie des services en proposant scalabilité et haute disponibilité.

Kubernetes peut fonctionner avec n’importe quel système de container conforme au standard Open Container Initiative et notamment le plus connu d’entre eux : Docker.

## Architecture de Kubernetes

![Architecture de Kubernetes](media/Kubernetes-architecture.png)

**Master** : Le Kubernetes master est responsable du maintien de l’état souhaité pour votre cluster. Il gère la disponibilité des nodes.

**etcd** : les données de configuration du cluster. Il représente l’état du cluster à n’importe quel instant.

**Node** : Machine virtuelle ou physique permettant l’exécution de pods.

**Kubelet** : il est responsable de l’état d’exécution de chaque nœud. Il prend en charge le démarrage, l’arrêt, et la maintenance des conteneurs d’applications organisés en pods.

**kubeproxy** : Il est responsable d’effectuer le routage du trafic vers le conteneur approprié.

**Pod** : unité d’exécution de K8s. Contient un ou plusieurs conteneur. Un Pod représente un processus en cours d’exécution dans votre cluster.

## Kubernetes dans Azure

En mars 2016, Microsoft lance Azure Container Service. ACS est une offre PAAS, aujourd’hui obsolète, permettant de déployer rapidement un cluster Kubernetes, DC/OS ou Docker Swarm pour provisionner et manager des conteneur Docker.

Aussi, compte tenu de l’engouement autour de Kubernetes, Microsoft a décidé avec AKS (Azure Kubernetes Services) d’investir fortement sur l’intégration de Kubernetes dans Azure et d’en faire un service managé. Azure gère pour nous les tâches critiques telles que le déploipement, l’analyse de l’intégrité et la maintenance.  Nous devons uniquement nous soucier des nœuds de notre cluster.

Grâce à AKS, plateforme de choix pour la mise en oeuvre de microservices, Microsoft nous propose dans Azure une stack technique agile et robuste pour déployer et manager nos containers.

--sep--
---
title: Un projet .Net 5
---

# Un projet .Net

Le but de ce tutoriel est de déployer un projet ASP .Net 5 conteneurisé dans un cluster Kubernetes dans Azure (AKS).

## Génération d'une image Docker de notre projet

Pour les utilisateurs sous Windows, configurer votre Docker en mode "Linux containers".
Sous Windows toujours, Docker pourra etre paramétré pour utiliser WSL2 ou la virtualisation HyperV. Les deux fonctionneront.

Le projet en lui même est à télécharger [ici](https://github.com/thomasrannou/aks-workshop/tree/main/ApplicationDemoWorkshop).

On va donc commencer par récupérer le projet puis ouvrir une fenêtre Powershell. Dans un premier temps, vous allez devoir générer une image Docker et donc compiler le projet que vous avez récupéré. Dans votre fenêtre powershell, positionnez vous à la racine de votre repertoire projet et executez :

_docker build -f "ApplicationDemoWorkshop/Dockerfile" . -t aksworkshop_

![Génération de l'image Docker](media/1-build.PNG)

On peux maintenant demander la création d'un conteneur basé sur cette image :

_docker run -d -p 8080:80 --name conteneurdemo aksworkshop_

![Execution d'un conteneur](media/2-execution.PNG)

Et accéder à : http://localhost:8080/ pour valider le bon fonctionnement de notre site :

![Validation du site web](media/3-website.PNG)

--sep--
---
title: Un environnement Azure
---

# Un environnement Azure

## Créer un resource group

Toujours dans votre fenêtre Powershell, executez un _az login_ pour vous connecter à votre souscription Azure.

Nous allons commencer par créer un groupe de ressources (_resource group_). C'est un conteneur logique pour l'ensemble des services que vous allez créer ensuite. 
Chaque service Azure doit absolument être déployé dans un resource group. Ici le groupe de ressource va accueillir mon container registry et mon cluster Kubernetes :

_az group create --name rg-workshop --location francecentral_

![Creation du ressource group](media/4-ressourcegroup.PNG)

## Déploiement du projet dans un Azure Container Registry

Ce [registry](https://azure.microsoft.com/fr-fr/services/container-registry) vous permettra de gérer vos images Docker pour ensuite pouvoir les déployer facilement dans une Azure Web App, dans une Azure Container Instance ou, comme ici, dans un cluster Kubernetes.

Vous pouvez demander la création de votre Container Registry grâce à cette commande :

_az acr create --resource-group rg-workshop --name acr-workshopdevcongalaxy --sku Basic_

![Creation du container registry](media/5-registrycreate.PNG)

Connectez vous maintenant à cette registry :

_az acr login --name acrworkshopdevcongalaxy_

L'authentification se fait ici de façon implicite étant donnée que vous êtes connecté à votre souscription Azure.

![Connexion au container registry](media/6-registryconnect.PNG)

Nous pouvons maintenant y publier notre image Docker.
Pour cela, vous devrez tout d'abord tagguer votre image locale avec le nom de votre registry Azure :

_docker tag aksworkshop acrworkshopdevcongalaxy.azurecr.io/appworkshop:v1_

Puis effectuez un push de l'image : 

_docker push acrworkshopdevcongalaxy.azurecr.io/appworkshop:v1_

Ce qui aura pour effet de démarer l'upload vers votre registry :

![Push de l'image](media/7-registrypush.PNG)

![Fin du push de l'image](media/7-registrypush-end.PNG)

Ces opérations réalisées, si on se rend sur portal.azure.com, on doit trouver un ressourcegroup rg-workshop contenant un Azure Container Registry :

![Vérification sur le portail](media/8-portail-rg.PNG)

Hébergeant lui même une image Docker nommée appworkshop :

![Vérification sur le portail](media/9-portail-registry.PNG)

## Gestion des droits

Avant de provisionner notre cluster AKS, nous avons un aspect sécurité à gérer pour permettre la communication entre notre registry et notre futur cluster.
Ce besoin passe par ce qu'on appelle un service principal.

Je créé tout d’abord un service principal pour mon cluster Kubernetes : Pour permettre à un cluster AKS d’interagir avec d’autres ressources Azure, un service principal Azure Active Directory est utilisé.  Ici on veux faire en sorte que mon cluster AKS est accès à mon container registry.

J’utilise la commande :

_az ad sp create-for-rbac --skip-assignment --name sp-acr_

![Création du service principal](media/10-serviceprincipal.PNG)

Et pour leur utilisation ultérieure je les affecte manuellement à deux variables :

![Création du service principal](media/11-setserviceprincipal.PNG)

J’ai maintenant besoin d’identifier ma registry :

_$registryId=$(az acr show --resource-group rg-workshop --name acrworkshopdevcongalaxy --query "id" --output tsv)_

Je vais maintenant donner à mon service principal le droit d’utiliser ma registry (uniquement pour récupérer une image : acrpull) grâce à cette commande :

_az role assignment create --assignee $spid --scope $registryId --role acrpull_

J’obtiens alors un JSON descriptif de l’autorisation accordée :

![Affectation du rôle](media/12-roleassignacrpull.PNG)

NB : je vous propose dans ce workshop cette façon de faire pour montrer que l'intégration entre ces deux composants est soumise à des droits. Un service Azure n'est pas libre d'utiliser comme bon lui semble un autre service ! 

A noter tout de même que la gestion de ce service principal et l'autorisation acr-pull peux se faire de façon implicite à la création du cluster AKS gràce au paramètre --attach-acr $registryId.

## Création du cluster Azure Kubernetes Services

Je créé maintenant mon Azure Kubernetes Service, en précisant l’id et le password de mon service principal :

_az aks create --resource-group rg-workshop --name aks-workshopdevcongalaxy --location francecentral --node-count 2 --service-principal $spid --client-secret $sppwd --enable-addons monitoring --generate-ssh-keys --vm-set-type VirtualMachineScaleSets --load-balancer-sku standard --zones 1 2 3_

![Déploiement du cluster AKS](media/13-deployaks.PNG)

Avant de continuer, je vais expliquer le dernier paramètre : *--zones* !

Une région est un emplacement physique de datacenter Azure. Par exemple la région France Centre est constitué de 3 datacenters situés en région parisienne.

Les zones de disponibilité sont des emplacements physiquement séparés au sein d’une région Azure comme France Centre (minimum 3). Chaque zone de disponibilité est composée d’un ou de plusieurs centres de données équipés d’une alimentation, d’un refroidissement et d’un réseau indépendants.

Si vous avez bien suivis, vous comprenez maintenant que les clusters AKS déployés à l’aide de zones de disponibilité peuvent répartir les nœuds sur plusieurs zones au sein d’une même région ! Par exemple, un cluster dans la région  France Centre  peut créer des nœuds dans les trois zones de disponibilité de France Centre. Il y aura des noeuds sur chaque datacenter composant la région !

Grace à cela votre cluster AKS est capable de tolérer une défaillance dans l’une de ces zones ; si un des datacenter de la région est indisponible la continuité de service est assuré. A contrario, si tout les nœuds de notre cluster était déployé au même endroit, le service serait indisponible.
Cette notion de zone de disponibilité est fondamentale lorsqu'on s'intéresse à des notions de haute disponibilité et de continuité de service.

![Déploiement du cluster AKS](media/14-deployaksned.PNG)

Nous avons donc provisionner un cluster Azure Kubernetes Service ! Voyons maintenant comment l'administrer en local.
Pour gérer un cluster Kubernetes, on utilise *kubectl*, le client de ligne de commande Kubernetes . Pour installer kubectl, si il n'est pas déja présent, utilisez :

_az aks install-cli_

![Installation de kubectl](media/15-installkubectl.PNG)

L'installeur vous demande de jouer cette commande Powershell : $env:path += 'C:\Users\trannou\.azure-kubelogin' pour poursuivre. Cette modification ne sera valable que pour la fenêtre Powershell courante. Pour une solution pérenne , ajoutez cette même entrée aux gestionnaires de variables d'environnement Windows :

![Gestion des variables d'environnement](media/16-environnementvar.PNG)

Maintenant pour pouvoir utiliser votre cluster en local, il faut executer :

_az aks get-credentials --resource-group rg-workshop --name aks-workshopdevcongalaxy_

![Connexion au cluster AKS](media/17-credentialsaks.PNG)

Cette commande permet de renseigner le kubeconfig local contenant les informations nécessaires pour accéder au cluster distant :
- L’utilisateur et ses certificats/clés
- L’adresse vers l’API server
- Le namespace par défaut

Notre cluster est déployé et j'y ai accès depuis mon poste local, nous allons pouvoir passer au déploiement de l'application !

--sep--
---
title: Déploiement de l'application
---

# Déploiement de l'application

Je vvous propose d'utiliser un fichier yaml pour déployer une instance de notre image Docker (hebergé dans mon container registry) dans mon cluster AKS. 
Le fichier yaml à utiliser est présent sur le repo, dans le dossier de l'application.

Attention à la ligne 22, le champ containers/image spécifie le chemin vers mon image dans mon ACR.
Si vous avez choisis un nom différent du mien, il faudra modifier le fichier.

## Déploiement via un fichier Yaml

Il est constitué d'une partie Deployment contenant :
- la description du ReplicaSet qui conditionne le nombre de pods actifs.
- la stratégie de rolling update pour les montée de versions.
- la configuration des conteneurs que nous déployons (image, ressources, port exposé)

Ainsi qu'une partie Service pour rendre accessible mes pods de l'extérieur au travers d'un Service de type LoadBalancer.

La commande à executer pour utiliser ce fichier et créer une instance de conteneur à partir de mon image est :

_kubectl apply -f .\deploytoaks.yaml_

En résultat vous devez obtenir ceci :

![Yaml Apply](media/18-apply.PNG)

Vérifions que ce nous avons déployé !

![Vérification du déploiement](media/19-check.PNG)

Je trouve normalement sur mon cluster un deployment qui execute un pod ainsi qu'un service pour exposer mon application.
Vous voyez également une ip externe à été affectée à mon service. Si vous la renseignez dans votre navigateur vous devriez retomber sur une interface connue.

![Vérification du service](media/20-checkservice.PNG)

Maintenant que notre application est déployée, telle que je l'ai demandé, on peux commencer à appréhender la puissance de Kubernetes. Si je demande la suppression de mon pod :

_kubectl delete pod idpod_

Grace au replicaset, un nouveau pod est automatiquement créé pour le remplacer.

![Création automatique d'un nouveau pod](media/21-checkdeployment.PNG)

--sep--
---
title: Autoscaling
---

# Autoscaling

## Autoscaling de pods

Le scaling consiste à augmenter ou diminuer le nombre d’instances d’une application. Cela permet par exemple de résister à un pic de charge si votre service est fortement sollicité par moment et très peu le reste du temps. On peut configurer grâce à Kubernetes l’upscale et le downscale pour s’adapter en temps réel aux besoins de nos utilisateurs.

Si on reprend le yaml utilisé précédemment, une section va nous intéresser particulièrement pour l'autoscaling :

![Ressource d'un pod](media/23-ressource.PNG)

Il sagit de **ressources** avec la définition des propriétés requests et limits.

- **Requests**, c’est ce que le pod est garanti d’avoir à sa disposition pour fonctionner. Ici mon container aura donc 250m de CPU (1/4 de VCPU)
- **Limits** en revanche c’est une sécurité sur la consommation de ressources du container. Il ne pourra pas utiliser plus de 500m de CPU (1/2 VCPU). Si il va au-delà, le pod sera détruit.

Les valeurs de CPU sont définis en milli-cores. Si vous avez besoin de deux VCPU il faudra indiquer 2000m. La notation "2" sera équivalente.

La commande kubectl autoscale que nous utiliserons ensuite crée un objet **HorizontalPodAutoscaler** (HPA) qui cible une ressource spécifiée et la fait évoluer si nécessaire. Le HPA ajuste périodiquement le nombre d’instances dupliquées de façon à respecter la valeur d’utilisation moyenne du processeur que vous spécifiez.

Ici, je vais demander un HPA sur mon application appworkshop. Je lui indique de scaler entre 5 et 100 réplicas. L’upscale se fera si le pourcentage CPU consommé dépasse les 70% de ce qui est alloué, ici c’est donc 70% de 500millicores de CPU.

Pour demander la création d'un HPA sur mon deployment executez :

_kubectl autoscale deployment deployment-appworkshop --max 100 --min 5 --cpu-percent 70_

Si je vérifie grace à cette commande : 
_kubectl describe pod | select-string -pattern '^Name:','^Node:'_

![Déploiement d'un HPA](media/22-deployhpa.PNG)

Je constate qu’après la mise en place de mon HPA, j’ai désormais 5 pods d’opérationnel déployés sur les deux nodes à notre disposition. Les appels vers mon site web seront automatiquement répartis vers ces deux pods via le loadbalancer de mon service.

Précision : La commande kubectl get hpa me permet d’afficher mes differents scaling mis en place sur mon cluster k8s.

_kubectl get hpa_

NB : cette configuration de l'autoscale de mon deployment peux également être géré par un fichier [yaml](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/).

## Autoscaling de nodes

AKS nous offre la possibilité d’allouer et de désallouer automatiquement des nodes Kubernetes pour l’hébergement de nos applications en fonction de critères que nous détaillerons ci-dessous, c’est grâce à **l’autoscaler de cluster**. Cet autoscale utilise les VMSS, cette fonctionnalité Azure nous permet de gérer des collections de machines virtuelles (identiques), capable de s’instancier à la demande.
 
L’autoscaler augmente ou diminue donc automatiquement la taille du [pool de nœuds](https://thomasrannou.azurewebsites.net/2020/01/28/les-pools-de-noeuds-dans-aks), en analysant la demande de ressources des pods.

- Si les pods ne peuvent pas être démarrés, car il n’y a pas assez de puissance cpu/ram sur les nœuds du pool, l’autoscaler de cluster en ajoute, jusqu’à atteindre la taille maximale du pool de nœuds.
- Si les nœuds sont sous-utilisés et que tous les pods peuvent être déployés en utilisant moins de noeuds, l’autoscaler de cluster supprime des nœuds, jusqu’à atteindre la taille minimale du pool.

Concretement, pour le mettre en place un simple *aks update* fait l'affaire :

_az aks update --resource-group rg-workshop --name aks-workshopdevcongalaxy --enable-cluster-autoscaler --min-count 2 --max-count 5_

![Description des nodes](media/24-deployautoscalecluster.PNG)

On a maintenant un cluster avec un nodepool scalable entre 2 et 5 :

_kubectl describe nodes | select-string -pattern '^Name:','zone='_

![Description des nodes](media/25-aksnodes.PNG)

Si on s'attarde sur le résultat de cette commande, on constate bien l'utilisation des zones de disponibilités. La 1ere VM est sur francecentral-1 et la seconde sur francecentral-2.

## Stress Test

Maintenant, nous allons stresser un peu l'application et simuler un fort trafic sur mon site. 
En toute logique, l’autoscaling configuré pour mon cluster AKS doit intervenir et le nombre de pods devraient se dupliquer. 
L'autoscaler de cluster devrait également se déclencher pour provisionner des nodes pour héberger tout les pods.

Pour ce faire je vais utiliser un outil du nom de [Vegeta](https://github.com/tsenart/vegeta/releases) !

![Scaling des pods](media/gif-vegeta.gif)

La version Windows est en général un peu en retard sur les mises à jour. Il faut donc si besoin revenir à la version précédente pour trouver l'asset Windows.
Dezippez le dossier dans le repertoire de votre choix.

Pour héberger cet outil, je vais déployer une VM dans Azure. 

![Création de la VM dans Azure](media/26-vm.PNG)

Je la positionne dans mon ressource group de travail, je configure un User/Password pour m'y connecter tout à l'heure.
Important : conservez le port RDP (3389) ouvert tel que configuré par défaut.
Une fois la 1ere page complétée, vous pouvez directement cliquer sur "Vérifier et créer" :

![Paramétrage de la VM](media/26-vm2.PNG)

Le déploiement est en cours :
![Déploiement de la VM](media/26-vm3.PNG)

Une fois terminé, cliquez sur "Acceder à la ressource" :

![Déploiement terminé](media/26-vm4.PNG)

Puis télécharger le fichier RDP via le bouton "Connecter" :

![Téléchargement du fichier RDP](media/26-vm5.PNG)

![Téléchargement du fichier RDP](media/26-vm6.PNG)

Pour déclencher le test, utilisez cette commande. Il faut renseigner la bonne url, la durée, et le nombre d'appels par seconde (ici 5000)

_echo GET http://20.74.40.2 | vegeta.exe attack -duration=5m -rate=5000 -output=stress-results.bin_

Je lance mon test de charge : 
![Lancement de Vegeta](media/27-vegeta.PNG)

Au bout de quelques minutes je vois déja le scaling en oeuvre avec la création d'un node :

![Scaling des nodes](media/28-scalenodes.PNG)

Et la multiplicaiton du nombres de pods :

![Scaling des pods](media/29-scalepods.PNG)

Pour visualiser le résultat de Vegeta, cette commande peux vous créer un graphique sur la latence (et les potentielles erreurs http) induite par votre test de charge :

_vegeta.exe plot -title=Results stress-results.bin > stress-results-plot.html_

Ouvrez le fichier html pour constatez la latence induite par la charge sur votre application.

Quand je stoppe le load, je vais constater l’inverse et voir progressivement mon nombre de pods diminuer, jusqu’à revenir à ma situation initiale, au bout de seulement quelques minutes :

![Etat initial](media/30-etatinitial.PNG)

On se rend ici compte de toute la puissance de Kubernetes dans cette getion de la scalabilité applicative.
Ce genre de solutions est donc parfaitement adapté au service avec un trafic réseau variable tel que :
- Un site e-commerce particulierement fréquenté en période de solde ou de fêtes de fin d'année.
- Une application de saisie des temps de travail très sollicité à la fin de la semaine ou du mois et quasiment pas en semaine ou le week-end.

--sep--
---
title: Conclusion
---

# Conclusion

Bravo, vous avez fini le workshop !

En résumé, depuis notre application .Net 5 et son Dockerfile nous avons :

- Déployer notre image dans un Azure Container Registry
- Déployer notre application vers notre cluster Kubernetes grâce à un fichier yaml.
- Configurer l'autoscaling sur notre cluster

## Pour aller plus loin

Si il y a une seule raison à retenir pour l'utilisation d'un cluster Kubernetes c'est bien pour garantir de la haute disponibilité.
Je vous propose donc ces deux liens pour gérer cette disponibilité :
- [au niveau de votre infrastructure](https://dev.to/thomasrannou/de-la-haute-disponibilite-avec-azure-kubernetes-services-le-focus-infra-2jdc)
- [et de vos applications](https://dev.to/thomasrannou/de-la-haute-disponibilite-avec-azure-kubernetes-services-le-focus-apps-3l8l) 

## Crédit

Ce workshop a été créé par [Thomas Rannou](https://twitter.com/thomas_rannou) puis relu et réarrangé par [Olivier Leplus](https://twitter.com/olivierleplus). 
