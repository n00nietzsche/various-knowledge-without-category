> nohup, disown, &는 언제 어떻게 써야될까?

# 글을 쓰는 이유

리눅스 서버에서 노드를 실행시킬 때, 사실 내부에서 무슨 일이 일어나는지 모르고 nohup ./노드실행파일 & 명령어로 그냥 백그라운드 실행했던 때가 많았다. 그래서 인터넷 검색을 해보니 [이 링크](https://unix.stackexchange.com/questions/3886/difference-between-nohup-disown-and)가 나왔고 누가 엄청 친절하게 무슨 일이 일어나는지 전부 답변한 글이 있었다. 근데 그 답변으로는 모든 설명을 하기엔 모자라서 내가 모르는 부분은 또 인터넷으로 찾아서 정리해본다.

# nohup, disown, &

```bash
$ nohup foo
```

```bash
$ foo &
```

```bash
$ foo &
$ disown
```

의 차이는 무엇일까요?

일단, 프로그램이 터미널과 연결된 interactive shell에서 리다이렉팅 없이 그냥 `foo`를 타이핑했다고 가정해봅시다.

```bash
$ foo
```

- `foo`라는 실행중인 프로세스가 만들어집니다.
- 프로세스는 stdin, stdout, stderr을 shell로부터 상속받습니다. 그 결과, 이 프로세스는 같은 터미널에 연결됩니다.
- 만일 shell이 `SIGHUP`라는 명령어를 받게되면, `SIGHUP`을 프로세스로 보냅니다. (이런 일은 일반적으로 프로세스를 끄려고 일어납니다.)
- 그렇지 않다면, (블락된) shell은 프로세스가 꺼질 때까지 기다립니다.
