# Feature Specification: SpeccedOS: 커널 기반 CLI 운영체제 스펙 문서 (서술형 완전판)

**Feature Branch**: `001-speccedos-cli-speccedos`  
**Created**: Friday, October 10, 2025  
**Status**: Draft  
**Input**: User description: "SpeccedOS: 커널 기반 CLI 운영체제 스펙 문서 (서술형 완전판) SpeccedOS는 리눅스와 유사한 완전한 커널 기반 CLI 운영체제이다. 이 프로젝트의 핵심 목표는 단순한 사용자모드 환경이 아닌, 하드웨어 초기화부터 커널 부팅, 프로세스 스케줄링, 파일시스템 마운트, 셸 실행까지의 모든 단계를 스스로 수행하는 완전한 운영체제를 직접 만드는 것이다. 이 시스템은 실험적이지만 실질적으로 부팅 가능한 OS를 구현하는 것을 목표로 하며, QEMU 또는 실제 머신에서 부팅 가능한 ISO 이미지를 생성할 수 있어야 한다. 즉, SpeccedOS는 “리눅스를 단순히 흉내 내는 OS”가 아니라, 진짜로 커널을 직접 작성하고, 하드웨어와 인터럽트를 다루며, 그 위에서 CLI를 구동하는 자립형 운영체제이다. 1. 프로젝트 개요 SpeccedOS는 부트로더, 커널, 시스템 콜 계층, 그리고 유저랜드로 구성된다. 각 컴포넌트는 명확히 분리되며, 스펙에 따라 점진적으로 구현된다. 전체 시스템의 부팅 순서는 다음과 같다. [BIOS/UEFI] → [Bootloader] → [Kernel Init] → [Memory Setup] → [Scheduler + Syscalls] → [Init Process] → [Shell] 부트로더는 부팅 시 커널 이미지를 메모리에 로드하고, 제어를 커널 엔트리포인트로 넘긴다. 커널은 기본적인 하드웨어 초기화(메모리, 타이머, 인터럽트, 시리얼 포트 등)를 수행한 후, 첫 번째 프로세스인 init을 실행한다. init은 /bin/specced-shell을 실행시켜 CLI 환경을 구성한다. 사용자는 커널이 직접 제공하는 쉘을 통해 시스템 명령을 수행한다. 2. 핵심 철학 SpeccedOS의 철학은 네 가지로 요약된다. 직접 제어(Direct Control) — 하드웨어, 메모리, 프로세스, 파일을 완전히 직접 제어할 수 있는 환경 제공. 단순한 구조(Simple Kernel) — 불필요한 계층을 최소화하고, 커널 내부를 완전히 투명하게 설계. 학습과 실험 중심(Learn-by-Building) — 운영체제를 만들면서 OS 원리를 체득. 확장 가능성(Modular Extensibility) — 모듈 단위로 커널 기능을 교체하거나 확장 가능. 이 철학에 따라, SpeccedOS는 단순한 CLI 운영체제가 아닌 **“학습 가능한 리눅스 대체 핵심 아키텍처”**로 작동한다. 3. 기술 목표 SpeccedOS의 기술적 목표는 다음과 같다. 부팅 가능한 커널 이미지 생성 x86_64 아키텍처에서 BIOS/UEFI 부팅 지원 GRUB 또는 자체 부트로더를 사용 가능 커널 초기화 루틴 메모리 매핑, GDT/IDT 초기화, 타이머 및 인터럽트 처리 시리얼 포트 또는 VGA를 통한 콘솔 출력 지원 프로세스 및 스케줄링 시스템 기본적인 프로세스 생성(fork), 컨텍스트 스위칭, 시그널 전달 Round-Robin 또는 Priority 기반 스케줄러 파일시스템 단순한 VFS 레이어와 FAT16 또는 자체 파일시스템 구현 디스크 드라이버와 블록 캐시 시스템 제공 유저랜드 (Shell 및 유틸리티) /bin/specced-shell을 통해 CLI 환경 제공 기본 명령어(ls, cat, echo, ps, kill, mount, shutdown) 구현 시스템콜 인터페이스 read/write, fork, exec, open, close, exit 등의 syscall 지원 유저 프로세스는 syscall 인터럽트를 통해 커널과 통신 개발 툴체인 크로스 컴파일러(x86_64-elf-gcc) QEMU 가상 머신 테스트 및 디버깅 스크립트 패키징 및 부팅 ISO 이미지 생성 (make iso) qemu-system-x86_64 -cdrom speccedos.iso로 부팅 가능 4. 시스템 구조 개요 SpeccedOS는 다음 네 계층으로 구성된다: 1) Bootloader GRUB 또는 자체 작성된 Stage1/Stage2 로더 사용 커널 ELF를 로드하고 kernel_main() 호출 실모드에서 보호모드 전환 처리 2) Kernel Core 메모리 관리자, 인터럽트, 스케줄러, VFS, 드라이버 관리자 포함 C로 작성하되, 초기 진입점 및 저수준 루틴은 Assembly로 구현 주요 모듈: mm/, sched/, fs/, drivers/, sys/, libk/ 3) System Call Layer int 0x80 또는 syscall 명령으로 유저모드-커널모드 전환 유저 프로그램은 링3 권한에서 실행되며, 커널은 링0 4) Userland /bin, /usr/bin, /dev, /etc 등의 디렉토리 구성 init 프로세스 실행 후 specced-shell 진입 간단한 텍스트 기반 CLI 환경 제공 5. 개발 언어 및 환경 커널 언어: C (C99) + Assembly 유저랜드: C 빌드 도구: GNU Make, NASM, LD 에뮬레이터: QEMU 테스트: Python 기반 부팅 자동화 테스트 디버깅: GDB + Serial console 개발은 리눅스 호스트에서 진행되며, make run 명령으로 QEMU를 통해 부팅 테스트를 수행한다. 6. 폴더 구조 초안 speccedos/ ├─ boot/ # 부트로더 및 엔트리포인트 코드 ├─ kernel/ │ ├─ arch/ # 아키텍처별 코드 (x86_64) │ ├─ mm/ # 메모리 관리 │ ├─ sched/ # 스케줄러 │ ├─ fs/ # 파일시스템 │ ├─ drivers/ # 드라이버 (timer, keyboard, disk) │ └─ sys/ # 시스템콜 ├─ user/ │ ├─ bin/ # 셸 및 유틸리티 │ └─ init/ # init 프로세스 ├─ docs/ ├─ scripts/ ├─ tests/ └─ .specify/ ├─ spec.md ├─ plan.md ├─ constitution.md └─ tasks/ 7. 핵심 요구사항 요약 SpeccedOS는 부팅 가능한 커널을 생성해야 한다. 시스템은 메모리 관리, 프로세스, 파일시스템, 인터럽트를 갖춘다. 사용자는 부팅 후 직접 CLI 환경에서 명령어를 입력할 수 있다. 시스템은 QEMU에서 완전한 독립 실행이 가능해야 한다. 빌드 과정은 make all로 단일 ISO를 출력해야 한다. 8. 수용 기준 (Acceptance Criteria) make run 명령 실행 시 QEMU 콘솔에 SpeccedOS 로고 및 쉘 프롬프트가 표시되어야 한다. ls, cat, echo, ps 명령이 정상 작동해야 한다. fork, exec, wait 시스템콜이 정상적으로 작동해야 한다. shutdown 명령으로 OS가 정상 종료되어야 한다. .specify/spec.md는 최신 상태로 유지되고, /plan, /tasks 생성이 가능해야 한다. 9. 철학적 목표 SpeccedOS는 “AI와 인간이 함께 만든 최초의 완전 커널 기반 CLI 운영체제”라는 비전을 가진다. 이 프로젝트는 단순한 코드 실험이 아니라, 운영체제의 구조와 철학을 직접 구현하는 창조 행위다. 커널은 코드로서의 예술이며, 셸은 인간과 기계의 대화 창구이다. SpeccedOS의 존재 이유는 단순하다 — “운영체제를 배우는 가장 빠른 길은, 직접 만드는 것”이기 때문이다."

