
# practice-4

 # Практична робота №4
## Тема: Робота з динамічною пам’яттю в C (malloc, realloc, free, memory pool)

##Завдання 4.1 — malloc і size_t:

###Створення:

nano task4_1.c

### Код:

#include <stdio.h>
#include <stdlib.h>
#include <limits.h>
int main() {
size_t max = (size_t)-1;
printf("Max size_t: %zu\n", max);
printf("Trying malloc...\n");
void *ptr = malloc(max);
if (!ptr) {
printf("malloc failed (expected)\n");
} else {
printf("malloc succeeded (unexpected)\n");
free(ptr);
}
return 0;
}

### Виконання:

gcc task4_1.c -o task4_1
./task4_1


## Завдання 4.2 — переповнення int:

### Створення: 

nano task4_2.c

### Код:

#include <stdio.h>
#include <stdlib.h>
int main() {
int xa = 50000;
int xb = 50000;
int num = xa * xb; // overflow!
printf("num = %d\n", num);
void *ptr = malloc(num);
if (!ptr) {
printf("malloc failed\n");
} else {
printf("malloc success\n");
free(ptr);
}
return 0;
}


### Виконання:

gcc task4_2.c -o task4_2
./task4_2

## Завдання 4.3 — malloc(0):

### Створення:

nano task4_3.c

### Код:

#include <stdio.h>
#include <stdlib.h>
int main() {
void *ptr = malloc(0);
printf("ptr = %p\n", ptr);
if (ptr)
free(ptr);
return 0;
}

### Виконання:

gcc task4_3.c -o task4_3
ltrace ./task4_3

## Завдання 4.4 — помилка з free в циклі:

### Створення:

nano task4_4_bad.c

### Код:

#include <stdio.h>
#include <stdlib.h>
int main() {
int *ptr = NULL;
for (int i = 0; i < 5; i++) {
if (!ptr)
ptr = malloc(10 * sizeof(int));
ptr[i] = i;
printf("%d\n", ptr[i]);
free(ptr);
}
return 0;
}

### Виконання:

gcc task4_4_bad.c -o task4_4_bad
./task4_4_bad

### Створення:

nano task4_4_good.c

### Код:

#include <stdio.h>
#include <stdlib.h>
int main() {
int *ptr = malloc(5 * sizeof(int));
for (int i = 0; i < 5; i++) {
ptr[i] = i;
printf("%d\n", ptr[i]);
}
return 0;
}

### Виконання:

gcc task4_4_good.c -o task4_4_good
./task4_4_good


## Завдання 4.5 — realloc fail:

### Створення:

nano task4_5.c

### Код:

#include <stdio.h>
#include <stdlib.h>
int main() {
size_t big = (size_t)-1;
void *ptr = malloc(100);
void *newptr = realloc(ptr, big);
if (!newptr) {
printf("realloc failed, original still valid\n");
free(ptr);
}
return 0;
}


### Виконання:

gcc task4_5.c -o task4_5
./task4_5


## Завдання 4.6 — realloc(NULL / 0):

### Створення:

nano task4_6.c

### Код:

#include <stdio.h>
#include <stdlib.h>
int main() {
void *a = realloc(NULL, 100);
printf("a = %p\n", a);
void *b = realloc(a, 0);
printf("b = %p\n", b);
return 0;
}

### Виконання:

gcc task4_6.c -o task4_6
./task4_6


## Завдання 4.7 — reallocarray:

### Створення:

nano task4_7.c

### Код:

#include <stdio.h>
#include <stdlib.h>
struct sbar {
int x;
};
int main() {
struct sbar *ptr, *newptr;
ptr = calloc(1000, sizeof(struct sbar));
newptr = reallocarray(ptr, 500, sizeof(struct sbar));
if (!newptr) {
free(ptr);
return 1;
}
free(newptr);
return 0;
}


### Виконання:

gcc task4_7.c -o task4_7
./task4_7

## Завдання 4.8 ВАРІАНТ 9 — Memory Pool:

### Створення:

nano memory_pool.c

### Код:

#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#define POOL_SIZE 10000
#define BLOCK_SIZE 64
char pool[POOL_SIZE][BLOCK_SIZE];
int used[POOL_SIZE];
void* pool_alloc() {
for (int i = 0; i < POOL_SIZE; i++) {
if (!used[i]) {
used[i] = 1;
return pool[i];
}
}
return NULL;
}
void pool_free(void *ptr) {
for (int i = 0; i < POOL_SIZE; i++) {
if (pool[i] == ptr) {
used[i] = 0;
return;
}
}
}
int main() {
clock_t start, end;
start = clock();
for (int i = 0; i < 100000; i++) {
void *p = pool_alloc();
if (p) pool_free(p);
}
end = clock();
printf("Pool time: %lf sec\n",
(double)(end - start) / CLOCKS_PER_SEC);
return 0;
}



### Створення:

nano malloc_test.c


### Код:

#include <stdio.h>
#include <stdlib.h>
#include <time.h>
int main() {
clock_t start = clock();
for (int i = 0; i < 100000; i++) {
void *p = malloc(64);
free(p);
}
clock_t end = clock();
printf("malloc time: %lf sec\n",
(double)(end - start) / CLOCKS_PER_SEC);
return 0;
}



### Виконання:

gcc memory_pool.c -o pool
gcc malloc_test.c -o malloc_test

./pool
./malloc_test

## Висновок

У ході практичної роботи було досліджено особливості роботи з динамічною пам’яттю в мові C, зокрема функцій malloc(), calloc(), realloc() та free(). Розглянуто поведінку програми при переповненні цілочисельних типів, виклику malloc(0), а також при невдалому виділенні пам’яті через realloc(). Окремо було реалізовано власний memory pool і проведено порівняння його продуктивності зі стандартним malloc(). Результати показали, що помилки керування пам’яттю можуть призводити до непередбачуваної поведінки програми, тоді як використання оптимізованих підходів, таких як memory pool, дозволяє підвищити ефективність роботи з пам’яттю у спеціалізованих задачах.
