# INF2610 — Chapitre 5: Synchronisation de processus

### 1. Les conditions de concurrence

Étant donné qu'on partage les ressources entre plusieurs processus, il faut s'assurer que les processus ne vont pas entrer en conflit.

Lorsqu'on écrit nos programmes, on essaie de concentrer le code qui entre dans une condition de concurrence dans une section appelée section critique pour ainsi minimiser le temps d'exécution de cette section.

#### 1.1 Les 4 conditions de la section critique

1. Deux processus ne peuvent pas être, en même temps, dans leurs sections critiques;
2. Aucune hypothèse ne doit être posée sur les vitesses relatives des processus, le nombre de processeurs, etc. (On ne peut pas poser l'hypothèse que le processeur 0 est plus rapide que le processeur 1, qu'il va s'exécuter avant, que ça va s'exécuter en concurrence ou en parallèle, etc.);
3. Aucun processus en dehors de sa section critique ne doit bloquer les autres processus d'entrer dans leurs sections critiques.
4. Aucun processus ne doit attendre trop longtemps avant d'entrer en section critique (attente bornée).

![alt text](image-3.png)

#### 1.2 Le problème de la section critique

```cpp

while (true) {
    // Section d'initialisation
    // ...

    Section critique
    ...

    // Section de terminaison
    // ...
}

```

1. **Un seul processus** à la fois en section critique;
2. **Aucune hypothèse** quant aux vitesses relatives des processus et au nombre de processeurs;
3. **Aucun blocage** par un processus qui n'est pas dans sa section critique;
4. **Pas d'attente excessive** subie par un processus avant d'entrer en section critique.

Donc finalement, on veut que les processus arrivent à entrer en section critique dans l'ordre et sans exécuter du code inutile (pour optimiser le temps).

#### Solution de Peterson

```cpp
// initialisation

bool flag[2];
flag[0] = false;
flag[1] = false;
int tour;

// Fil d'exécution 0

while (true) {
  flag[0] = true;
  tour = 1;

  // Scrutation pour vérifier si le fil d'exécution 1 est en section critique (on évite de rentrer en section critique si le fil d'exécution 1 est en section critique)
  while(flag[1] && tour == 1);
  // Section critique
  flag[0] = false;
  // Section restante
}

// Fil d'exécution 1

while (true) {
  flag[1] = true;
  tour = 0;
  while(flag[0] && tour == 0);
  // Section critique
  flag[1] = false;
  // Section restante
}
```

### 2. Masquage des interruptions

```c
// Désactive les interruptions
unsigned long flags;
local_irq_save(flags);

section_critique();

// Réactive les interruptions
local_irq_restore(flags);
```

- Tant que les interruptions ne sont pas réactivées, le processeur (exécutant le code) ne peut pas réagir à une interruption..
- Cette solution ne fonctionne pas dans un système de multiprocesseur.
- C'est dangeureux, donc à éviter.
- Réservé au noyau.

### 3. Fonctions atomiques

# Fonctions atomiques en C et Java

## Qu'est-ce qu'une fonction atomique en C ?

Les fonctions atomiques en C permettent d'exécuter des opérations de manière **atomique**, c'est-à-dire sans être interrompues. Elles sont utilisées pour éviter les **conditions de concurrence** en environnement multithread et garantir la **cohérence des données**.

Exemple : `atomic_fetch_add` permet d'ajouter une valeur à une variable de manière atomique, sans que plusieurs threads interfèrent entre eux.

### Exemple de code en C :

```c
#include <stdatomic.h>
#include <stdio.h>
#include <pthread.h>

atomic_int counter = 0;

void* increment_counter(void* arg) {
    for (int i = 0; i < 1000; ++i) {
        atomic_fetch_add(&counter, 1);
    }
    return NULL;
}

int main() {
    pthread_t t1, t2;

    pthread_create(&t1, NULL, increment_counter, NULL);
    pthread_create(&t2, NULL, increment_counter, NULL);

    pthread_join(t1, NULL);
    pthread_join(t2, NULL);

    printf("Counter value: %d\n", counter);
    return 0;
}
```

### Qu'est-ce qu'une fonction atomique en Java ?

Les fonctions atomiques en Java permettent d'exécuter des opérations de manière **atomique**, c'est-à-dire sans être interrompues. Elles sont utilisées pour éviter les **conditions de concurrence** en environnement multithread et garantir la **cohérence des données**.

Exemple : `AtomicInteger` permet d'ajouter une valeur à une variable de manière atomique, sans que plusieurs threads interfèrent entre eux.

### Exemple de code en Java :

```java
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicCounter {
    private AtomicInteger counter = new AtomicInteger(0);

    public void increment() {
      counter.incrementAndGet();
    }

    public int getCounter() {
      return counter.get();
    }
}

public class Main {
    public static void main(String[] args) {
        AtomicCounter counter = new AtomicCounter();

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter.increment();
            }
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter.increment();
            }
        });

        t1.start();
        t2.start();

        try {
            t1.join();
            t2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Counter value: " + counter.getCounter());
    }
}
```

### 4. Verrous Actifs

Un verrou est une sorte de jeton d'accès aux sections critiques. Seul le processus/thread qui detient le verrou va pouvoir exécuter sa section critique. Les autres doivent attendre que le processus/thread relâche le verrou. Un verrou a deux états : libre et pris (entier 0 ou 1). Libre si 0.

Comme un verrou est partagé entre plusieurs processus/threads, la lecture et la modification de l'état d'un verrou doivent être réalisées en exclusion mutuelle. Le verrou fonctionne par scrutation.

```c
// Ne peut pas y avoir d'interruption au milieu de l'instruction (le noyau l'empêche).
int TSL(int &lock) {
  int old_value = lock;
  lock = 1;
  return old_value;
}
```

On remarque ici que cette fonction retourne l'ancienne valeur du verrou. On peut donc savoir si le verrou est libre ou pris en comparant cette valeur avec 0. Ça permet de savoir si on peut entrer en section critique ou non.

Donc on vient mettre un 1 dans le verrou (s'il était déjà pris, ça change rien). Si le verrou était déjà pris, la fonction TSL retourne 1 et donc on sait qu'il faut attendre. Sinon, on peut entrer en section critique (et on a déjà mis un 1 dans le verrou). On veut pas perdre notre place.

Important de savoir : c'est **ATOMIQUE**, donc tout la fonction TSL est exécuté sans interruption.

```c
while (TSL(lock) == 1);

// Section critique

lock = 0;

// Section restante
```
