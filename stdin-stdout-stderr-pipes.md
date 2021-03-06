# 들어가기 전에

- 이 포스팅은 https://github.com/kennyyu/bootcamp-unix/wiki/stdin,-stdout,-stderr,-and-pipes 에 있는 포스팅을 번역한 것입니다. 오역이나 의역이 있을 수 있습니다. 지적해주시면 확인 후 바로 정정하겠습니다.

- original source of this posting is from https://github.com/kennyyu/bootcamp-unix/wiki/stdin,-stdout,-stderr,-and-pipes If the original author requests deletion, it will be deleted immediately.

- Translated by Jake Seo (서진규)

	- https://velog.io/@jakeseo_me
	- https://github.com/n00nietzsche

> 유닉스의 stdin, stdout, stderr 그리고 pipes에 대해 알아보자!

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

> 역자 주 : wc 명령어는 어떤 파일의 길이를 지정한 옵션에 맞추어 반환해주는 명령어입니다. -l 옵션은 line 숫자, -w 는 단어 숫자, -c 는 바이트 숫자를 의미합니다.

## 다중 커멘드와 변수

파이프와 함께 커멘드 체인을 구성하는 것 뿐도 가능하고, 원한다면 명령어를 순차적으로 실행할 수도 있습니다. 다음과 같이 하시면 됩니다.

```bash
mkdir foo; touch foo/file.txt
```

> 역자 주 : 여기서 touch 명령어는 단순히 빈 파일을 생성하기 위해 쓰였습니다. 원래 touch는 파일의 타임스탬프를 업데이트 하기 위해 씁니다. 자세한 설명과 용례는 [여기](https://vitux.com/8-common-uses-of-the-linux-touch-command/)에서 확인하세요.

위 명령어를 사용하면 우리는 `foo`라 불리는 디렉토리에 `file.txt`파일을 생성할 수 있습니다. `;` 문자는 터미널에게 처음 명령어를 실행하고 이후 따로 두번째 명령어를 실행하라는 신호를 보냅니다. 단지 2개의 명령어로 제한되어 있진 않습니다. 원하는 만큼의 숫자의 명령어를 입력하고 명령어 사이를 `;`로 나누면 됩니다.

마지막으로, 명령어의 결과를 *변수*에 저장할 수 있습니다. 그리고 그 변수를 같은 시퀀스의 커멘드에서 사용할 수 있습니다.

```bash
myvar="foo"; echo $mybar | tr '[:lower:]' '[:upper:]'
```

> 역자 주 : tr 명령어는 문자열을 translate 하는 명령어로 앞의 조건에 해당하는 문자열들을 찾아내어, 뒤의 문자열의 케이스로 바꿉니다. 위의 명령어는 `[:lower:]`에서 `[:upper:]` 즉 소문자를 찾아 대문자로 바꾸는 것입니다.

명령어들의 순서를 쪼개봅시다.

1. 처음에, `foo` 문자열 값을 갖기 위해서 우리는 `myvar` 변수를 할당했습니다. `myvar`와 `=`와 원하는 값 사이에 스페이스가 들어올 수 없다는 것을 인지하여야 합니다.
2. 다음으로 `myvar` 문자의 값을 `echo`로 출력합니다. 변수의 값을 얻기 위해 `$` 문자를 변수명 앞에 적습니다.
3. `tr '[:lower:]' '[:upper:]'`를 이용하여 `echo` 명령어의 결과를 파이프합니다. 이 것은 모든 소문자를 대문자로 바꾸라는 명령입니다. 이 결과로 우리는 `FOO`라는 대문자 문자를 얻습니다.

어떤 명령어의 결과를 변수에 넣고 싶다면, 다음과 같은 문법을 쓰시면 됩니다: `myvar=$(... 실행 시키고 싶은 커멘드들...)` 예를 들면 다음과 같은 명령어입니다.

```bash
bar=$(echo "kenny"); echo "Hello $bar"
```

출력은 `Hello Kenny`가 나올 것입니다.
