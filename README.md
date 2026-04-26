# ft_irc — IRC Server

> 42Seoul 과제. C++ / POSIX Socket 기반 IRC 서버 구현.  
> RFC 1459 프로토콜을 준수하며 실제 IRC 클라이언트(irssi)로 접속 및 동작 검증.

---

## Preview

<!-- screenshot 또는 irssi 접속 화면 캡처를 여기에 추가 -->

---

## Architecture

### AMessage — Abstract Command Class (본인 설계)

모든 IRC 커맨드를 단일 인터페이스로 처리하기 위해 `AMessage` 추상 클래스를 설계했다.  
Client 객체는 커맨드 종류를 알 필요 없이 아래 하나의 로직으로 모든 커맨드를 처리한다.

```cpp
AMessage* message = AMessage::GetMessageObject(this, buff); // Factory
if (message != NULL)
{
    message->ExecuteCommand(); // Polymorphism
    delete message;
}
```

- `GetMessageObject()` — 수신 버퍼에서 커맨드 종류를 판별해 해당 커맨드 객체를 생성하는 Factory Method
- `ExecuteCommand()` — 순수 가상 함수. 각 커맨드 클래스가 구체 동작을 구현
- Client는 `AMessage*` 포인터만 다루므로 커맨드 추가 시 기존 로직 수정 불필요 (OCP)

### Message Parser (본인 구현)

Client 객체가 수신 버퍼를 감시하다 `\r\n` 발견 시 raw 메시지를 `AMessage::GetMessageObject()`에 전달하면,
메세지를 커멘드/파라미터로 분리하여 커맨드 객체를 생성한다.

---

## Implemented Commands

| Command | Description |
|---------|-------------|
| `PASS` | 서버 접속 비밀번호 인증 |
| `NICK` | 닉네임 설정 및 변경 |
| `USER` | 사용자 등록 |
| `PING` | 연결 유지 확인 |
| `JOIN` | 채널 입장 (없으면 생성) |
| `PART` | 채널 퇴장 |
| `PRIVMSG` | 유저/채널 메시지 전송 |
| `KICK` | 채널에서 유저 강제 퇴장 |
| `INVITE` | 채널에 유저 초대 |
| `TOPIC` | 채널 주제 설정/조회 |
| `MODE` | 채널/유저 모드 설정 |
| `QUIT` | 서버 연결 종료 |

---

## Number Baseball Bot (본인 구현)

봇을 서버 내장 기능으로 구현하지 않고 **독립적인 봇 클라이언트**로 설계했다.  
봇은 일반 클라이언트와 동일하게 서버에 접속하며, 채널에 초대하는 방식으로 실행된다.

이 구조를 선택한 이유: 서버에 종속된 부속 기능으로 구현하면 서버 코드와 결합도가 높아진다.  
독립 클라이언트 형태로 분리하면 어떤 채널에도 범용적으로 초대·사용할 수 있고, 서버 코드를 수정하지 않고도 봇 기능을 확장할 수 있다.

---

## I/O Multiplexing

`epoll`을 사용하여 단일 스레드에서 다중 클라이언트 접속을 처리한다.  
클라이언트 이벤트 발생 시에만 해당 소켓을 처리하므로 블로킹 없이 다수의 연결을 유지한다.

---

## Build & Run

**Requirements**: g++ or clang++, Linux (epoll)

```bash
make
./ircserv <port> <password>
make clean    # object files 제거
make fclean   # 전체 빌드 산출물 제거
```

**irssi 접속 예시**
```
/connect localhost <port> <password>
```

---

## What I Learned

| 개념 | 내용 |
|------|------|
| TCP Socket | 소켓 생성 → bind → listen → accept → send/recv 전 과정 |
| epoll | Level-triggered 이벤트 감지로 단일 스레드 다중 접속 처리 |
| IRC Protocol | RFC 1459 메시지 포맷 (`:<prefix> <command> <params>\r\n`) |
| Factory Method | 커맨드 문자열로부터 적절한 객체를 생성하는 패턴 적용 |
| OCP | 추상 클래스 설계로 커맨드 추가 시 기존 코드 수정 없이 확장 가능한 구조 구현 |

---

## Team

3인 팀 프로젝트 (42Seoul)

| 담당 | 내용 |
|------|------|
| 본인 | `AMessage` 추상 클래스 및 커맨드 클래스 포맷 설계, 메시지 파서 구현, 숫자야구 봇 구현, 커맨드 수정 및 보완 |
| 팀원 | 소켓 서버 구조, epoll I/O 처리, 각 커맨드 구현 |

---

## Stack

- **Language**: C++98
- **Networking**: POSIX Socket, epoll
- **Protocol**: IRC (RFC 1459)
- **Build**: Makefile
- **Test**: irssi, echo client
