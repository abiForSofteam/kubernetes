**NODES**  
1. Les nœuds constituent la base où sont exécutés les pods, car ils offrent l&apos;infrastructure requise pour le déploiement et l&apos;exécution des conteneurs.  
2. Chaque nœud abrite un composant nommé **kubelet**, chargé de surveiller la création, le fonctionnement et la suppression des conteneurs pour maintenir l&apos;état désiré.  
3. Le **kubelet** envoie régulièrement des rapports d&apos;état au plan de contrôle, ce qui garantit une synchronisation continue et la possibilité d&apos;agir rapidement en cas de divergence.  
4. Le composant **kube-proxy** fait aussi partie du nœud et assure la mise en place des règles réseau, permettant le bon routage du trafic vers les pods voulus.  
5. Les conteneurs sont exécutés via un runtime tel que **containerd** ou **CRI-O**, lequel gère l&apos;isolation et la performance de chaque conteneur.  
6. Les administrateurs peuvent associer des labels aux nœuds et définir des **taints** ou **tolerations** pour influer sur la planification des pods selon divers critères (performance, localisation, etc.).  
7. Lorsque les opérateurs souhaitent effectuer une maintenance, ils peuvent utiliser les modes &quot;cordon&quot; ou &quot;drain&quot; pour empêcher la planification de nouveaux pods et déplacer ceux existants.  
8. La logique d&apos;allocation de ressources repose sur des mécanismes comme les **cgroups**, qui veillent à ce qu&apos;aucun conteneur ne dépasse la mémoire ou le CPU autorisés.  
9. Les états d&apos;un nœud (Ready, NotReady, etc.) indiquent s&apos;il est disponible pour héberger des pods ou s&apos;il rencontre des problèmes techniques nécessitant une inspection.  
10. Cette architecture flexible assure une haute disponibilité et une grande évolutivité, car il est aisé d&apos;ajouter ou retirer des nœuds en fonction des besoins.  

**CONTROL PLANE NODE COMMUNICATION**  
1. Le plan de contrôle coordonne la totalité du cluster, tandis que les nœuds exécutent réellement les conteneurs et communiquent régulièrement avec le plan de contrôle.  
2. Le composant **kubelet** envoie des informations sur l&apos;état local d&apos;un nœud au plan de contrôle, incluant le statut des conteneurs et l&apos;occupation des ressources.  
3. Cette communication sortante permet d&apos;éviter d&apos;avoir à autoriser des connexions entrantes sur chaque nœud, ce qui simplifie la configuration du pare-feu.  
4. De son côté, l&apos;API server peut contacter le **kubelet** pour exécuter des commandes telles que `kubectl logs` ou `kubectl exec`, bien que cela puisse être restreint pour des raisons de sécurité.  
5. Les certificats TLS assurent l&apos;authentification et le chiffrement entre le plan de contrôle et les nœuds, renforçant la protection des données échangées.  
6. Les administrateurs peuvent choisir de désactiver ou de restreindre le trafic entrant depuis le plan de contrôle vers le **kubelet**, afin de limiter la surface d&apos;attaque potentielle.  
7. Dans certains scénarios, le routage des commandes vers le nœud est géré via un proxy ou un tunnel, fournissant un intermédiaire entre l&apos;API server et le kubelet.  
8. Cette séparation des flux — node vers API server et API server vers node — constitue un modèle plus robuste, facilitant le déploiement du cluster derrière des pare-feu stricts.  
9. Les bonnes pratiques consistent à définir clairement quels ports doivent être autorisés et à utiliser des certificats client pour vérifier l&apos;identité de chaque kubelet.  
10. Cette approche garantit une communication fiable et sécurisée entre toutes les composantes essentielles de Kubernetes, contribuant à la stabilité globale du cluster.  

**CONTROLLER**  
1. Les contrôleurs s&apos;appuient sur un **control loop** qui compare en permanence l&apos;état actuel d&apos;un objet Kubernetes à l&apos;état souhaité.  
2. Ce mécanisme détecte toute différence entre ce qui est spécifié et la réalité, puis applique les actions nécessaires pour rétablir la conformité.  
3. Le **ReplicationController** (et son successeur ReplicaSet) veille à maintenir un nombre prédéfini de pods, en créant ou en supprimant pour respecter le niveau de répliques souhaité.  
4. Le **DaemonSet** garantit qu&apos;un pod spécifique tourne sur chaque nœud, ce qui est particulièrement utile pour les tâches d&apos;infrastructure comme la journalisation ou la supervision.  
5. Le **Deployment** s&apos;appuie sur un ReplicaSet pour gérer la mise à jour progressive des pods et prendre en charge les retours arrière (rollback) si nécessaire.  
6. Les contrôleurs intégrés sont pris en charge par le composant **kube-controller-manager**, tandis qu&apos;il est également possible de créer des contrôleurs personnalisés pour gérer des ressources définies par l&apos;utilisateur.  
7. Un contrôleur se compose généralement d&apos;un watcher qui surveille l&apos;API server pour les événements, et d&apos;une logique qui décide des actions à entreprendre.  
8. Si un pod est supprimé ou échoue, le contrôleur concerné (ReplicaSet, Deployment, etc.) entreprend rapidement de le recréer ou de le rééquilibrer, assurant ainsi la haute disponibilité.  
9. Certains contrôleurs agissent sur des objets plus abstraits comme les volumes ou les services réseau, de manière à coordonner automatiquement la configuration sous-jacente.  
10. Cette architecture déclarative, basée sur des boucles de contrôle, confère à Kubernetes une résilience et une adaptabilité remarquables pour gérer les charges de travail conteneurisées.  

