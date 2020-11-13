# Chap 6. Synchronization

## 6.1 Background

- process 는 concurrently 하게 또는 parallel 하게 실행 된다.
- Concurrent 또는 parallel 한 access는 데이터의 consistency에 문제가 생길 수 있다.
- 공유 데이터에 접근하는 순서를 정해야 데이터의 consistency를 유지시킬 수 있다.

```c
/* Producer */
while (true) {
		/* produce an item in next_produced */    

		while (counter == BUFFER SIZE) ;
				/* do nothing */
		buffer[in] = next_produced;
		in = (in + 1) % BUFFER_SIZE;  // ring buffer이기 때문에 모듈러 연산
		counter++
}

```

```c
/* Consumer */
while (true) {
		while (counter == 0) ;
			/* do notiong */
		next_consumed = buffer[out];
		out = (out + 1) % BUFFER_SIZE; // ring buffer이기 때문에 모듈러 연산
		counter--;

		/* consume the item in next_consumed */
}
```

- Producer와 Consumer가 counter라는 변수를 공유함
- counter 변수가 consistency를 유지할 수 있나?
- Serial execution일 때는 차례대로 두 프로세스가 변수에 접근하므로 consistency가 유지된다.
- 하지만 concurrunt(parallel)한 공유 데이터의 접근은 consistency를 깨트릴 수 있다.
- 공유데이터를 사용하는 영역을 critical section이라고 함

### Race Condition

- 읽기만 한다(Read) → 문제를 발생시키지 않음 (No problem)
- write에 대해서 문제 발생
- counter++ 과 counter—는 atomic instruction이 아님
- counter++
    - register1 = counter (load)
    - register1 = register1 + 1 (add)
    - counter = register1 (store)
- counter++ 과 counter— 가 동시에 실행된다면 두 프로세서의 공유 데이터 접근은 경쟁 상태(race condition)

## 6.2 Critical Section Problem

- n개의 프로세스 p_0 p_1 .... p_n-1 에 대해서...
- critical section이란 공유하는(common) 변수, updating table, writing file 등.. 공유하는 모든 것
- 하나의 프로세스가 critical section에 있으면 다른 프로세스는 critical section에 들어갈 수 없다.
- critical section에 들어가기 전에 하는 조치를 entry section 나와서 하는 조치를 remainder section

### Solution to Critical-Section Problem

1. Mutual Exclusion : 어떤 프로세스가 critical section에 들어가면 다른 프로세스는 그 critical section에 접근할 수 없다.
2. Progress : 무한정 앞의 프로세스를 기다릴 수는 없다. 앞의 프로세스가 나온지 모르고 계속 대기해서는 안된다. 나가는 프로세스가 무언가 조치를 취해서 내가 들어갈 수 있게 해줘야 한다.
3. Bounded Waiting : starvation이 생기면 안된다. 아무리 운이 없어도 유한한 시간이 지나면 접근할 수 있음을 보장해야 한다. 몇 번째 순서 안에는 critical section에 들어갈 수 있음을 보장

### kernel의 접근

system call을 통해 kernel의 데이터를 쓰기도 함(user process는 여러개지만 kernel은 하나, 그래서 유저는 kernel의 데이터를 공유), 

- preemptive kernel : 프로세스가 system call로 kernel mode에 접근할 때 preemption을 허용 → race condition이 생길 수 있다. → 복잡함, 구현의 난도가 높음
- non-preemptive kernel : 한 프로세스가 kernel mode에 접근하면 preemption을 하지 않는다(한 프로세스가 진입하면 다른 프로세스가 중단시킬 수 없다.) → kernel의 영역이 serial execution → race condition이 발생하지 않음(multicore일 경우에는 생길 수도 있다.)

## 6.3 Peterson's Solution

- 두 프로세스가 경쟁하는 경우의 솔루션
- load와 store instruction이 atomic 하다고 가정, interrupt 당하지 않음
- 두 개의 변수

```c
int turn;  // 누가 critical section에 들어갈 차례인지
Boolean flag[2]  // 들어가기 희망함을 알려줌. flag[i] = true면 프로세스 i가 critical section에 들어갈 준비가 되었음을 내포함
```

```c
/* process i */

do {
		flag[i] =  true;
		turn = j;
		while (flag[j] && turn == j); // j를 기다림

		critical section		

		flag[i] = false; // i는 이제 빠진다
				remainder section
} while (true);
```

