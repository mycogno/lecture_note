# Chap 1. Introduction

OS란? 사용자와 컴퓨터 하드웨어 사이에서 작용하는 프로그램

- 컴퓨터 하드웨어의 효율적인 사용
- 사용자 프로그램을 실행, 사용자가 문제해결을 쉽게
- 컴퓨터시스템을 편리하게. 쉽게

## 컴퓨터 시스템 구조

- hardware - CPU, memory, I/O devices - 컴퓨팅 자원 제공
- operating system
- application programs
- users

OS is a resource allocator(자원 할당자) : efficient 하고 fair한 자원 사용

OS is a control program : error를 막고 부적절한 컴퓨터 사용을 막아준다.

항상 실행되는 프로그램은 kernel 그외의 모든 것은 system program

interrupt 발생 → 어떤 종류의 interrupt? → interrupt vector 생성 (interrupt에 따라 적절한 수행을 호출. service routine을 저장해 놓은 table)

software가 발생시키는 interrupt : trap ,exception (error나 user request에 의해 발생)