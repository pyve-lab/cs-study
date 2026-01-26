# 🧠 42강 파일 시스템(완강)

파일 시스템(File System)은 **보조기억장치(SSD/HDD)** 위에 **파일과 디렉터리**를 저장하고,
운영체제가 **“어디에 저장했는지(위치)”** 와 **“어떻게 찾아갈지(탐색)”** 를 관리하는 규칙/구조입니다.

- 파일은 그냥 “연속된 바이트 덩어리”처럼 보이지만,
- 실제 저장 장치에서는 **블록(Block) 단위로 쪼개져** 여러 군데에 저장될 수 있고,
- OS는 파일 시스템의 규칙을 이용해 **파일 이름 → 메타데이터(위치/크기/권한) → 데이터 블록** 순서로 접근합니다.

![overview](<./images/overview.png>)

---

## 📚 목차
- [🧠 42강 파일 시스템(완강)](#-42강-파일-시스템완강)
  - [📚 목차](#-목차)
  - [🔎 개요](#-개요)
    - [파일 시스템이 해결하는 2가지 질문](#파일-시스템이-해결하는-2가지-질문)
    - [대표 파일 시스템](#대표-파일-시스템)
  - [🧩 파티셔닝과 포매팅](#-파티셔닝과-포매팅)
    - [🧱 파티셔닝](#-파티셔닝)
    - [🧰 포매팅](#-포매팅)
  - [🗂 파일 할당 방법](#-파일-할당-방법)
    - [1) 연속 할당 (Contiguous Allocation)](#1-연속-할당-contiguous-allocation)
      - [개념](#개념)
    - [2) 불연속 할당: 연결 할당 (Linked Allocation)](#2-불연속-할당-연결-할당-linked-allocation)
      - [개념](#개념-1)
      - [🔑 디렉터리 엔트리에는 뭘 적어둘까?](#-디렉터리-엔트리에는-뭘-적어둘까)
    - [3) 불연속 할당: 색인 할당 (Indexed Allocation)](#3-불연속-할당-색인-할당-indexed-allocation)
      - [개념](#개념-2)
      - [🔑 디렉터리 엔트리에는 뭘 적어둘까?](#-디렉터리-엔트리에는-뭘-적어둘까-1)
  - [🧪 파일 시스템 살펴보기](#-파일-시스템-살펴보기)
  - [💾 FAT 파일 시스템](#-fat-파일-시스템)
  - [🐧 유닉스 파일 시스템](#-유닉스-파일-시스템)
    - [1) i-node 기본 역할(메타데이터 + 블록 주소)](#1-i-node-기본-역할메타데이터--블록-주소)
    - [2) 블록 관점에서 “파일이 저장되는 모습”](#2-블록-관점에서-파일이-저장되는-모습)
    - [3) 큰 파일은 어떻게 가리킬까? (직접/간접 블록)](#3-큰-파일은-어떻게-가리킬까-직접간접-블록)
    - [4) 디렉터리 엔트리: “파일 이름 → i-node 번호”](#4-디렉터리-엔트리-파일-이름--i-node-번호)
    - [5) 경로로 파일을 읽는 과정 예시: `/home/minchul/a.sh`](#5-경로로-파일을-읽는-과정-예시-homeminchulash)
  - [✅ 한눈에 비교 요약](#-한눈에-비교-요약)
  - [🧾 정리: “파일 하나를 읽을 때” 실제 흐름](#-정리-파일-하나를-읽을-때-실제-흐름)

---

## 🔎 개요

### 파일 시스템이 해결하는 2가지 질문
1) **저장**: 파일/디렉터리를 디스크(보조기억장치)에 *어떤 방식으로 배치할까?*  
2) **접근**: `/home/a.txt` 같은 경로로 요청이 들어왔을 때, *어떻게 빠르게 찾을까?*

즉, 파일 시스템은 “저장 구조(배치/할당)” + “탐색 구조(메타데이터/디렉터리)”의 조합입니다.

### 대표 파일 시스템
- **FAT 파일 시스템**: 연결 할당 기반(포인터 정보를 FAT 테이블로 모아 관리)
- **유닉스 파일 시스템**: 색인 할당 기반(i-node 사용)

---

## 🧩 파티셔닝과 포매팅

새 SSD/HDD는 보통 “그냥 빈 창고”입니다.  
OS가 파일을 넣으려면 먼저 **구역을 나누고(파티셔닝)**, **관리 규칙을 정해야(포매팅)** 합니다.

---

### 🧱 파티셔닝

- 저장 장치의 **논리적인 영역을 구획**하는 작업
- 구획된 각 영역을 **파티션(Partition)** 이라고 합니다.
- 파티션이 나뉘면 “한 디스크”를 여러 용도로 분리할 수 있습니다.
  - 예: OS용(C:), 데이터용(D:), 복구용(Recovery) 등

![windows disk partitioning](<./images/windows-disk-partitioning.png>)

<details>
<summary>🖼️ 파티셔닝 예시(“서랍을 칸칸이 나누기”)</summary>

- 파티셔닝은 “큰 서랍장”에 **칸막이를 쳐서 구역을 만드는 것**과 유사합니다.
- 칸이 나뉘면 각 칸은 독립적으로 관리(포매팅/파일시스템 적용)할 수 있습니다.

![partitioning example 01](<./images/partitioning-example-01.png>)
![partitioning example 02](<./images/partitioning-example-02.png>)

</details>

---

### 🧰 포매팅

- 파티션에 **파일 시스템을 설정**하는 작업
- “어떤 방식으로 파일/디렉터리를 관리할지”를 결정하고, **새 데이터를 쓸 준비**를 합니다.
- 파일 시스템은 **포매팅할 때 결정**됩니다.
- 파티션마다 서로 다른 파일 시스템을 설정할 수도 있습니다.

![windows usb formatting](<./images/windows-usb-formatting.png>)

> 포매팅 화면에서 파일 시스템(FAT32/NTFS 등)을 고르는 순간,  
> 그 파티션은 “그 파일 시스템의 규칙”으로 관리됩니다.

![formatting complete](<./images/formatting-complete.png>)

---

## 🗂 파일 할당 방법

운영체제는 보조기억장치에 데이터를 보통 **블록(Block) 단위**로 읽고 씁니다.

- 디스크의 물리 최소 단위는 섹터(Sector)이지만,
- 운영체제는 효율을 위해 **블록 단위(여러 섹터 묶음)** 로 관리하는 경우가 많습니다.

따라서 파일 하나는 보통 **여러 블록에 나뉘어 저장**됩니다.

![file allocation types](<./images/file-allocation-types.png>)
![file allocation overview](<./images/file-allocation-01-overview.png>)

---

### 1) 연속 할당 (Contiguous Allocation)

#### 개념
- 파일을 **연속된 블록 구간**에 저장합니다.
- 디렉터리 엔트리(메타 정보)에는 보통 다음이 기록됩니다.
  - 파일 이름
  - **첫 번째 블록 주소**
  - **길이(블록 수)**

✅ 장점
- 구현이 단순합니다.
- 파일이 연속된 공간에 있으니 **순차 읽기**가 빠릅니다.
- 파일의 첫 블록과 길이만 알면 “쭉” 읽기 쉬움.

❌ 단점: 외부 단편화(External Fragmentation)
- 시간이 지나며 파일이 생성/삭제되면 디스크가 “군데군데 빈 공간”이 생깁니다.
- 빈 공간이 총합으로는 충분해도,
  **연속된 큰 공간이 없으면 큰 파일을 저장 못하는 상황**이 발생합니다.

![contiguous allocation](<./images/file-allocation-02-contiguous.png>)

<details>
<summary>🖼️ 연속 할당 예시/단점(외부 단편화)</summary>

![contiguous example](<./images/file-allocation-03-contiguous-example.png>)
![fragmentation 01](<./images/file-allocation-04-contiguous-fragmentation-01.png>)
![fragmentation 02](<./images/file-allocation-05-contiguous-fragmentation-02.png>)

</details>

> 💡 핵심 요약  
> 연속 할당은 “빠르지만”, 디스크가 오래 쓰일수록 “연속 공간” 확보가 어려워져 현실적으로 불리합니다.

---

### 2) 불연속 할당: 연결 할당 (Linked Allocation)

#### 개념
- 파일을 구성하는 블록들이 디스크 곳곳에 흩어져 있어도 됩니다.
- 각 블록 내부에 **다음 블록 주소(포인터)** 를 저장해서 연결 리스트처럼 이어줍니다.

✅ 장점
- 연속된 큰 공간이 없어도 저장 가능 → 외부 단편화 문제를 완화합니다.

❌ 단점
- 중간 블록으로 “점프”가 어렵습니다.  
  즉, **임의 접근(Random Access)** 이 불리합니다(처음부터 포인터를 따라가야 함).
- 블록 하나가 손상되면 체인이 끊겨 이후 데이터 접근이 어려울 수 있습니다.

![linked allocation](<./images/file-allocation-06-linked.png>)

#### 🔑 디렉터리 엔트리에는 뭘 적어둘까?
연결 할당은 보통 디렉터리 엔트리에
- 파일 이름
- **첫 번째 블록 주소**
- 길이(블록 수)
를 기록합니다.

![linked directory entry](<./images/file-allocation-07-linked-directory-entry.png>)

> 💡 핵심 요약  
> 연결 할당은 “저장 유연성”을 얻는 대신 “빠른 임의 접근”을 희생합니다.

---

### 3) 불연속 할당: 색인 할당 (Indexed Allocation)

#### 개념
- 파일이 사용하는 모든 블록 주소를 **색인 블록(Index Block)** 하나에 모아둡니다.
- 즉 “주소 목록”을 한 곳에 두고, 데이터 블록은 흩어져 있어도 됩니다.

✅ 장점
- 색인 블록에서 원하는 블록 주소를 찾아갈 수 있어 **임의 접근이 유리**합니다.

❌ 단점
- 색인 블록 자체가 필요하므로 **추가 공간/오버헤드**가 생깁니다.
- 파일이 아주 작아도 색인 구조를 위한 비용이 존재합니다(구현 방식에 따라 다름).

![indexed allocation](<./images/file-allocation-08-indexed.png>)

#### 🔑 디렉터리 엔트리에는 뭘 적어둘까?
색인 할당은 보통 디렉터리 엔트리에
- 파일 이름
- **색인 블록 주소(주소 목록 블록)**
를 기록합니다.

![indexed directory entry](<./images/file-allocation-09-indexed-directory-entry.png>)

> 💡 핵심 요약  
> 색인 할당은 “임의 접근 성능”을 위해 “주소 목록 블록(색인)”을 둡니다.

---

## 🧪 파일 시스템 살펴보기

---

## 💾 FAT 파일 시스템

FAT(File Allocation Table)는 “연결 할당”을 실제 파일 시스템으로 구현하면서,
연결 할당의 단점을 줄이기 위해 **포인터(다음 블록 주소) 정보를 한 곳에 모아둔 방식**입니다.

- 연결 할당에서는 각 블록에 “다음 블록 주소”를 넣어야 했죠.
- FAT에서는 그 정보를 블록이 아닌 **FAT 테이블**이 가지고 있습니다.

✅ 기대 효과
- 다음 블록을 찾기 위해 디스크 블록을 매번 읽기보다,
  FAT를 메모리에 캐시하면 **탐색이 빨라질 수 있습니다.**

![fat fs overview](<./images/fat-fs-01-overview.png>)

<details>
<summary>🖼️ FAT 구조/디렉터리 엔트리/읽는 과정</summary>

![fat fs layout](<./images/fat-fs-02-layout.png>)
![fat table example](<./images/fat-fs-03-fat-table.png>)
![fat directory entry](<./images/fat-fs-04-directory-entry.png>)
![fat read process](<./images/fat-fs-05-read-process.png>)

</details>

---

## 🐧 유닉스 파일 시스템

유닉스 계열 파일 시스템은 대표적으로 **i-node**라는 구조를 중심으로 동작하며,
큰 흐름은 “색인 할당(Indexed Allocation)”입니다.

---

### 1) i-node 기본 역할(메타데이터 + 블록 주소)

i-node에는 두 종류의 정보가 들어갑니다.

1) **메타데이터(속성)**
- 파일 크기, 소유자, 권한, 생성/수정/접근 시간 등

2) **데이터 블록 주소**
- 실제 파일 내용이 들어있는 데이터 블록들의 위치

![inode structure](<./images/unix-fs-01-inode-structure.png>)
![unix disk layout](<./images/unix-fs-02-disk-layout.png>)

---

### 2) 블록 관점에서 “파일이 저장되는 모습”

디스크를 “블록 번호”로 바라보면,
파일은 블록들 중 일부를 차지합니다.

![blocks](<./images/unix-fs-03-blocks.png>)
![block allocation example](<./images/unix-fs-04-block-allocation-example.png>)

---

### 3) 큰 파일은 어떻게 가리킬까? (직접/간접 블록)

i-node에 저장할 수 있는 주소 칸이 제한되어 있을 때,
아주 큰 파일은 “주소 칸”이 모자랄 수 있습니다.

그래서 유닉스 파일 시스템은 보통 다음 방식으로 확장합니다.

- **직접 블록(Direct)**: i-node가 데이터 블록을 바로 가리킴
- **단일 간접(Single Indirect)**: i-node가 “주소 목록 블록”을 가리킴
- **이중/삼중 간접(Double/Triple Indirect)**: 주소의 주소를 따라 확장

![large file problem](<./images/unix-fs-05-large-file-problem.png>)

<details>
<summary>🖼️ 직접/간접 블록 그림</summary>

![direct blocks](<./images/unix-fs-06-direct-blocks.png>)
![single indirect](<./images/unix-fs-07-single-indirect.png>)

</details>

> 💡 핵심 요약  
> i-node는 “작은 파일은 빠르게(직접)” 처리하고,  
> “큰 파일은 단계적으로(간접)” 확장할 수 있게 설계됩니다.

---

### 4) 디렉터리 엔트리: “파일 이름 → i-node 번호”

유닉스 계열에서 디렉터리는 보통 다음 역할을 합니다.

- **파일 이름을 보고**
- 그 파일의 **i-node 번호**를 찾아줍니다.

즉 “이 파일의 메타데이터/블록 주소는 i-node에 있다”는 구조입니다.

![unix directory entry](<./images/unix-fs-08-directory-entry.png>)

---

### 5) 경로로 파일을 읽는 과정 예시: `/home/minchul/a.sh`

경로 탐색은 “디렉터리 엔트리 따라가기”의 반복입니다.

1) 루트(`/`) 디렉터리에서 `home`을 찾고 → 해당 i-node로 이동  
2) `home` 디렉터리에서 `minchul`을 찾고 → 해당 i-node로 이동  
3) `minchul` 디렉터리에서 `a.sh`를 찾고 → 해당 i-node로 이동  
4) i-node 안의 데이터 블록 주소들을 따라 파일 내용을 읽음

![path lookup](<./images/unix-fs-09-path-lookup.png>)

---

## ✅ 한눈에 비교 요약

| 구분 | 기반 할당 방식 | 핵심 아이디어 | 임의 접근(랜덤 액세스) | 대표 키워드 |
|---|---|---|---|---|
| 연속 할당 | 연속 | 연속 블록 구간에 저장 | ✅ 매우 유리 | 외부 단편화 |
| 연결 할당 | 불연속(연결) | 블록이 다음 블록을 가리킴 | ❌ 불리 | Linked List |
| 색인 할당 | 불연속(색인) | 색인 블록이 주소 목록 보관 | ✅ 유리 | Index Block |
| FAT | 연결 기반 개선 | 포인터를 FAT 테이블로 모아 관리 | △ 캐시 시 개선 | FAT |
| 유닉스 | 색인 기반 | i-node가 메타+주소(확장) 보관 | ✅ 유리 | i-node |

---

## 🧾 정리: “파일 하나를 읽을 때” 실제 흐름

아래 흐름이 이해되면 파일 시스템의 큰 그림이 잡힙니다.

1) 사용자가 경로로 파일 요청  
   - 예: `/home/minchul/a.sh`

2) OS는 디렉터리 엔트리를 따라가며 “식별자”를 찾음  
   - FAT: “시작 블록” 등으로 이어지는 힌트  
   - 유닉스: “파일 이름 → i-node 번호”

3) 파일의 실제 데이터 블록 위치를 확보  
   - FAT: FAT 테이블에서 다음 블록 체인을 따라감  
   - 유닉스: i-node가 들고 있는 주소(직접/간접)를 따라감

4) 확보한 블록들을 읽어 사용자에게 파일 내용을 전달

> 결론:  
> **디렉터리 엔트리(이름→힌트) + 메타데이터 구조(FAT/i-node) + 블록 주소 관리 방식**이  
> 파일 시스템의 핵심입니다.
