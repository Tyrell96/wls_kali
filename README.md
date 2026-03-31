# Kali Linux WSL2 Setup (HTB 환경) — 최종본

> **환경:** Windows 11 + WSL2 + Windows Terminal
> **날짜:** 2026-03-17
> **목적:** HackTheBox 풀이용 Kali 세팅

---

## 0. 사전 준비

### Nerd Font 설치 (Windows)
1. https://github.com/ryanoasis/nerd-fonts/releases/latest
2. `JetBrainsMono.zip` 다운로드
3. 압축 해제 → 전체 선택 → 우클릭 → **모든 사용자용으로 설치**

### Kali Linux 설치
```powershell
# Microsoft Store에서 Kali Linux 설치 후
wsl -l -v  # 설치 확인
```

---

## 1. Windows Terminal — Nord 테마 + 단축키 설정

`Ctrl+,` → JSON 파일 열기

### `"schemes"` 배열에 추가
```json
{
    "name": "Nord",
    "background": "#2E3440",
    "foreground": "#D8DEE9",
    "black": "#3B4252",
    "blue": "#81A1C1",
    "brightBlack": "#4C566A",
    "brightBlue": "#81A1C1",
    "brightCyan": "#8FBCBB",
    "brightGreen": "#A3BE8C",
    "brightPurple": "#B48EAD",
    "brightRed": "#BF616A",
    "brightWhite": "#ECEFF4",
    "brightYellow": "#EBCB8B",
    "cyan": "#88C0D0",
    "green": "#A3BE8C",
    "purple": "#B48EAD",
    "red": "#BF616A",
    "white": "#E5E9F0",
    "yellow": "#EBCB8B"
}
```

### `"profiles"` → `"list"` 배열에 Kali 프로필 추가
> `"source": "Microsoft.WSL"` 방식이 안 될 경우 `"commandline"` 방식 사용

```json
{
    "colorScheme": "Nord",
    "font": {
        "face": "JetBrainsMono Nerd Font",
        "size": 11
    },
    "guid": "{46ca431a-3a87-5fb3-83cd-11ececc031d2}",
    "hidden": false,
    "name": "Kali Linux",
    "commandline": "wsl -d kali-linux",
    "opacity": 95,
    "useAcrylic": true
}
```

### `"actions"` 배열에 추가 (Alt+방향키 Windows Terminal 차단 해제)
> Alt+방향키를 zsh 커서 이동용으로 쓰기 위해 Windows Terminal이 가로채지 않도록 설정

```json
"actions": [
    { "command": "unbound", "keys": "alt+left" },
    { "command": "unbound", "keys": "alt+right" }
],
```

---

## 2. Zsh + Oh My Zsh + Powerlevel10k

> Kali는 zsh 기본 설치되어 있으므로 zsh 설치 스킵

```bash
# Oh My Zsh 설치
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Powerlevel10k 테마
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git \
  ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k

# 플러그인
git clone https://github.com/zsh-users/zsh-autosuggestions \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

git clone https://github.com/zsh-users/zsh-syntax-highlighting \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# ~/.zshrc 설정
sed -i 's/ZSH_THEME="robbyrussell"/ZSH_THEME="powerlevel10k\/powerlevel10k"/' ~/.zshrc
sed -i 's/plugins=(git)/plugins=(git zsh-autosuggestions zsh-syntax-highlighting z sudo)/' ~/.zshrc

# Go bin + pip 설정
echo 'export PATH=$PATH:~/go/bin' >> ~/.zshrc
echo 'export PIP_BREAK_SYSTEM_PACKAGES=1' >> ~/.zshrc

# Alt+방향키 단어 단위 커서 이동
cat >> ~/.zshrc << 'EOF'

# ── Alt+방향키 단어 단위 이동 ──────────────────────
bindkey "^[[1;3C" forward-word            # Alt+Right
bindkey "^[[1;3D" backward-word           # Alt+Left
bindkey "^[[1;3A" history-search-backward # Alt+Up
bindkey "^[[1;3B" history-search-forward  # Alt+Down
EOF

# tmux 자동 실행
cat >> ~/.zshrc << 'EOF'

# ── tmux 자동 실행 ─────────────────────────────────
if [ -z "$TMUX" ]; then
  tmux attach -t main 2>/dev/null || tmux new -s main
fi
EOF

source ~/.zshrc
# → p10k 설정 마법사 자동 실행, 폰트 깨짐 없으면 y로 진행
```

