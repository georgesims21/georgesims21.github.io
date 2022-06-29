# pthreads in C

This tutorial assumes you know the basic concepts of multithreading. It is based on [this short video series by Dr. Brian Fraser](https://www.youtube.com/watch?v=ynCc-v0K-do&t=5s)

## Creating single threads 

The pthread library (POSTIX thread) allows C programmers to use multithreading capabilities. First you must define the <pthread.h> header, and link -pthread when compiling the program. 
The function to create a thread is the pthread_create():
```C
<pthread.h>
int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg);
```
This takes 4 different parameters:
* The pthread_t thread variable you wish to use
* Any special attributes you wish this thread to have contained in the pthread_attr_t struct
* The routine you want this thread to do. This will be a function pointer, and this pointer should return a void* as well as accepting void* as params.
* Finally any arguments you wish to pass to this routine

From this it is important to point out that threads can only run routines which return and take void pointer types. This means casting must be done within the routine if you wish to use particular variables.

```C
#include <stdio.h>
#include <pthread.h>
/*
* Simply allow a thread to increment a global variable
*/

int limit = 0; // global as threads share address space

void* run(void* arg) {

    int *lim = (int *)arg; // must cast the void pointer
    // int lim = *(int *)arg; would dereference and place value in lim instead
    printf("Thread running, before incrementing limit is: %d\n", limit);

    *lim = *lim + 1;
    printf("Thread running, after incrementing limit is: %d\n", limit);

    pthread_exit(0); 
}

int main(void) {

    printf("In main before thread, limit is: %d\n", limit);

    // create thread
    pthread_t tid;

    // create attributes - can also pass NULL if not needed
    pthread_attr_t attr;
    // init attributes
    pthread_attr_init(&attr);

    // create and run thread
    pthread_create(&tid, &attr, run, &limit); // run is function pointer as without
                                              // paranthesis it points to starting 
                                              // address of function

    // wait for thread to finish
    pthread_join(tid, NULL); // if param 2 is NULL, main ignores any return value

    printf("In main after thread, limit is: %d\n", limit);

    return 0;
}      
```

The output of such program should look like this:
```bash
➜  ~ gcc -Wall threads.c -lpthread -o threads
➜  ~ ./threads  
In main before thread, limit is: 0
Thread running, before incrementing limit is: 0 # Thread is running 
Thread running, after incrementing limit is: 1  # while main waits
In main after thread, limit is: 1   
```
It is important to note that without the pthread_join function in this program, the main thread would simply print the statement and return without giving the thread a chance.

## Returning a value from the run function
In the previous example a global variable was used. If you want to return a variable from the function you must use pthread_exit(<return value>) and catch it when using pthread_join like so:

```C
void* run(void* arg) {
    int value;
    ...
    pthread_exit(&value); // auto-converted to void* by method
}

int main(void) {
    ...
    int *retval;
    pthread_join(tid, (void **)&retval); // join returns ptr to ptr of type void
    ...
}      
```

## Giving multiple parameters to the run function
The problem with having the void* as a parameter for the run function is giving more than one argument. The only way to pass more than one would be to create a struct containing all of the variables that you wish to pass to the function. It is good practice to name this similar to the function itself:
```C
...
struct run_struct { // name matching function
    int limit; 
    int answer;
};

void* run(void* arg) {
    struct run_struct *args = (struct run_struct *) arg; // typecast to use vars

    args->answer = args->limit++;
    ...
}

int main(void) {
    ...
    struct run_struct rs = {5, 0};
    ...
    pthread_create(&tid, &attr, run, &rs); // here the struct is given as param
    ...
    printf("In main after thread, answer is: %d\n", rs.answer);
 }
```

## Creating multiple threads
Multiple threads can be created using a for loop:
```C
...

pthread_t tids[10]; // create array of threads size 10

for(int i = 0; i <= 10; i++) {
    // this can be done here for each or you can use one pthread_attr_t for
    // all threads, if you wish them to have the same attributes
    pthread_attr_t attr;
    pthread_attr_init(&attr);

    pthread_create(&tids[i], &attr, run, &limit); // create 10 threads
}

for(int i = 0; i <= 10; i++) {
    pthread_join(tids[i], NULL); // make main wait for all of them
}

...
```

## Multiple threads having own param variables
```C
#include <stdio.h>
#include <pthread.h>
/*
* Allow multiple threads to increment their own struct variable and save into the other
* then main waits and prints. - Doesn't increment but too lazy to change
*/

struct run_struct {
    int limit;
    int answer;
    int tn;
};

void* run(void* arg) {

    struct run_struct *args = (struct run_struct *) arg;
    printf("Thread[%d] running, before incrementing answer is: %d\n", args->tn, 
                                                                      args->answer);

    args->answer = args->limit++;
    printf("Thread[%d] running, after incrementing answer is: %d\n", args->tn, 
                                                                     args->answer);

    pthread_exit(0); 
}

int main(void) {

    struct run_struct rs[10]; // to give all threads own variables for function
    for(int i = 0; i < 10; i++) {
        rs[i].limit = 5;
        rs[i].answer = 0;
        rs[i].tn = 0;
    }

    // create threads
    pthread_t tids[10];

    pthread_attr_t attr;
    pthread_attr_init(&attr);

    // create and run threads with threads own param struct
    for(int i = 0; i < 10; i++) {
        rs[i].tn = i; // give thread it's own tid
        pthread_create(&tids[i], &attr, run, &rs[i]); // each thread gives own 
                                                      // struct as param pointer
    }

    // wait for thread to finish
    for(int i = 0; i < 10; i++) {
        pthread_join(tids[i], NULL); // if param 2 is NULL, main ignores any return value
        printf("In main after thread[%d], limit is: %d\n", i, rs[i].answer);
    }


    return 0;
}       
```
If you compile and run the above program, you will see that the order of execution is only kept when main is printing the variables due to the wait function, but the threads themselves can finish before the others without order.. but main won't print information for a thread if the one before hasn't finished. Example output:
```bash
➜  ~ gcc -Wall threads.c -lpthread -o threads
➜  ~ ./threads
Thread[0] running, before incrementing answer is: 0
Thread[0] running, after incrementing answer is: 5
Thread[1] running, before incrementing answer is: 0
Thread[4] running, before incrementing answer is: 0 # not in order 
Thread[6] running, before incrementing answer is: 0
Thread[6] running, after incrementing answer is: 5
Thread[2] running, before incrementing answer is: 0
Thread[5] running, before incrementing answer is: 0
Thread[5] running, after incrementing answer is: 5
In main after thread[0], limit is: 5 # Thread[0] has finished so main can print it
Thread[4] running, after incrementing answer is: 5
Thread[8] running, before incrementing answer is: 0
Thread[8] running, after incrementing answer is: 5
Thread[2] running, after incrementing answer is: 5
Thread[3] running, before incrementing answer is: 0
Thread[3] running, after incrementing answer is: 5
Thread[1] running, after incrementing answer is: 5
Thread[9] running, before incrementing answer is: 0
Thread[9] running, after incrementing answer is: 5
In main after thread[1], limit is: 5
In main after thread[2], limit is: 5
In main after thread[3], limit is: 5
In main after thread[4], limit is: 5
In main after thread[5], limit is: 5
In main after thread[6], limit is: 5 # 8 will not be printed until 7 is finished
Thread[7] running, before incrementing answer is: 0
Thread[7] running, after incrementing answer is: 5
In main after thread[7], limit is: 5 # now 7 is done 8 can proceed
In main after thread[8], limit is: 5
In main after thread[9], limit is: 5
```

## Thread syncronization
### Mutexes
When threads modify the same address or variable, there is no telling which will go first and most likely without any form of syncronisation you will get race conditions.It is important to correctly identify the critical section(s) within the code and add some form of syncronisation to that area, so only one thread at a time can access this area. A mutex (or mutual exclusion) allows for such a scenario.

```C
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER; // macro used for init of mutex

...
// critical section start
pthread_mutex_lock(&mutex);
... // only 1 thread can acess this area of code
pthread_mutex_unlock(&mutex);
// critical section end
```
Keep in mind however that mutexes should only be used when needed as they make expensive calls to the kernel.
Further reading: [Implementing thread pools in C](https://nachtimwald.com/2019/04/12/thread-pool-in-c/)

