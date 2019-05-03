
# 들어가기 전에

- 이 포스팅은 https://unix.stackexchange.com/questions/3886/difference-between-nohup-disown-and 에 있는 포스팅을 번역한 것입니다. 오역이나 의역이 있을 수 있습니다. 지적해주시면 확인 후 바로 정정하겠습니다.

- original source of this posting is from https://unix.stackexchange.com/questions/3886/difference-between-nohup-disown-and If the original author requests deletion, it will be deleted immediately.

- Translated by Jake Seo (서진규)

	- https://velog.io/@jakeseo_me
	- https://github.com/n00nietzsche
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

- `foo`프로세스가 만들어지고 실행됩니다. 
- 프로세스는 stdin, stdout, stderr을 shell로부터 상속받습니다. 그 결과, 이 프로세스는 같은 터미널에 연결됩니다.
- 만일 shell이 `SIGHUP`라는 명령어를 받게되면, `SIGHUP`을 프로세스로 보냅니다. (이런 일은 일반적으로 프로세스를 끄려고 일어납니다.)
- 그렇지 않다면, (블락된) shell은 프로세스가 꺼질 때까지 기다립니다.

> 여기서 나온 stdin, stdout, stderr이 잘 이해되지 않는다면 [이 번역문서](https://velog.io/@jakeseo_me/%EC%9C%A0%EB%8B%89%EC%8A%A4%EC%9D%98-stdin-stdout-stderr-%EA%B7%B8%EB%A6%AC%EA%B3%A0-pipes%EC%97%90-%EB%8C%80%ED%95%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90)를 읽어보시면 도움이 될 겁니다.

이제, 백그라운드에 프로세스를 넣으면 무슨 일이 일어나는지 알아봅시다. 

- `foo`프로세스가 만들어지고 실행됩니다.
- 프로세스는 stdout/stderr을 shell로부터 상속받습니다. (그래서 아직 터미널에 쓰여집니다.)
- 원칙적으로는 stdin 또한 상속받지만 stdin에서 무언가 읽으려 하자마자 멈춥니다.
- shell이 관리하는 background job의 리스트로 들어갑니다. 이것이 의미하는 것은
	- 먼저 `jobs`와 함께 리스트화됩니다. 그리고 `%n`을 사용하여 접근됩니다. (`n`은 job의 number입니다.)
    - `fg`를 통해 이 프로세스는 foreground job이 될 수도 있습니다. 이 경우에는, `&` 기호를 사용하지 않았던것 처럼 계속됩니다. (그리고 stdin에서 무언가 읽으려해서 작동이 중지되면, terminal에서 읽기를 진행할 수 있습니다.)
    - shell이 `SIGHUP`을 받는다면, `SIGHUP`을 프로세스로 보냅니다. shell과 shell에 적용된 옵션에 달려있는데, shell을 종료할 때, shell은 이 프로세스로 `SIGHUP`을 보냅니다.
    
`disown`은 shell의 job 리스트에서 job을 제거합니다. 그래서 위의 모든 하위 포인트가 더 이상 적용되지 않게 합니다. (shell에서 `SIGHUP`으로 보내진 프로세스도 포함합니다.) 하지만 *아직* 터미널에 연결되어 있다는 것을 알아두세요. 그래서 만일 터미널(`putty`, `xterm`, `ssh`에도 동일하게 일어납니다.)이 꺼지면(destroyed) 프로그램도 stdin에서 내용을 읽어오려하거나 stdout에 무언가를 쓰려는 순간 꺼집니다.

한편, `nohup`이 하는 일은 효율적으로 프로세스를 터미널에서 분리하는 것입니다.

- stdin을 닫습니다. (프로그램은 더이상 입력 값을 읽지 못합니다. 심지어는 foreground에서 실행되고 있더라도 입력 값을 읽을 수 없습니다. stdin은 멈추지만 `EOF`나 에러 코드는 받을 수 있습니다.) 
- stdout이나 stderr의 내용을 `nohup.out`파일에 리다이렉트합니다. 그래서 터미널이 꺼져도 stdout에 무언가를 쓸 수 있습니다.
- `SIGHUP`을 받는 것으로부터 프로세스를 보호합니다. (이름 값을 합니다.)

`nohup`은 shell의 job 컨트롤로부터 프로세스를 제거할 수 없다는 것을 알아두세요 그리고 백그라운드에 넣지도 않습니다. (하지만 foreground `nohup` job은 쓸모가 없습니다. 왜냐하면 일반적으로는 `&`로 백그라운드로 보내기 위해 사용할 것이니까요.) 예를 들면, `disown`과는 다르게, nohup job이 완료되면 shell은 당신에게 정보를 전달합니다. (물론 shell이 종료되기 전까지 nohup 작업이 끝났을 때에 한정합니다.)

요약하자면 :

- `&`는 job을 백그라운드로 보냅니다. stdin을 block합니다. 그리고 shell이 job이 끝날 때까지 기다리지 않게 합니다.
- `disown`은 프로세스를 shell의 job control에서 벗어나게 합니다. 하지만 여전히 터미널과는 연결되어있습니다. shell이 `SIGHUP`을 보내지는 않습니다. 이건 확실히 백그라운드 job에만 적용될 수 있습니다. 왜냐하면 foreground job이 실행중일 때는 접근할 수 없으니까요.
- `nohup`은 터미널에서 프로세스의 연결을 끊습니다. stdout을 `nohup.out`으로 리다이렉팅합니다. 그리고 `SIGHUP`으로부터 프로세스를 보호합니다. 확실한 효과는 프로세스가 더이상 `SIGHUP`를 받지 않는다는 것입니다. 이렇게 되면 job control과는 완전히 독립적인 상태가 됩니다 그리고 원칙적으로는 foreground job에도 `nohup` 명령어를 활용할 수 있습니다. (다만 유용하진 않습니다.)