---

## 3. tmux + Nord 테마

```bash
# TPM 설치
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm

# Nord 테마 직접 클론 (TPM으로 설치 시 다운로드 실패 케이스 있음)
git clone https://github.com/arcticicestudio/nord-tmux.git \
  ~/.tmux/plugins/tmux-nord
```

### ~/.tmux.conf 전체
```bash
cat > ~/.tmux.conf << 'EOF'
# ── 기본 설정 ──────────────────────────────────────
set -g default-terminal "tmux-256color"
set -ag terminal-overrides ",xterm-256color:RGB"
set -g base-index 1
setw -g pane-base-index 1
set -g renumber-windows on
set -g history-limit 50000
set -g mouse on
set -sg escape-time 0

# ── 프리픽스 ───────────────────────────────────────
set -g prefix C-a
unbind C-b
bind C-a send-prefix

# ── 창 분할 (Alt, 프리픽스 없이) ──────────────────
bind -n M-v split-window -h -c "#{pane_current_path}"
bind -n M-s split-window -v -c "#{pane_current_path}"
bind -n M-c new-window -c "#{pane_current_path}"

# ── pane 이동 (Ctrl+방향키) ────────────────────────
unbind -n M-Left
unbind -n M-Right
unbind -n M-Up
unbind -n M-Down
unbind -n C-Left
unbind -n C-Right
unbind -n C-Up
unbind -n C-Down
bind -n C-Left  select-pane -L
bind -n C-Right select-pane -R
bind -n C-Up    select-pane -U
bind -n C-Down  select-pane -D

# ── 윈도우 이동 (Alt+숫자) ─────────────────────────
bind -n M-1 select-window -t 1
bind -n M-2 select-window -t 2
bind -n M-3 select-window -t 3
bind -n M-4 select-window -t 4

# ── pane 닫기 (Alt+w) ──────────────────────────────
bind -n M-w kill-pane

# ── 설정 리로드 ────────────────────────────────────
bind r source-file ~/.tmux.conf \; display "Reloaded!"

# ── 플러그인 ───────────────────────────────────────
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'tmux-plugins/tmux-yank'
set -g @plugin 'arcticicestudio/nord-tmux'

run '~/.tmux/plugins/tpm/tpm'
EOF

tmux source ~/.tmux.conf
```

### 플러그인 설치
```
tmux 안에서: Ctrl+a → Shift+I
설치 완료 후: Ctrl+a → r
```

### 단축키 요약

| 동작 | 키 |
|------|-----|
| 좌우 분할 | `Alt+v` |
| 상하 분할 | `Alt+s` |
| 새 윈도우 | `Alt+c` |
| pane 이동 | `Ctrl+방향키` |
| 단어 단위 커서 이동 | `Alt+Left / Right` |
| 히스토리 검색 | `Alt+Up / Down` |
| 윈도우 이동 | `Alt+숫자` |
| pane 닫기 | `Alt+w` |
| 설정 리로드 | `Ctrl+a → r` |

---

## 4. HTB 툴 설치

```bash
# Kali Web 툴 메타패키지 (nmap, sqlmap, gobuster, nikto, burpsuite 등 포함)
sudo apt install -y kali-tools-web

# Go 설치
sudo apt install -y golang-go

# Go 기반 툴
go install github.com/ffuf/ffuf/v2@latest
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
```

---

## 5. HTB 디렉토리 + alias

```bash
mkdir -p ~/htb/{machines,challenges,tools}

cat >> ~/.zshrc << 'EOF'

# ── HTB ───────────────────────────────────────────
alias htb='cd ~/htb/machines'
alias nse='ls /usr/share/nmap/scripts | grep'

htbscan() {
  mkdir -p ~/htb/machines/$1
  sudo nmap -sC -sV -oA ~/htb/machines/$1/initial $2
}

alias vpncheck='ip addr show tun0 2>/dev/null | grep inet'
EOF

source ~/.zshrc
```