## 6.4 Synchronization Hardware

- atomic하게 동작하도록 하드웨어적으로 보장 (atomic한 hardware instruction)
- 사전에 lock을 걸기 (locking)
- uniprocessors - disable interrupt : multiprocessor 환경에서 비효율

```c
do {
	acquire lock
		critical section
	release lock
		remainder section
} while (TRUE);
/*
 lock 획득과 해제를 atomic instruction으로 처리하면
 자연스럽게 critical section의 mutual exclusion을 보장
*/
```

```c
// test_and_set Instruction

boolean test_and_set (boolean *target) // target은 일종의 lock, target
{
		boolean rv = *target;  // target이 false로 들어오면 우선 rv에 백업
		*target = TRUE;  // target을 true로 만들어 lock을 검
		return rv; // 현재의 lock 상태 return
}

// 하드웨어적으로 atomic한 instruction으로 동작
```

```c
// test_and_set 적용 -> 현재 lock 상태 확인

do {
		while (test_and_set(&lock)); 
			/* do nothing */   // lock이 걸려있으면 아무것도 안함. busy waiting
		/* 위 loop를 빠져나왔다는 것은 현재 lock이 false고
		잽싸게 true로 바꾸고 나왔다는 것 (lock을 걸었다.) */
		/* 
		critical section 
		*/
		lock = false;  
		/* 이게 false가 되면 test_and_set()이 
		false를 return, 비로소 다른 프로세스가 위의 loop을 빠져나올 수 있다. (lock 해제) */
		/* remainder section */
} while (true);
```

- Bounded waiting은 보장이 안됨
- 그럼 어떻게 할까?

```c
// compare_and_swap Instruction
// expected가 lock이 안걸려있으면 value를 new_value로 바꿔줌
int compare_and_swap(int *value, int expected, int new_value) {
		int temp = *value;
		if (*value == expected)
			*value = new_value;
		return temp;  // 원래 백업했던 value를 return
}

//위 instruction을 사용한 예제

do {
		while (compare_and_swap(&lock, 0, 1) != 0) // lock이 false이면 critical section 진입, true이면 busy waiting
		; /* do nothing */
		/* critical section */
		lock = 0; // critical section이 끝나고 lock을 풀어줌
		/* remainder section */
} while (true);
```

```c
/* Bounded-waiting Mutual Exclusion with test_and_set */

do {
		waiting[i] = true; // critical section에 들어가고 싶다는 의사를 밝힘
		key = true; // 초기화
		while (waiting[i] && key) // 여기서 대기
				key = test_and_set(&lock); // lock이 안 걸려있으면 key는 false, lock은 true가 된다. 다른 프로세스 못들어오는 상태.
		waiting[i] = false;
		/* critical section */
		j = (i + 1) % n;
		while ((j != i) && !waiting[j]) // waiting[j]가 true면 빠져나옴 -> 기다리는 프로세서를 찾았다
				j = (j + 1) % n;
		if (j == i) // 한바퀴 돌아서 더 이상 할 일이 없음
				lock = false; // 기다리는 프로세서가 없으면 그냥 lock을 풀고 나온다.
		else
				waiting[j] = false; 
				/* waiting[j]가 false가 되어 위의 첫번째 while문을 빠져나가 
					j가 critical section에 진입할 수 있게됨 */
	 	/* remainder section */
} while (true);

/* 전체 프로세서가 n개라면 적어도 n번 안에는 진입 가능 */
```

## 6.5 Mutex Locks

- OS안에서 library를 제공하여 critical section problem을 쉽게 해결하도록 함 → Mutex와 Semaphore
- acquire(), release() 각각 lock을 걸고 풀어주는 함수 제공, 둘 다 atomic함을 보장
- mutex lock의 단점 : busy waiting이 발생

```c
acquire() {
		while (!available)
				; /* busy wait */
		available = false;
}

release() {
		available = true;
}

do {
	acquire lock
			critical section
	release lock
			remainder section
} while (true);
```

## 6.6 Semaphores

- 모든 프로세스가 공유하는 integer 변수
- standard operation : wait()와 signal() 각각 P( ), V( ) 또는 P연산, V연산이라고도 한다.

