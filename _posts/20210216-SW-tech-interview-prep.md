---
title: SW 기술면접 준비
date: 2021-02-16 20:38:19
mathjax: true
tags: 
- Programming
- Concurrency
- Parallelism
- Semaphore
- Mutex
- Thread
- Process
categories: 
- [  SW]
excerpt: 컴공 관련 여러가지 기본적인 기술면접 질문/답변
---

# Process vs Thread

- Program: 작업을 수행하기 위해 실행가능한 파일
- Process: 실행되고 있는 Program
  - Code | Heap | Register, counter, stack을 가지고 있음.
- Thread: Process 내부의 실행 단위
  - 하나의 process 안에 여러개의 thread가 있을 수 있고, 모든 thread는 code, data, heap을 공유.
  - Thread마다 register, counter, stack을 따로 가지고 있음.

## 여러개의 작업을 할 때 여러개의 process를 쓰는게 좋은가? 아니면 여러개의 thread를 쓰는게 좋은가?

- Process를 새로 생성하거나 제거하는 것이 Thread를 새로 생성하거나 제거하는 것 보다 오래 걸림.
- Process간의 context switching이 Thread간의 context switching보다 오래 걸림.
  - Context switching: 현재 작업이 사용하는 CPU의 자원을 process control block에 저장하고, 기존의 하던 작업을 로드함. 이로써 작업 변환이 가능.


# Multi-processing vs Multi-threading

- Multi-processing : 하나의 작업을 위해 여러개의 process를 한번에 돌리는 것.
  - 여러개의 CPU가 필요함.
  - Socket 통신과 같은 Process들간의 데이터 통신이 가능.
  - Process 중 하나가 죽어도 나머지는 계속 돌아감. (안정적?)
  - Context switching으로 여러개의 process가 동시에 돌아가는 것 처럼 보이게함.
- Multi-threading : 하나의 작업을 위해 여러개의 thread를 한번에 돌리는 것.
  - 하나의 process에 대해 여러 thread가 동시에 수행.
  - Context switching으로 여러개의 thread가 동시에 돌아가는 것 처럼 보이게함.

# Concurrency vs Parallelism

- Concurrency: 다수의 작업을 동시에 진행하는 것. 하지만 정말로 어떠한 시점에서 여러개의 작업이 동시에 처리되고 있지 않아도 된다.
  - 하나의 작업을 진행하다가 context switching을 해서 다른 작업을 하다가, 다시 돌아와서 또 하고...
- Paralellism: 다수의 작업을 *동시에* 진행하는 것. 어떠한 시점에서 여러개의 작업이 동시에 처리되고 있어야함.
  - 1번 작업과 2번 작업이 각각 CPU 하나씩 할당해서 동시에 처리되는 느낌.

# Semaphore vs Mutex

- Semaphore : 공유된 자원의 데이터를 여러 process가 접근하는 것을 막는 것
  - wait / signal 이 있는데, process가 resource를 점유하고 있는지 아닌지를 알려준다. 
    - Couting semaphore - n/0 시그널. 0이면 resource가 점유중, n이면 n개만큼의 process가 추가로 점유 가능.
    - Binary semaphore - 1/0 시그널. 1이면 resource가 오픈, 0이면 resource가 점유중. 
- Mutex : 공유된 자원의 데이터를 여러 thread가 접근하는 것을 막는 것
  - Process가 resource를 점유하면 mutex object의 lock 권한을 가지게 된다. 

# Overriding vs Overloading

- Overriding: 부모 클래스가 가지고 있는 메소드를 자식 클래스가 재정의해서 사용하는 것.
  - 메소드의 이름/매개변수/반환형이 같은 상속 받은 메소드를 덮어쓰는 것 (부모 클래스의 메소드는 무시하고 자식 클래스가 필요로하는 메소드를 추가하여 사용하는 것)
- Overloading: 같은 이름의 메소드를 여러개 정의하되, 매개 변수의 유형과 개수를 달리하여 다양한 유형의 호출에 응답하는 것.