# Linux-IPC-Semaphores
Ex05-Linux IPC-Semaphores

# AIM:
To Write a C program that implements a producer-consumer system with two processes using Semaphores.

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - Sempahores

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:

## Write a C program that implements a producer-consumer system with two processes using Semaphores.
#include <stdio.h>      /* standard I/O routines */
#include <stdlib.h>     /* rand() and srand() */
#include <unistd.h>     /* fork(), sleep() */
#include <time.h>       /* nanosleep() */
#include <sys/types.h>  /* type definitions */
#include <sys/ipc.h>    /* IPC structures */
#include <sys/sem.h>    /* semaphore functions */

/* Number of loops to perform */
#define NUM_LOOPS 20

/* union semun for semctl */
#if defined(_GNU_LIBRARY_) && !defined(_SEM_SEMUN_UNDEFINED)
/* union semun is defined in <sys/sem.h> */
#else
union semun {
    int val;                  /* value for SETVAL */
    struct semid_ds buf;     / buffer for IPC_STAT, IPC_SET */
    unsigned short array;    / array for GETALL, SETALL */
    struct seminfo __buf;    / buffer for IPC_INFO */
};
#endif

int main(int argc, char *argv[])
{
    int sem_set_id;        /* ID of the semaphore set */
    union semun sem_val;   /* value for semctl */
    int child_pid;         /* child PID */
    int i;                 /* loop counter */
    struct sembuf sem_op;  /* semaphore operation struct */
    struct timespec delay; /* for nanosleep */

    /* Create a private semaphore set with 1 semaphore */
    sem_set_id = semget(IPC_PRIVATE, 1, 0600);
    if (sem_set_id == -1) {
        perror("semget");
        exit(1);
    }
    printf("Semaphore set created with ID %d\n", sem_set_id);

    /* Initialize semaphore to 0 */
    sem_val.val = 0;
    if (semctl(sem_set_id, 0, SETVAL, sem_val) == -1) {
        perror("semctl");
        exit(1);
    }

    /* Fork child process */
    child_pid = fork();
    if (child_pid == -1) {
        perror("fork");
        exit(1);
    }

    switch (child_pid) {
        case 0: /* child process - consumer */
            for (i = 0; i < NUM_LOOPS; i++) {
                /* Wait (decrement) semaphore */
                sem_op.sem_num = 0;
                sem_op.sem_op = -1;
                sem_op.sem_flg = 0;
                semop(sem_set_id, &sem_op, 1);

                printf("Consumer: %d\n", i);
                fflush(stdout);
            }
            exit(0); /* child exits */
            break;

        default: /* parent process - producer */
            for (i = 0; i < NUM_LOOPS; i++) {
                printf("Producer: %d\n", i);
                fflush(stdout);

                /* Signal (increment) semaphore */
                sem_op.sem_num = 0;
                sem_op.sem_op = 1;
                sem_op.sem_flg = 0;
                semop(sem_set_id, &sem_op, 1);

                /* Optional small delay sometimes */
                if (rand() > 3 * (RAND_MAX / 4)) {
                    delay.tv_sec = 0;
                    delay.tv_nsec = 10;
                    // nanosleep(&delay, NULL);
                    sleep(1);
                }
            }

            /* Remove semaphore set */
            if (semctl(sem_set_id, 0, IPC_RMID, sem_val) == -1) {
                perror("semctl IPC_RMID");
            }
            break;
    }

    return 0;
}




## OUTPUT
$ ./sem.o 
<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/fdd1270a-7b0c-40cc-a2e5-96af48c573e6" />



$ ipcs:
<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/6efec6ac-020c-4bb0-890c-f6d0f9bd9d28" />






# RESULT:
The program is executed successfully.