## User Scenarios & Testing

### User Story 1 - Boot SpeccedOS and access the shell (Priority: P1)

The user wants to successfully boot SpeccedOS in QEMU and be presented with a functional command-line interface (shell prompt).

**Why this priority**: This is the fundamental entry point and core functionality of the entire operating system. Without successful booting and shell access, no other features can be tested or utilized.

**Independent Test**: Can be fully tested by running `make run` and observing the QEMU console for the SpeccedOS logo and shell prompt.

**Acceptance Scenarios**:

1.  **Given** a built SpeccedOS ISO image, **When** `make run` is executed, **Then** the QEMU console displays the SpeccedOS logo and a shell prompt.

---

### User Story 2 - Execute basic shell commands (Priority: P1)

The user wants to be able to execute essential CLI commands like `ls`, `cat`, `echo`, and `ps` within the SpeccedOS shell.

**Why this priority**: These commands are critical for basic interaction, file system navigation, and process monitoring, forming the foundation of any usable CLI environment.

**Independent Test**: Can be fully tested by booting SpeccedOS and typing each command at the shell prompt, verifying the expected output.

**Acceptance Scenarios**:

1.  **Given** SpeccedOS is booted and the shell prompt is active, **When** the user types `ls` and presses Enter, **Then** a list of files in the current directory is displayed.
2.  **Given** SpeccedOS is booted and the shell prompt is active, **When** the user types `echo "Hello, SpeccedOS!"` and presses Enter, **Then** "Hello, SpeccedOS!" is displayed on the console.
3.  **Given** SpeccedOS is booted and the shell prompt is active, **When** the user types `ps` and presses Enter, **Then** a list of running processes is displayed.

---

### User Story 3 - Utilize core system calls for process management (Priority: P2)

The user wants to ensure that fundamental system calls for process management, specifically `fork`, `exec`, and `wait`, are correctly implemented and functional.