### alias 요약

| alias | 동작 |
|-------|------|
| `htb` | `~/htb/machines` 이동 |
| `nse <keyword>` | nmap 스크립트 검색 |
| `htbscan <머신명> <IP>` | nmap 자동 스캔 + 결과 저장 |
| `vpncheck` | HTB VPN tun0 IP 확인 |

---

## 6. pip 설정

> Ubuntu 24.04 / Kali 최신은 시스템 Python 보호 정책으로 pip 기본 차단
> WSL2는 격리 환경이므로 break-system-packages 사용해도 무방

```bash
# ~/.zshrc에 추가 (이미 위에서 설정됨)
echo 'export PIP_BREAK_SYSTEM_PACKAGES=1' >> ~/.zshrc

# 이후 그냥 사용
pip install <패키지>
```

---

## 7. HTB 시작 루틴

```bash
# 1. VPN 연결
sudo openvpn ~/htb/your.ovpn &

# 2. VPN 확인
vpncheck

# 3. 머신 디렉토리 이동
htb && mkdir <머신명> && cd <머신명>

# 4. 스캔 시작
htbscan <머신명> <IP>
```
# WSL2 클립보드 연동 문제 해결

> **환경:** Windows 11 + WSL2 (Kali Linux)
> **증상:** WSL 터미널에서 복사한 텍스트가 외부 앱에서 깨져서 붙여넣어짐

---

## 원인

WSL2 ↔ Windows 클립보드 interop이 비활성화된 상태.  
`/etc/wsl.conf` 에 interop 설정이 없거나 disabled 상태일 때 발생.

---

## 진단

```bash
# clip.exe 연동 확인
echo "test" | clip.exe
```

- 정상: Windows 클립보드에 "test" 복사됨
- 비정상: 에러 또는 아무 반응 없음

```bash
# wsl.conf 확인
cat /etc/wsl.conf
```

---

## 해결

### 1. wsl.conf 설정 추가

```bash
sudo tee /etc/wsl.conf > /dev/null << 'EOF'
[interop]
enabled=true
appendWindowsPath=true
EOF
```

### 2. WSL 재시작 (PowerShell)

```powershell
wsl --shutdown
```

이후 Kali 터미널 다시 열기.

### 3. 동작 확인

```bash
echo "클립보드 테스트" | clip.exe
```

Windows 메모장 등에 붙여넣기로 확인.

---

## 참고

| 방향 | 상태 |
|------|------|
| Windows → WSL | 기본 동작 |
| WSL → Windows | interop 설정 필요 |

---

## 트러블슈팅

| 문제 | 해결 |
|------|------|
| `clip.exe: command not found` | `appendWindowsPath=true` 설정 후 재시작 |
| 설정 후에도 안 됨 | `wsl --shutdown` 후 재시작 필수 |
| 한글 깨짐 | 터미널 인코딩 `UTF-8` 확인 |

---

## 트러블슈팅

| 문제 | 해결 |
|------|------|
| tmux-nord TPM 다운로드 실패 | `git clone https://github.com/arcticicestudio/nord-tmux.git ~/.tmux/plugins/tmux-nord` 직접 클론 |
| Kali 프로필 Windows Terminal 미등록 | `"source": "Microsoft.WSL"` → `"commandline": "wsl -d kali-linux"` 로 교체 |
| nmap/sqlmap command not found | `sudo apt install -y nmap sqlmap` 재설치 |
| ffuf/httpx command not found | `echo 'export PATH=$PATH:~/go/bin' >> ~/.zshrc && source ~/.zshrc` |
| Ctrl+방향키 tmux로 전달 안 됨 | Windows Terminal `"actions"` 에 `unbound` 추가 |
| Alt+방향키 대문자 출력됨 | `~/.zshrc` 에 `bindkey "^[[1;3C"` 등 escape sequence 매핑 추가 |
