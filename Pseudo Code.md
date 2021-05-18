# Starve Free Readers-Writers Problem
Problem deals with multiple processes reading from and writing to a shared resource synchronously where neither of them is starved for indefinate time. 

## Solution
Solution consists of pseudocodes for the problem and all of them are in c++ syntax.

#### Header files and global variables

```cpp
#include <bits/stdc++.h>
#include <pthread.h>
#include <semaphore.h>
#define ll long long
#define pb push_back
#define mp make_pair
#define modulo 1000000007
#define fast_io ios_base::sync_with_stdio(false),cin.tie(NULL),cout.tie(NULL)
using namespace std;
const ll N=1e6;

// Global variable declaration
int readers = 100;
int writers = 100;
int MAX_ID = 100;

// semaphore declarations
sem_t queue_of_rw;
sem_t resources;
sem_t mutex_of_r;

// critical data to be shared
int data = 1;

// readers counter
int r_count = 0;

```
#### Write shared code

```cpp
void *writer(void *x){
	
	// ENTRY section
	
	sem_wait(&queue_of_rw);
	sem_wait(&resources);
	sem_post(&queue_of_rw);
	

	// CRITICAL section
	
	data*=2;
	cout<<"Writer "<<*((int *)x)<<" modified data to "<<data<<"\n";


	// EXIT section
	sem_post(&resources);

}
```
#### Read shared code

```cpp
void *reader(void *x){
	
	// ENTRY section
	
	sem_wait(&queue_of_rw);
	sem_wait(&mutex_of_r);
	r_count++;
	if(r_count == 1)
		sem_wait(&resources);
	sem_post(&queue_of_rw);
	sem_post(&mutex_of_r);
	
	// CRITICAL section
	
	cout<<"Reader "<<*((int *)x)<<"read data as "<<data<<"\n";
	
	// EXIT section
	
	sem_wait(&mutex_of_r);
	r_count--;
	if(!r_count)
		sem_post(&resources);
	sem_post(&mutex_of_r);
	
}

```
#### Main function

```cpp
int main(){

	int identify[MAX_ID];
	
	int j=0;

	while(j < MAX_ID)
		identify[j] = ++j;
	
	// create tread
	pthread_t read[readers];
	pthread_t write[writers];
	
	// initialise semaphore 
	sem_init(&queue_of_rw,0,1);
	sem_init(&mutex_of_r,0,1);
	sem_init(&resources,0,1);

	j=0;
	while(j < readers){
		pthread_create(&read[j],NULL,(void *)reader,(void *)&identify[j]);
		j++;
	}

	j=0;
	while(j < writers){
		pthread_create(&write[j],NULL,(void *)writer,(void *)&identify[j]);
		j++;
	}

	j=0;
	while(j < readers){
		pthread_join(read[j],NULL);
		j++;
	}

	j=0;
	while(j < writers){
		pthread_join(write[j],NULL);
		j++;
	}

	//destroy threads at the end of execution
	sem_destroy(&mutex_of_r);
	sem_destroy(&queue_of_rw);
	sem_destroy(&resources);

	return 0;
	
}
```

## Explanation

### Semaphores

Three semaphores are used namely - 
- mutex_of_r - It provides mutual exclusion to r_count variable which counts number of readers in critical section.
- queue_of_rw - It maintains order of arrival of readers and writers.
- resources - It prevents readers and writers or multiple writers to be present in critical section at same time.

### Shared variables

- r_count - It counts number of readers present in critical section at given time.
- data - It is data which is shared among various threads.

### Reader function

- Entry section - It acquires required semaphores to enter critical section.
Reader tries to acquires queue_of_rw initially, if it is unavailable, reader is added to queue for the given semaphore.
After aquiring queue_of_rw, it tries to aquire mutex_of_r to modify r_count.
If it is the first reader, it tries to aquire resources to confirm there are no writers in the critical section.
It releases queue_of_rw and mutex_of_r before entering critical section.

- Critical section - Reader reads the shared data in this section.

- Exit section - It releases the aquired semaphores.
Reader tries to aquire mutex_of_r to modify r_count.
If it is the last reader, it releases resources.
It releases mutex_of_r at the end.

### Writer function

- Entry section - This section acquires required semaphores to enter critical section.
Writer tries to acquires queue_of_rw  initially, if it is unavailable, writer is added to queue for the given semaphore.
After aquiring queue_of_rw, it tries to aquire resources and enter critical section.
It releases queue_of_rw before entering critical section.

- Critical section - Writer modifies the shared data in this section.

- Exit section - This section releases the aquired semaphores.
Writer releases the resources SEMAPHORE.

### Main function

It creates required threads followed by initialisation of semaphores.
It then initialises reader threads with reader function and writer threads with writer function.
Then, all threads are joined back to the parent thread.
After the execution, all initialized semaphores are destroyed.