```c
/* 
	S의 초기값이 중요
	초기값이 1이면 wait의 처음 while loop은 false -> S는 0으로 바뀜
	이 상태에서 다른 프로세서가 wait()를 부르면 while loop를 빠져나오지 못함
	signal()을 호출하면 다시 S가 0에서 1이 된다.
	그러면서 기다리던 프로세서가 접근할 수 있게 된다. 접근할 때는 또 다시 0이 됨
	이건 binary semaphore의 예시! (S의 초기값 1)
*/
wait (S) {
		while (S <= 0)
				; // busy wait
		S--;
}

signal (S) {
		S++;
}
```

### Semaphore Usage

- Counting semaphore : S가 1이 아닐 때; critical section에 들어가 있는 프로세스가 몇 개인지 확인할 때 사용
- Binary semaphore : S가 1일 때; mutex(mutual exclusion 즉, 하나의 프로세스 접근만 허용) lock와 같다.
- P1이 P2 전에 실행되도록 하고 싶을 때는? (S1 → S2)

```c
P1:
		S1; //statement
		signal(synch);
P2 :
		wait(synch);
		S2; //statement

/*
	synch를 0으로 초기화하면 P2의 wait()가 기다리다가(busy waiting)
	P1의 S1 실행되고 signal()을 보내주어 1이되면 비로소 P2의 S2가 실행될 수 있다.
*/
```

### Semaphore Implementation

- 두 개 이상의 프로세서가 wait() 또는 signal()에 진입할 수 없다(wait, signal함수 자체가 critical section)
- busy waiting(spin lock)의 구현
- busy waiting에 기반한 solution은 구현이 쉽다는 장점이 있지만(코드 간단), 하지만 상황에 따라 비효율적일 수 있다.
- busy waiting 없이 구현하려면? → block(waiting queue 이용)과 wakeup operation
    - waiting queue에는
- S 값이 -5면? waiting queue에 대기하고 있는 프로세스가 5개 있다는 의미

```c
typedef struct{
		int value;
		struct process *list;
} semaphore;

wait(semaphore *S) {
		S->value--;
		if (S->value < 0) {  // 최초 프로세스는 여기서 안 걸리고 진입한다. (1 -> 0)
				add this process to S->list; 
				block();
		}
}
signal(semaphore *S) {
		 S->value++;
		if (S->value <= 0) { /* 현재 나가는 프로세스 외에 critical section 진입을 기다리는 프로세스가 리스트에 있다. */
				remove a process P from S->list;
				wakeup(P);  // 하나를 list에서 가져와 wakeup 시킨다.
		}
}
```

- 구현이 복잡하지만 효과적인 CPU 자원 활용 가능

### Deadlocks and Starvation

- S, Q를 1로 초기화 하자

```c
// S, Q동시에 가지고 있어야 진입 가능

P0
wait(S);
wait(Q);
critical section
signal(S);
signal(Q);

P1
wait(Q);
wait(S);
critical section
signal(Q);
signal(S);

// P0, P1가 서로 가진 자원을 요구
// Deadlock!
```

- Starvation - indefinite blocking → 나보다 우선순위가 높은 프로세서에 의해 critical section 접근이 계속 지연됨
- Priority Inversion - 프로세스의 우선순위가 있으면(priority-driven scheduling = real-time scheduling이면) 이 문제 발생 → priority-inheritance protocol로 해결 가능

## 6.7 Classic Problems of Synchronization

### Bounded-Buffer Problem

- n개의 buffer는 각각 하나의 item만 가질 수 있다.
- Semaphore mutex 1로 초기화(binary semaphore, critical section의 mutually exclusion보장을 위해 사용)
- Semaphore full 0으로 초기화(counting semaphore)
- Semaphore empty n으로 초기화(counting semaphore)

```c
// Producer : buffer를 채우는 프로세스
// buffer에 접근할 때는 동시에 producer나 consumer가 접근하면 안된다.

do {
		...
		/* produce an item in next_produced */
		...
		wait(empty);  // 비어있는 buffer의 크기를 count하는 semaphore. 0에서 wait(empty) 호출되면 block.
									// 처음에 n으로 초기화 되어있음
		wait(mutex);
		...
		/* add next produced to the buffer */  // buffer 수정 critical section, 하나의 프로세스만 접근
		...
		signal(mutex);
		signal(full);
} while (true);
```

```c
// consumer : buffer를 비우는 프로세스

do {
		wait(full); /* producer가 먼저 채워 넣지 않으면 block(buffer에 아무것도 없으니!) */ 
								/* buffer에 얼마나 차있는지 count */
		wait(mutex);
		...
		/* remove an item from buffer to next_consumed */
		...
		signal(mutex);
		signal(empty);
		...
		/* consume the item in next consumed */
		...
} while (true);
```

