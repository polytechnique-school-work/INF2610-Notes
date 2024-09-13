# INF2610 — Chapitre 2: Processus

#### 1. Qu’est ce qu’un processus ?

- **Définition** : Un processus est un programme en cours d'exécution.
- **Identifiant** : Un processus a un numéro d’identification unique appelé PID (Process Identifier).
- **Représentation** : Un processus est représenté par une structure de données appelée PCB (Process Control Block).

  - Dans Linux, le PCB est une `task_struct`.
  - La table des processus est une liste doublement enchaînée de `task_struct`.

  **Différents états d'un processus** :

  - En cours d'exécution (running)
  - Prêt (ready)
  - Bloqué (blocked)
  - Terminé (terminated)
  - Zombie (zombie) : Processus terminé mais dont le père n'a pas encore récupéré l'état de terminaison (PCB non détruit).

**Le parent ultime : systemd (PID = 1):** il lance périodiquement des wait pour récupérer les états des processus terminés (et éviter les zombies). Il récupère les processus enfants sans parent.

#### 2. Bloc de contrôle de processus (PCB)

- **Composantes du PCB** : sauvegarder l'état d'un processus pour le reprendre plus tard au même point.

  - Compteur ordinal (adresse de la prochaine instruction).
  - Registres (sauvegarde des contenus des registres).
  - État du processus (prêt, en exécution, bloqué, zombie).
  - Priorité (ordonnancement des processus).
  - Espace d’adressage (code, données, pile d'exécution).
  - Fichiers ouverts (accessibles au processus).
  - Attributs Père-Fils (arborescence des processus).
  - Autres attributs nécessaires à la gestion du processus.

#### 3. Création de processus - Appel système fork

- **Fonctionnement** :

  - La création de processus se fait par duplication (Copy On Write - COW).
    **_Copy On Write_** : En gros on ne duplique pas le code tant que le processus n'écrit pas quelque chose de nouveau dessus. On duplique seulement les valeurs qui changent lorsqu'elles changent.
  - Le processus fils est un clone du père, exécutant le même code à partir de l'instruction qui suit l’appel à `fork()`.
  - **Retour** :
    - Le père reçoit le PID du fils.
    - Le fils reçoit 0.
  - **Échec** : La fonction `fork` retourne -1 en cas de manque de ressources.

  - **Différence entre le fork et son parent** :
    - Le père et le fils ont des PID différents.
    - Pour le processus parent, `fork` retourne le PID du fils.
    - Pour le processus fils, `fork` retourne 0.

#### 4. Terminaison de processus - Appel système \_exit

- **Fonction \_exit** :
  - Met fin à l'exécution du processus.
  - Enregistre l’état de terminaison dans le PCB.
  - Le processus devient un zombie jusqu’à ce que le père récupère cet état.
  - Types de terminaisons :
    - Normale : `_exit(value)` avec une valeur de retour.
    - Anormale : due à un signal (`SIGKILL`, `SIGINT`).

Attention, sous POSIX, la fin d'un parent ne met pas automatiquement fin à ses fils.

#### 5. Attente de la fin d’un processus fils - Appels système wait et waitpid

- **Fonctions** :

  - `wait(&status)`, `waitpid(pid, &status, options)` permettent à un père d'attendre la fin d'un fils et de récupérer son état de terminaison.
  - **Retour** :
    - PID du fils terminé.
    - -1 en cas d’échec.
  - **Macros** :

    - `WIFEXITED(status)` : vraie si la terminaison est normale.
    - `WIFSIGNALED(status)` : vraie si la terminaison est anormale.
    - `WEXITSTATUS(status)` : valeur de retour du processus.
    - `WTERMSIG(status)` : numéro du signal de terminaison.

  - **Récupérer l'information de status**:
    - `wait(&status)` : attend la fin d'un fils et récupère son état de terminaison.
    - Récupérer la vraie valeur du status: `WEXITSTATUS(status)`. Attention, le status doit tenir sur les 8 bits de poid fort.

#### 6. Remplacement du code d’un processus - Appels système exec

- **Fonctions exec** :
  - `execl`, `execlp`, `execle`, `execv`, `execvp`, `execve`, `execvpe` permettent de remplacer le code d'un processus.
  - **Succès** : Le processus continue avec le nouveau code sans retour à l'ancien.
  - **Échec** : Le processus continue avec le code courant.
- **Exemples** :
  - Remplacer le code courant par `ls -a` :
    - `execl("/bin/ls", "ls", "-a", NULL)`.
    - `execvp("ls", params[] ... )` avec `params[] = {"ls", "-a", NULL}`.

#### 7. Exemple d'utilisation de fork, exec et wait

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid;
    int status;

    pid = fork();

    if (pid == -1) {
        perror("fork");
        exit(EXIT_FAILURE);
    } else if (pid == 0) {
        // Code du fils
        printf("Fils : Je vais exécuter ls -a\n");
        execl("/bin/ls", "ls", "-a", NULL);
        perror("execl");
        exit(EXIT_FAILURE);
    } else {
        // Code du père
        printf("Père : Je vais attendre la fin du fils\n");
        wait(&status);
        printf("Père : Le fils a terminé\n");
        if (WIFEXITED(status)) {
            printf("Père : Le fils a terminé normalement avec le code %d\n", WEXITSTATUS(status));
        } else if (WIFSIGNALED(status)) {
            printf("Père : Le fils a terminé par un signal %d\n", WTERMSIG(status));
        }
    }

    return 0;
}
```