**Why this priority**: These system calls are crucial for multi-tasking and running user applications, which are core to an operating system's functionality beyond a simple shell.

**Independent Test**: Can be tested by running a simple userland program that utilizes `fork`, `exec`, and `wait` to create and manage child processes, verifying their behavior.

**Acceptance Scenarios**:

1.  **Given** SpeccedOS is running, **When** a user program calls `fork()`, **Then** a new child process is created with a copy of the parent's address space.
2.  **Given** a child process exists, **When** the child process calls `exec()`, **Then** its memory space is replaced with a new program.
3.  **Given** a parent process has forked a child, **When** the parent calls `wait()`, **Then** it pauses execution until the child process terminates.

---

### User Story 4 - Gracefully shut down the OS (Priority: P3)

The user wants to be able to shut down the SpeccedOS cleanly using a dedicated command.

**Why this priority**: A proper shutdown mechanism is essential for system integrity and preventing data corruption, even in an experimental OS.

**Independent Test**: Can be tested by booting SpeccedOS, executing the shutdown command, and verifying that QEMU terminates or the system halts.

**Acceptance Scenarios**:

1.  **Given** SpeccedOS is running and the shell prompt is active, **When** the user types `shutdown` and presses Enter, **Then** the operating system performs a graceful shutdown and terminates.

---

### Edge Cases

-   What happens when an invalid command is entered in the shell? The shell should display an error message (e.g., "command not found").
-   How does the system handle out-of-memory conditions during process creation or memory allocation? The kernel should gracefully handle the error, potentially killing the offending process or returning an error to the caller.
-   What happens if a critical hardware interrupt (e.g., timer) is not handled correctly? The system should ideally log the error and attempt recovery, or panic if unrecoverable.

## Requirements

### Functional Requirements

-   **FR-001**: SpeccedOS MUST generate a bootable kernel image for x86_64 architecture.
-   **FR-002**: SpeccedOS MUST support BIOS/UEFI booting, using GRUB or a custom bootloader.
-   **FR-003**: The kernel MUST perform essential hardware initialization (memory mapping, GDT/IDT, timer, interrupts, serial port).
-   **FR-004**: SpeccedOS MUST provide console output via serial port or VGA.
-   **FR-005**: SpeccedOS MUST implement basic process creation (`fork`), context switching, and signal delivery.
-   **FR-006**: SpeccedOS MUST include a Round-Robin or Priority-based scheduler.
-   **FR-007**: SpeccedOS MUST implement a simple Virtual File System (VFS) layer.
-   **FR-008**: SpeccedOS MUST support FAT16 or a custom filesystem, including disk drivers and a block cache.
-   **FR-009**: SpeccedOS MUST provide a `/bin/specced-shell` for the CLI environment.
-   **FR-010**: The shell MUST implement basic commands: `ls`, `cat`, `echo`, `ps`, `kill`, `mount`, `shutdown`.
-   **FR-011**: SpeccedOS MUST provide system call interfaces for `read`, `write`, `fork`, `exec`, `open`, `close`, `exit`.
-   **FR-012**: User processes MUST interact with the kernel via syscall interrupts (Ring 3 to Ring 0 transition).
-   **FR-013**: SpeccedOS MUST be buildable with an x86_64-elf-gcc cross-compiler, GNU Make, NASM, and LD.
-   **FR-014**: SpeccedOS MUST be testable and debuggable using QEMU, Python-based boot automation, and GDB with serial console.
-   **FR-015**: The build process (`make iso`) MUST generate a single ISO image bootable with `qemu-system-x86_64 -cdrom speccedos.iso`.
-   **FR-016**: The system MUST ensure `.specify/spec.md` is kept up-to-date and allow generation of `/plan` and `/tasks`.

### Key Entities

-   **Process**: Represents an executing program, with its own memory space, registers, and state. Key attributes include PID, parent PID, state (running, ready, blocked), and memory regions.
-   **File**: Represents data stored on a persistent storage device, accessible via the VFS. Key attributes include name, size, permissions, and content.
-   **Filesystem**: Manages the organization and storage of files on a disk.
-   **System Call**: An interface for user-mode programs to request services from the kernel.
-   **Interrupt**: A signal to the processor emitted by hardware or software indicating an event that needs immediate attention.

## Success Criteria

### Measurable Outcomes

-   **SC-001**: `make run` command successfully boots SpeccedOS in QEMU, displaying the SpeccedOS logo and a shell prompt within 30 seconds.
-   **SC-002**: All specified basic shell commands (`ls`, `cat`, `echo`, `ps`, `kill`, `mount`, `shutdown`) function correctly and produce expected output.
-   **SC-003**: `fork`, `exec`, and `wait` system calls are fully functional, allowing for the creation and management of multiple processes.
-   **SC-004**: The `shutdown` command successfully terminates the operating system cleanly.
-   **SC-005**: The generated `speccedos.iso` image is consistently bootable in QEMU.
-   **SC-006**: The `.specify/spec.md` document accurately reflects the current state of the project and can be used to generate `plan.md` and `tasks.md`.