**LEASES**  
1. Les leases constituent un mécanisme essentiel pour gérer l&apos;élection de leader et le suivi d&apos;activité dans un cluster, en conférant un contrôle fin sur l&apos;attribution du rôle de leader.  
2. Cette ressource s&apos;avère particulièrement utile pour éviter que plusieurs instances d&apos;un composant ne s&apos;exécutent en parallèle en tentant d&apos;assumer la même fonction critique.  
3. Le composant qui détient le lease met à jour régulièrement des champs tels que **holderIdentity** et **renewTime**, indiquant qu&apos;il est actif et toujours présent.  
4. Lorsqu&apos;un participant ne renouvelle plus le lease avant l&apos;expiration spécifiée, d&apos;autres entités peuvent alors prétendre à la position de leader.  
5. Les **contrôleurs** et opérateurs, qu&apos;ils soient intégrés ou personnalisés, se servent de ce modèle pour coordonner leurs tâches et éviter les conflits de ressource.  
6. Cette stratégie s&apos;appuie sur un objet minimaliste stocké dans la base de données du cluster, ce qui limite l&apos;impact sur les performances et la consommation de mémoire.  
7. Le modèle d&apos;élection de leader repose sur la simplicité : une entité est marquée leader tant qu&apos;elle maintient et renouvelle le lease, puis elle cède automatiquement sa place lorsqu&apos;elle échoue à le faire.  
8. Les développeurs peuvent définir la **durée** de validité du lease, permettant d&apos;adapter la tolérance aux interruptions et de réguler la fréquence de renouvellement.  
9. La haute disponibilité du cluster se trouve renforcée, car dès qu&apos;un leader cesse d&apos;être réactif, un nouveau peut émerger rapidement et prendre le relais.  
10. Cette mécanique assure une coordination robuste et évite les duplications de traitement, en garantissant qu&apos;une seule instance d&apos;un composant gère les actions critiques à un moment donné.  

**CLOUD CONTROLLER**  
1. Le **Cloud Controller Manager** est un composant indépendant qui permet à Kubernetes d&apos;interagir de manière native avec un fournisseur de cloud spécifique.  
2. Il englobe plusieurs contrôleurs dédiés, tels que le **Node Controller**, qui vérifie l&apos;existence et la validité des nœuds dans l&apos;infrastructure cloud, et le **Route Controller**, qui gère les tables de routage lorsque la plateforme le permet.  
3. Le **Service Controller** crée et met à jour automatiquement les load balancers lorsque des Services de type LoadBalancer sont définis, simplifiant ainsi la mise en place de points d&apos;entrée externes.  
4. Cette séparation entre la logique cloud et le noyau de Kubernetes facilite la maintenance : chaque fournisseur peut évoluer à son propre rythme, sans perturber le plan de contrôle.  
5. Les opérateurs peuvent déployer le Cloud Controller Manager en tant que binaire ou conteneur, et le configurer à l&apos;aide de flags comme **--cloud-provider** et **--cloud-config** pour indiquer l&apos;API cible.  
6. Cette modularité renforce la portabilité du cluster : il devient possible de changer de fournisseur cloud ou d&apos;en ajouter un nouveau sans réécrire le code central de Kubernetes.  
7. En activant le **Node Controller**, on garantit qu&apos;un nœud inexistant ou déjà supprimé dans le cloud ne persiste pas indéfiniment dans la configuration du cluster.  
8. Les déploiements haute disponibilité se trouvent simplifiés, car plusieurs instances du Cloud Controller Manager peuvent fonctionner en parallèle, évitant la panne unique et favorisant la continuité du service.  
9. Ce découplage offre une flexibilité supplémentaire : certains clouds exigent des réglages précis pour les load balancers ou les volumes, que le Cloud Controller Manager peut appliquer automatiquement.  
10. Grâce à cette architecture, Kubernetes préserve son caractère générique tout en s’adaptant aux spécificités de chaque cloud, ce qui améliore l’expérience globale de gestion et d’évolutivité.  