### The Readers-Writers Problem

- reader는 값 update를 하기 때문에 동시에 실행되어도 문제 없음
- writer는 read와 write를 둘 다 하기 때문에 동시에 실행되면 문제 발생 가능
- 그러므로, 오직 하나의 writer가 공유 데이터에 동시에 접근할 수 있다.
- 모든 reader를 즉각 중단 시키고 writer가 critical section에 들어가서 값 변경 (writer에 우선순위를 주는 해결법)  → second variation
- reader가 critical section에서 데이터를 모두 읽을 때 까지 기다린 후 write가 critical section에 들어가 값 변경(reader에 우선순위를 주는 해결법) → first variation, 바로 뒤에 나올 예시

- Shared Data
    - Data set
    - Semaphore rw_mutex 1로 초기화
    - Semaphore mutex 1로 초기화
    - Integer read_count 0으로 초기화

```c
// The structure of a writer process

do {
		wait(rw_mutex); // 먼저 점유한 것이 rw_mutex를 잡음. 다른 프로세스가 이걸 먼저 쥔 프로세스를 기다려야 함
		...
		/* writing is performed */
		...
		signal(rw_mutex);
} while (true);
```

```c
// The structure of a reader process

do {
		wait(mutex); 
		read_count++;  // critical section start
		if (read_count == 1)  // 첫 reader의 경우 rw_mutex에 lock을 건다. (다른 writer가 못들어 오도록)
				wait(rw_mutex);  //critical section end
		signal(mutex);  
		...
		/* reading is performed */
		...
		wait(mutex);
		read_count--;  // critical section start
		if (read_count == 0) // 마지막 reader의 경우 rw_mutex의 lock을 풀어줌
			signal(rw_mutex);  // critical section end
		signal(mutex);
} while (true);
```

- reader가 다 읽고 나가면 writer가 들어오는 first variation

- 하나의 writer가 준비되면 즉각 read를 중단하고 write를 수행하는 second variation (wrlter 우선)
- 두 가지 경우 모두 reader, writer가 starvation이 생길 수 있다.

### 참고 : second variation

```c
int read_count, write_count;
semaphore r_mutex, w_mutex, rw_mutex, read_try;
// r_mutex, w_mutex는 각각 read_count, write_count의 mutual exclusive한 갱신을 위한 세마포어
// w_mutex는 접근 자체를 위한다고 할 수 있다.
// rw_mutex는 critical section data의 mutual exclusive한 접근을 위한 세마포어
// read_try는 reader가 writer 실행 중 들어오지 못하도록 하는 세마포어

// reader process

do {
		wait(read_try); // read 시도, writer가 실행되고 있는지 확인. writer 실행 중 여기서 멈추게 된다.
		wait(r_mutex);
		read_count++;
		if (read_count == 1)
				wait(rw_mutex); // 첫 reader의 경우 다른 writer가 들어오지 못하도록 lock
		signal(r_mutex); 
		signal(read_try);  // read 시도를 성공했으므로 lock을 풀어줌
		...
		/* reading is performed */
		...
		wait(r_mutex);
		read_count--;
		if (read_count ==  0)
				signal(rw_mutex); // 마지막 reader의 경우 lock을 풀어줌
		signal(r_mutex);
} while (true);
```

```c
// writer process

do {
		wait(w_mutex);
		write_count++;
		if (write_count == 1)
				wait(read_try);  // 첫 writer의 경우 read_try에 lock을 걸어 그 다음부터 reader가 접근하지 못하도록 한다.
		signal(w_mutex);

		wait(rw_mutex); // 다른 writer가 들어오지 못하도록 lock을 검
		...
		/* writing is performed */
		...
		signal(rw_mutex); // lock 해제

		wait(w_mutex);
		write_count--;
		if (write_count == 0)
				signal(read_try); // 마지막 writer의 경우 read_try의 lock을 풀어줌으로써 다음 reader의 접근을 허락한다.
		signal(w_mutex);
} while (true);
```

### The Dining-Philosophers Problem

- chopstick이 critical section → 양 철학자가 하나의 chopstick을 공유
- 철학자는 thinking과 eating 두가지를 한다.
- 한 철학자는 양 쪽 2개의 chopstick을 쥐어야 식사가 가능
- chopstick[5]를 1로 초기화

