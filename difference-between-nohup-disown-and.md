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

## Shell과 3개의 stream(stdin, stdout, stderr)

- 여기서 잠깐, 위에서 말한 프로세스가 shell로부터 stdin, stdout, stderr을 상속받는 다는 것은 무슨 이야기 일까요?
	- 여기서 말하는 `stdin`, `stdout`, `stderr`은 stream이라고 불리는 것들입니다.
    	- `stdin`은 받은 입력 값을 프로그램에 나타내주는 stream입니다. (예를 들면, 비밀번호를 입력하기 위한 프롬프트 같은 것들이 있습니다.)
        - `stdout`은 모든 출력값들이 가는 곳입니다. C로 프로그래밍을 할 때, `printf`를 생각해보시거나 Java로 프로그래밍을 할 때는 `System.out.println` 파이썬으로 프로그래밍할 때는 `print`를 생각하면 됩니다.
        - `stderr`은 또 다른 출력 채널입니다. 주로 디버깅 정보를 출력하거나 에러를 출력하는데에 쓰입니다.

우리가 다음 명령어를 실행했다고 가정해봅시다.

```bash
echo foo
```

다음은 `foo`의 출력입니다. 무엇이 어떻게 돌아가는지에 대한 다이어그램입니다.

![stdinstdoutstderr.png](https://images.velog.io/post-images/jakeseo_me/ecf11ca0-6d70-11e9-8ea3-211446efebf3/stdinstdoutstderr.png)

`echo` 명령어는 커멘드라인의 인자 `foo`를 받아들입니다(stdin에서 받아들이는 것이 아닙니다.). 그리고 출력 값을 `stdout`으로 던져줍니다.

## 리다이렉팅(Redirecting)
### 리다이렉팅 출력(Redirecting output)

지금 다음 명령어를 실행했다고 가정해보세요.

```bash
echo foo > temp.txt
ls
```

이 명령어는 `temp.txt`라는 새로운 파일을 생성했을 것입니다. 안의 내용을 보기 위해서는 `cat temp.txt` 명령어를 이용하면 됩니다. 무슨 일이 일어나나요? `>`가 하는 일은 무엇일까요?

![redirecting.png](https://images.velog.io/post-images/jakeseo_me/b9b41670-6d71-11e9-8ea3-211446efebf3/redirecting.png)

우리는 `echo`의 출력물을 콘솔에서 `temp.txt`라는 파일로 리다이렉트 했습니다. 기존에 있는 파일에 추가하고 싶으면 `>>`기호를 사용하시면 됩니다.

```bash
echo " bar" >> temp.txt
cat temp.txt
```

## 파이프(Pipe)

우리는 어떤 프로그램의 출력 결과를 다른 프로그램의 입력 값으로 쓸 수 있을까요? 이럴 때 쓰는게 바로 파이프입니다.

다음 명령어를 봅시다.

```bash
echo "foo bar baz" | wc -w
```

echo 명령어를 수행하면 `3`개의 단어를 결과로 받을 것입니다. `|` 문자는 `echo`로 인해 stdout에서 나온 결과를 `wc` 명령어의 `stdin`에 넣겠다는 것과 동일한 의미입니다.

![pipe.png](https://images.velog.io/post-images/jakeseo_me/6c167060-6d72-11e9-bebc-8307bd808a6b/pipe.png)

파이프를 이용해, 우리는 많은 간단한 프로그램들을 연결하고 매우 강력한 기능을 수행하게 할 수 있습니다.

> 참고 : wc 명령어는 어떤 파일의 길이를 지정한 옵션에 맞추어 반환해주는 명령어입니다. -l 옵션은 line 숫자, -w 는 단어 숫자, -c 는 바이트 숫자를 의미합니다.

## 다중 커멘드와 변수

파이프와 함께 커멘드 체인을 구성하는 것 뿐도 가능하고, 원한다면 명령어를 순차적으로 실행할 수도 있습니다. 다음과 같이 하시면 됩니다.

```bash
mkdir foo; touch foo/file.txt
```

> 참고 : 여기서 touch 명령어는 단순히 빈 파일을 생성하기 위해 쓰였습니다. 원래 touch는 파일의 타임스탬프를 업데이트 하기 위해 씁니다. 자세한 설명과 용례는 [여기](https://vitux.com/8-common-uses-of-the-linux-touch-command/)에서 확인하세요.

위 명령어를 사용하면 우리는 `foo`라 불리는 디렉토리에 `file.txt`파일을 생성할 수 있습니다. `;` 문자는 터미널에게 처음 명령어를 실행하고 이후 따로 두번째 명령어를 실행하라는 신호를 보냅니다. 단지 2개의 명령어로 제한되어 있진 않습니다. 원하는 만큼의 숫자의 명령어를 입력하고 명령어 사이를 `;`로 나누면 됩니다.

마지막으로, 명령어의 결과를 *변수*에 저장할 수 있습니다. 그리고 그 변수를 같은 시퀀스의 커멘드에서 사용할 수 있습니다.

```bash
myvar="foo"; echo $mybar | tr '[:lower]' '[:upper]'
```

명령어들의 순서를 쪼개봅시다.