**CGROUPS**  
1. Les **cgroups** constituent un mécanisme du noyau Linux permettant d’isoler et de gérer précisément l’utilisation des ressources (CPU, mémoire, E/S) par les processus.  
2. Kubernetes s’appuie sur ce mécanisme pour appliquer les **Resource Requests/ Limits**, garantissant qu’aucun conteneur ne dépasse la quantité de CPU ou de mémoire allouée.  
3. Le runtime de conteneurs (containerd, CRI-O, etc.) crée des groupes cgroups pour chaque pod ou conteneur, donnant au noyau la possibilité d’en surveiller la consommation en continu.  
4. Les cgroups peuvent imposer des restrictions strictes : si un conteneur dépasse sa limite de mémoire, le système peut émettre un OOM (Out Of Memory) et tuer ce conteneur.  
5. Cette architecture prévient la monopolisation des ressources par un seul pod, maintenant la stabilité du nœud et la fiabilité de l’ensemble du cluster.  
6. Les **classes de qualité de service** (Guaranteed, Burstable, BestEffort) dépendent des paramétrages de cgroups, définissant la priorité d’un pod et son accès préférentiel ou non aux ressources.  
7. Le kubelet collecte les informations de consommation grâce à ces contrôleurs du noyau, et peut, en cas d’excès, déclencher des actions d’éviction pour protéger la santé globale du nœud.  
8. Les administrateurs gagnent ainsi en visibilité sur le fonctionnement interne : ils savent quel conteneur utilise combien de CPU ou de mémoire, et peuvent ajuster les limites si nécessaire.  
9. Les cgroups fonctionnent de manière hiérarchique : chaque conteneur est un sous-groupe, et l’on peut appliquer des règles plus globales au niveau du nœud, voire du système entier.  
10. Cet ensemble de fonctionnalités favorise une gestion fine et performante des ressources, rendant possible un partage juste et une haute densité de conteneurs sans compromettre la stabilité.  

**CRI**  
1. Le **Container Runtime Interface** établit une frontière standard entre Kubernetes et les différents runtimes de conteneurs, permettant au kubelet d’interagir avec des solutions variées sans dépendre d’une implémentation précise.  
2. Cette interface repose sur un modèle gRPC qui définit plusieurs opérations, notamment la création, la suppression et la mise à jour de conteneurs ainsi que la gestion des images.  
3. Grâce à cette approche, on peut brancher un runtime comme **containerd**, **CRI-O** ou d’autres, sans qu’il soit nécessaire de modifier le code central de Kubernetes.  
4. Le kubelet se contente d’appeler des méthodes CRI pour lancer ou arrêter un conteneur, et laisse le runtime se charger des détails bas niveau, comme l’isolement ou le montage de volumes.  
5. Cette séparation offre une modularité accrue et favorise l’innovation, car chaque runtime peut être développé, amélioré ou remplacé indépendamment.  
6. Les appels CRI sont distribués en deux volets : d’une part la gestion des conteneurs (RuntimeService), d’autre part la gestion des images (ImageService).  
7. Les distributions Kubernetes récentes utilisent souvent **containerd** par défaut, mais des implémentations tierces demeurent compatibles via la CRI.  
8. Le mécanisme CRI permet également de définir des règles communes de logging et de reporting de ressources, le kubelet restant le point central qui analyse les retours et décide des actions.  
9. Les administrateurs bénéficient ainsi d’un environnement plus flexible, car ils peuvent passer d’un runtime à un autre si cela correspond mieux à leurs exigences de performance ou de sécurité.  
10. Finalement, cette conception maintient la philosophie de Kubernetes : décrire l’état souhaité, tandis que la couche d’exécution des conteneurs s’occupe des tâches spécifiques selon les contraintes de la plateforme.  

**GARBAGE COLLECTION**  
1. Le garbage collection repose sur un mécanisme qui supprime automatiquement les objets devenus inutiles dans le cluster, en s’appuyant sur des références de propriété (owner references).  
2. Chaque objet peut désigner un ou plusieurs parents, ce qui permet à Kubernetes de déterminer la hiérarchie et de savoir quels éléments supprimer lorsqu’un parent disparaît.  
3. Si une ressource est marquée avec des **ownerReferences**, elle est considérée comme dépendante et sera éliminée dès que son propriétaire est effacé, à moins qu’on ne lui assigne une stratégie différente.  
4. Le **kube-controller-manager** inclut des contrôleurs qui détectent quand un objet parent se retire ou change, puis procèdent à la suppression ou l’orphelinage des enfants concernés.  
5. Les utilisateurs peuvent spécifier la stratégie de suppression, comme &quot;Foreground&quot;, &quot;Background&quot; ou &quot;Orphan&quot;, afin de maîtriser la séquence et les conséquences de la disparition d’un objet.  
6. En mode &quot;Foreground&quot;, l’objet parent ne sera réellement éliminé qu’une fois tous ses enfants effacés, assurant ainsi un nettoyage ordonné.  
7. En mode &quot;Background&quot;, le parent est retiré immédiatement, puis le contrôleur s’occupe de purger les descendants en arrière-plan.  
8. L’option &quot;Orphan&quot; désactive le garbage collection pour les enfants, ce qui peut être utile si on souhaite conserver ces objets afin qu’ils deviennent autonomes.  
9. Les finalizers viennent compléter ce dispositif en bloquant la suppression d’un objet tant qu’on n’a pas exécuté certaines actions obligatoires (libérer des ressources externes, par exemple).  
10. Ce système garantit que le cluster ne s’encombre pas de ressources obsolètes, tout en offrant suffisamment de souplesse pour respecter les cas particuliers de dépendances et d’orphelinage.  

