# INF2610 — Chapitre 1: Concepts généraux

1. **Définition d'un Système d'Exploitation (SE)**

   - **Matériel** : Comprend le CPU, la mémoire, le bus et les périphériques d’E/S.
   - **Logiciels** : Inclut les programmes d'application, les utilitaires et le SE lui-même.
   - **Rôle du SE** : Gère les composants matériels et fournit une interface pour le développement de programmes d'application et d'utilitaires via des services (appels système).

2. **Fonctions Principales d'un SE**
   - **Gestion des processus** : Exécution et gestion des programmes en cours d’exécution.
   - **Gestion de la mémoire** : Allocation et libération de mémoire.
   - **Gestion des fichiers et périphériques** : Accès, lecture, écriture des fichiers et contrôle des périphériques d’E/S.

### Interface avec le matériel

- **Composants matériels** : Chaque composant a son propre code pour fonctionner et interagir.
- **Interruptions** : Utilisées pour informer le processeur d'événements sans vérification continue.
  - **Interruptions matérielles** : Ex. horloges, fin de transferts E/S.
  - **Interruptions logicielles** : Ex. erreurs arithmétiques, défauts de page, appels système.

### Interactions utilisateur/système

- **Modes de fonctionnement**
  - **Mode noyau** : Réservé au SE, accès complet aux instructions.
  - **Mode utilisateur** : Pour les programmes et utilitaires, accès restreint pour assurer la protection.
- **Appel système** : Interruption logicielle activant le SE pour exécuter le service demandé (ex: `read`).

### Concepts de base

1. **Processus**
   - **Définition** : Programme en cours d'exécution avec son espace d'adressage et état.
   - **PCB** : Bloc de contrôle de processus contenant toutes les informations de gestion.
2. **Fichiers**

   - **Fichier ordinaire** : Données stockées, gérées par les systèmes de fichiers.
   - **Fichier spécial** : Représente des périphériques, répertoires, tubes.
   - **i-nœud** : Structure contenant les métadonnées du fichier sous UNIX/Linux.

3. **Mémoire virtuelle**
   - **Concept** : Permet l'exécution de processus sans charger tout l'espace d'adressage en mémoire physique.

### Évolution du mode d’exploitation

1. **Traitement par lots (1955-1965)**

   - Utilisation de cartes perforées, machines dédiées aux calculs et périphériques lents.

2. **Traitement par lots + multiprogrammation (1965-1980)**

   - Plusieurs travaux en mémoire, organisation en partitions.
   - Introduction des disques pour accès directs, contrôleurs DMA pour libérer le processeur.

3. **Va-et-vient (swapping)**

   - Retrait des travaux **bloqués** de la mémoire pour les remplacer par des travaux **prêts** à être exécutés.

4. **Multiprogrammation et partage de temps (1965-1980)**
   - Partage équitable du temps processeur, SE limite le temps d'allocation et suspend les travaux **bloqués**.

### Études de cas : Appel système `read(fd, &buffer, nbytes)`

- **Étapes** :
  1. Empilement des arguments.
  2. Fonction `read` de la librairie.
  3. Copie du numéro d'appel système.
  4. Basculement vers le mode noyau.
  5. Récupération et branchement à `sys_read`.
  6. Retour au mode utilisateur après l'exécution de `sys_read`.