```c
do {
		wait(chopstick[i]);
		wait(chopstick[(i + 1) % 5]);
		// eat

		signal(chopstick[i]);
		signal(chopstick[(i + 1) % 5]);
		// think
} while (true)
```

- 모두가 동시에 같은 방향의 chopstick을 잡으면? → deadlock 모두가 다른쪽 chopstick을 가지지 못함
- 운이 좋지 않으면 계속 식사를 못할 수도 있다. → starvation 발생 가능

## 6.8 Monitors

- 프로세스 동기화를 위한 high-level abstraction (abstract data type)
- internal variable은 오직 프로시저 안에서만 접근 가능
- 오직 하나의 프로세스만 monitor안에서 active 될 수 있다.
    - entry queue에서 wait

```c
monitor monitor-name
{
		// shared variable declarations
		procedure P1 (...){
		....
		}
	
		procedure Pn (...){
		......
		}

		Initialization code (...){
		....
		}
}
```

- example은 수업에서 생략하고 유용성을 중심으로

condition variables

- condition x, y;  (shared data)
- two operation on a condition variable
    - x.wait() - signal이 올때까지 기다림
    - x.signal() - 기다리고 있던 하나를 깨움
- 만약 프로세스 P가 x.signal()을 실행하고 Q가 x.wait() 상태라면?
    - signal and wait - Q가 모니터 안으로 바로 들어감 끝나면 그다음 P가 진입
    - signal and continue - 일단 P가 마무리 그다음 Q가 들어옴

### Solution to Dining Philosophers

```c
monitor DiningPhilosophers
{
		enum {THINKING, HUNGRY, EATING} state[5];
		condition self[5]; 

		void pickup(int i) {  // 철학자 i 가 젓가락을 쥔다
				state[i] = HUNGRY;
				test(i); 
				if (state[i] != EATING) // 양 철학자 중 누군가가 식사중이라 내가 식사 불가능 
						self[i].wait(); // 근데 이거 누가 깨워주지? putdown에서 깨워준다.
		}

		void putdown(int i) {
				state[i] = THINKING;
				test((i + 4) % 5);
				test((i + 1) % 5);
				/*
				양 철학자를 test 시켜 식사 가능 상태면 signal을 준다.
				*/
		}

		void test(int i) {
				if ((state[(i + 4) % 5] != EATING && (state[i] == HUNGRY) && 
						(state[(i + 1) % 5] != EATING)) { // 양 철학자가 EATING이 아니고 내가 HUNGRY 상태면?
						state[i] = EATING; // 나를 EATING 상태로 바꿈
						self[i].signal(); // self 배열은 내가 식사할 수 있는 상태가 되었는지 판단
				} 
		}

		initialization_code() {
				for (int i = 0; i < 5; i++)
						state[i] = THINKING;
		}
}

DiningPhilosophers.pickup(i);
/* ```
   eat
   ``` */
DiningPhilosophers.putdown(i);
```

### Monitor Implementation Using Semaphores

```c
semaphore mutex; // 초기값 1
semaphore next; // 초기값 0 (counting semaphore)
int next_count = 0;

wait(mutex);
```
body of F;
```
if (next_count > 0)
		signal(next); // Function을 나갈 때, 기다리던 프로세스에 신호를 준다.
else
		signal(mutex);
```

```c
semaphore x_sem; // 0으로 초기화 (x라는 condition variable의 counting 목적)
int x_count = 0;

// x.wait
x_count++;
if (next_count > 0)
		signal(next); // 밖에서 기다리는 프로세스에서 신호를 줌
else
		signal(mutex); // 없으면, 모니터를 빠져나옴, wait(mutex) 때문에 진입하지 못하던 것을 할 수 있게 해줌
wait(x_sem);
x_count--;

// x.signal
if (x_count > 0) { // 누군가 기다리고 있다.-> signal을 보내고 monitor나가서 기다리고 있는다.
 		next_count++;
		signal(x_sem);
		wait(next);
		next_count--;
}
```

- condition x가 x.signal()을 받았다면?
    - 먼저 와서 기다리는 프로세스 먼저 깨우기 → FCFS
    - conditional-wait

```c
// A monitor to allocate a single resource
monitor ResourceAllocator
{
		boolean busy;
		condition x;

		void acquire(int time) {
			if (busy)
				x.wait(time);
			busy = true;
		}

		void release() {
			busy = false;
			x.signal();
		}

		initialization_code() {
			busy = false;
		}
}
```