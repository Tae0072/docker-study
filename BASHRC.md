# .bashrc 파일 정리

> **파일 위치:** `~/.bashrc`  
> **실행 시점:** 사용자가 대화형(interactive) bash 쉘을 열 때마다 자동 실행  
> **대상 환경:** Debian/Ubuntu 계열 Linux (Docker 컨테이너 포함)

---

## 1. 대화형 세션 확인

```bash
[ -z "$PS1" ] && return
```

- `$PS1`은 프롬프트 문자열 변수로, **대화형 쉘에서만 설정됨**
- 스크립트나 비대화형 실행 시 `$PS1`이 비어 있으므로 `.bashrc` 이후 내용을 **즉시 종료**
- 이를 통해 불필요한 설정이 자동화 스크립트에 영향을 주지 않도록 방지

---

## 2. 명령어 히스토리(History) 관리

```bash
HISTCONTROL=ignoredups:ignorespace
```
- `ignoredups`: **중복 명령어**를 히스토리에 저장하지 않음
- `ignorespace`: **공백으로 시작하는 명령어**는 히스토리에 저장하지 않음 (비밀번호 등 민감한 명령 숨기기에 활용)

```bash
shopt -s histappend
```
- 쉘 종료 시 히스토리 파일(`~/.bash_history`)을 **덮어쓰지 않고 뒤에 추가(append)**
- 여러 터미널을 동시에 사용할 때 히스토리가 유실되지 않도록 보호

```bash
HISTSIZE=1000
```
- **메모리(현재 세션)**에 보관할 최대 히스토리 줄 수: `1000개`

```bash
HISTFILESIZE=2000
```
- **디스크(`~/.bash_history` 파일)**에 저장할 최대 히스토리 줄 수: `2000개`

---

## 3. 터미널 환경 설정

```bash
shopt -s checkwinsize
```
- 명령어 실행 후마다 **터미널 창 크기를 자동 감지**하여 `$LINES`, `$COLUMNS` 변수를 갱신
- 터미널 창 크기를 조절했을 때 출력이 깨지는 현상 방지

```bash
[ -x /usr/bin/lesspipe ] && eval "$(SHELL=/bin/sh lesspipe)"
```
- `lesspipe`가 설치되어 있을 경우 활성화
- `less` 명령어로 **바이너리, 압축파일, 이미지 등 비텍스트 파일**도 읽기 편하게 변환

```bash
if [ -z "$debian_chroot" ] && [ -r /etc/debian_chroot ]; then
    debian_chroot=$(cat /etc/debian_chroot)
fi
```
- 현재 쉘이 **chroot(가상 루트) 환경** 안에 있는지 확인
- chroot 환경이면 프롬프트에 환경 이름을 표시하여 실수 방지 (예: `(chroot)user@host:~$`)

---

## 4. 프롬프트(Prompt, PS1) 디자인 설정

```bash
case "$TERM" in
    xterm-color) color_prompt=yes;;
esac
```
- 터미널 타입(`$TERM`)이 `xterm-color`인 경우 **컬러 프롬프트 활성화**

```bash
if [ -n "$force_color_prompt" ]; then
    if [ -x /usr/bin/tput ] && tput setaf 1 >&/dev/null; then
        color_prompt=yes
    else
        color_prompt=
    fi
fi
```
- `force_color_prompt=yes`로 강제 설정된 경우, `tput`으로 **실제 색상 지원 여부를 검증**
- 색상 미지원 터미널에서 깨진 문자 출력 방지

```bash
# 컬러 프롬프트 (초록색 user@host, 파란색 경로)
PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '

# 일반 프롬프트 (색상 없음)
PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
```

| 구성 요소 | 의미 |
|---|---|
| `\u` | 현재 사용자 이름 |
| `\h` | 호스트 이름 (컨테이너 ID) |
| `\w` | 현재 작업 디렉토리 (전체 경로) |
| `\$` | 일반 사용자면 `$`, root면 `#` |
| `\[\033[01;32m\]` | ANSI 색상 코드 (굵은 초록) |
| `\[\033[01;34m\]` | ANSI 색상 코드 (굵은 파랑) |
| `\[\033[00m\]` | 색상 초기화 |

```bash
case "$TERM" in
xterm*|rxvt*)
    PS1="\[\e]0;${debian_chroot:+($debian_chroot)}\u@\h: \w\a\]$PS1"
    ;;
esac
```
- `xterm`, `rxvt` 계열 터미널에서 **창 제목(Title Bar)**에 `user@host: 경로` 표시

---

## 5. 색상(Color) 별칭 설정

```bash
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    alias ls='ls --color=auto'
    alias grep='grep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias egrep='egrep --color=auto'
fi
```
- `dircolors`가 있을 경우 **파일/디렉토리 색상 규칙** 로드
- `~/.dircolors`가 있으면 사용자 정의 색상, 없으면 시스템 기본값 사용
- `ls`, `grep` 계열 명령어에 **자동 색상 출력** 적용

---

## 6. ls 단축 명령어(Alias)

```bash
alias ll='ls -alF'   # 모든 파일(숨김 포함) 상세 정보 + 파일 유형 표시 (/ @ * 등)
alias la='ls -A'     # 숨김 파일 포함 표시 (. 과 .. 제외)
alias l='ls -CF'     # 열 단위 간단히 표시 + 파일 유형 표시
```

| alias | 명령어 | 설명 |
|---|---|---|
| `ll` | `ls -alF` | 상세 목록, 숨김 파일 포함, 유형 문자 표시 |
| `la` | `ls -A` | 숨김 파일 포함 (`.`과 `..`만 제외) |
| `l` | `ls -CF` | 열 단위 간략 표시, 유형 문자 표시 |

---

## 7. 사용자 정의 별칭 파일 로드

```bash
if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi
```
- `~/.bash_aliases` 파일이 존재하면 **자동으로 불러옴**
- `.bashrc`를 직접 수정하지 않고 별칭을 별도 파일로 관리할 수 있어 유지보수에 유리

---

## 팁

| 작업 | 방법 |
|---|---|
| 변경사항 즉시 반영 | `source ~/.bashrc` 또는 `. ~/.bashrc` |
| 나만의 단축 명령어 추가 | `~/.bash_aliases`에 `alias 단축어='명령어'` 추가 |
| 컬러 프롬프트 강제 활성화 | `.bashrc`에서 `#force_color_prompt=yes` 주석 해제 |
| 히스토리 즉시 저장 | `history -a` 명령어 실행 |
