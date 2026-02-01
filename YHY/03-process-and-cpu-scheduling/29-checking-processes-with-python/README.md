# 🧠 29강 파이썬 코드로 프로세스 확인하기

[▶ 강의 바로가기](https://www.youtube.com/watch?v=XexLQRaKoTA)

파이썬에서 **프로세스(process)** 를 직접 만들어 보면서,  
- 내 프로세스의 PID는 무엇인지
- 자식 프로세스가 생기면 PID/PPID가 어떻게 달라지는지  
- 여러 개의 프로세스를 동시에 띄우면 어떻게 실행되는지  
를 “눈으로 확인”하는 실습 정리다.

---

## 📚 목차

- [🧠 29강 파이썬 코드로 프로세스 확인하기](#-29강-파이썬-코드로-프로세스-확인하기)
  - [📚 목차](#-목차)
  - [준비 지식: PID / PPID란?](#준비-지식-pid--ppid란)
  - [os.getpid()로 내 프로세스 PID 확인](#osgetpid로-내-프로세스-pid-확인)
    - [✅ 기대되는 흐름](#-기대되는-흐름)
  - [Process 1개 생성: 부모/자식 PID 관계 확인](#process-1개-생성-부모자식-pid-관계-확인)
    - [✅ 코드 설명](#-코드-설명)
    - [✅ 출력에서 확인할 것](#-출력에서-확인할-것)
  - [Process 3개 생성: 여러 자식 프로세스 동시에 실행](#process-3개-생성-여러-자식-프로세스-동시에-실행)
    - [✅ 핵심 포인트](#-핵심-포인트)
    - [✅ 관찰 포인트](#-관찰-포인트)
  - [같은 함수(foo)를 3번 실행](#같은-함수foo를-3번-실행)
    - [✅ 무엇을 확인하는 코드인가?](#-무엇을-확인하는-코드인가)
  - [서로 다른 함수(foo/bar/baz)를 각각 실행](#서로-다른-함수foobarbaz를-각각-실행)
    - [✅ 쉽게 이해하기](#-쉽게-이해하기)
    - [✅ 실무적으로 연결하면](#-실무적으로-연결하면)
  - [자주 하는 실수 / 체크포인트](#자주-하는-실수--체크포인트)
    - [`if __name__ == '__main__':` 생략](#if-__name__--__main__-생략)
    - [`.start()`의 반환값 오해](#start의-반환값-오해)
    - [출력 순서가 매번 달라짐](#출력-순서가-매번-달라짐)

---

## 준비 지식: PID / PPID란?

- **PID (Process ID)**: 운영체제가 프로세스에 부여하는 고유 번호  
- **PPID (Parent Process ID)**: 해당 프로세스를 만든 **부모 프로세스**의 PID  
- 파이썬 코드에서:
  - `os.getpid()` → **현재 실행 중인 프로세스의 PID**
  - `os.getppid()` → **부모 프로세스의 PID**

---

## os.getpid()로 내 프로세스 PID 확인

가장 먼저 “지금 이 파이썬 프로그램이 어떤 프로세스로 실행되고 있는지” 확인한다.

```python
import os

print('hello, os!')
print('my pid is ', os.getpid())
```

### ✅ 기대되는 흐름
- 터미널에서 실행하면 `hello, os!` 출력
- 그 다음 줄에 **현재 파이썬 프로세스의 PID**가 찍힘  
  (예: `my pid is  12345`)

> 같은 코드를 여러 번 실행하면 PID는 매번 달라질 수 있다. (실행할 때마다 새 프로세스가 생기기 때문)

---

## Process 1개 생성: 부모/자식 PID 관계 확인

`multiprocessing.Process`를 사용해 **자식 프로세스 1개**를 만든다.

```python
from multiprocessing import Process
import os

def foo():
    print('foo: child process: ', os.getpid())
    print('foo: parent prcess: ', os.getppid())

if __name__ == '__main__':
   print('parent proces: ', os.getpid())
   child = Process(target=foo).start()
```

### ✅ 코드 설명
- `if __name__ == '__main__':`
  - 멀티프로세싱에서 **필수 안전장치**에 가깝다.
  - 이 블록이 없으면 운영체제/환경에 따라 자식 프로세스가 코드를 “재실행”하면서 문제가 생길 수 있다.
- `print('parent proces: ', os.getpid())`
  - **부모 프로세스**(메인 파이썬 실행)의 PID 출력
- `Process(target=foo).start()`
  - `foo()` 함수를 실행하는 **새 프로세스(자식)** 를 생성하고 시작

### ✅ 출력에서 확인할 것
- 부모 출력의 PID ≠ 자식 출력의 PID (서로 다른 프로세스니까)
- 자식이 출력한 `os.getppid()` 값 ≈ 부모 PID (부모-자식 관계 확인)

> 출력 순서는 실행 타이밍에 따라 바뀔 수 있다. (프로세스 스케줄링 때문에)

---

## Process 3개 생성: 여러 자식 프로세스 동시에 실행

이번엔 자식 프로세스를 3개 띄운다.

```python
from multiprocessing import Process
import os

def foo():
    print('foo: child process: ', os.getpid())
    print('foo: parent prcess: ', os.getppid())

if __name__ == '__main__':
   print('parent proces: ', os.getpid())
   child1 = Process(target=foo).start()
   child2 = Process(target=foo).start()
   child3 = Process(target=foo).start()
```

### ✅ 핵심 포인트
- 자식 프로세스가 **3개 생성**됨
- 각각의 자식 PID는 **서로 다름**
- 하지만 `PPID`는 **모두 동일(=부모 PID)** 인 경우가 일반적

### ✅ 관찰 포인트
- 출력이 섞여 나올 수 있음  
  예를 들어 child2가 먼저 출력되고, child1이 나중에 출력되는 등 **순서가 매번 달라질 수 있음**

---

## 같은 함수(foo)를 3번 실행

이번엔 PID 출력 대신 “실행되었다”만 찍어본다.

```python
from multiprocessing import Process
import os

def foo():
    print('executed!!')

if __name__ == '__main__':
   child1 = Process(target=foo).start()
   child2 = Process(target=foo).start()
   child3 = Process(target=foo).start()
```

### ✅ 무엇을 확인하는 코드인가?
- 프로세스를 여러 개 만들면, 같은 함수라도 **각각 독립적으로 실행**된다는 걸 확인
- 결과적으로 `executed!!`가 **3번 출력**되는 것을 기대

> 출력 줄 순서는 여전히 랜덤일 수 있지만, 총 출력 횟수는 3번이 된다.

---

## 서로 다른 함수(foo/bar/baz)를 각각 실행

각기 다른 작업을 서로 다른 프로세스로 분리하는 예시다.

```python
from multiprocessing import Process
import os

def foo():
    print('foo executed!!')

def bar():
    print('bar executed!!')

def baz():
    print('baz executed!!')

if __name__ == '__main__':
   child1 = Process(target=foo).start()
   child2 = Process(target=bar).start()
   child3 = Process(target=baz).start()
```

### ✅ 쉽게 이해하기
- `foo`, `bar`, `baz`는 각각 “별도의 작업”이라고 생각하면 된다.
- `Process(target=...)`로 **작업 단위(함수)** 를 분리하고,
- `.start()`로 **동시에 실행**시킨다.

### ✅ 실무적으로 연결하면
- CPU를 많이 쓰는 작업(예: 영상 인코딩, 대규모 계산)을 나눠서 병렬 실행할 때 이런 구조를 쓴다.
- “작업을 함수로 쪼개고, 프로세스로 병렬 처리한다”의 가장 기본 형태.

---

## 자주 하는 실수 / 체크포인트

### `if __name__ == '__main__':` 생략
- 특히 Windows / 일부 IDE 환경에서 **무한 생성** 또는 이상 동작의 원인이 될 수 있음  
- 멀티프로세싱 예제는 습관처럼 넣는 게 안전

### `.start()`의 반환값 오해
- `Process(target=foo).start()` 는 **프로세스 객체를 반환하지 않는다** (`None` 반환)
- 그래서 아래처럼 쓰면 `child1`은 `None`이 된다.
  ```python
  child1 = Process(target=foo).start()  # child1 == None
  ```
- “프로세스 객체를 들고 있다가 join 하고 싶다”면 보통 이렇게 쓴다:
  ```python
  p = Process(target=foo)
  p.start()
  p.join()
  ```
  (이번 강의는 “프로세스 확인”이 목적이라 join은 생략해도 OK)

### 출력 순서가 매번 달라짐
- 멀티프로세스는 OS 스케줄링에 따라 실행 순서가 달라질 수 있음
- “왜 순서가 뒤죽박죽이지?”는 정상이다.
