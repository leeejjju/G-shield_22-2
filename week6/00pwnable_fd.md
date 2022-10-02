# HW00) pwnable fd


## week6 

<hr/>

[pwnable](https://pwnable.kr/play.php)

<br>
<br>

스터디장님이 기깔나게 시연해주신 00번 문제. pwnable의 가장 첫번째 카드 문제이기도 하다. 

![cardIMG](https://pwnable.kr/img/fd.png)


짱귀엽다. 
<hr>

<br>
<br>

## 풀이법은 다음과 같다. 

<br>
<br>

일단 powershell에서 

    ssh fd@pwnable.kr -p2222 

을 입력하여 접속하자. (pw:guest)

<br>
<br>

`ls -l` 명령어를 통해 확인하면 우리가 알고싶은 flag는 권한이 부족해 직접 읽을 수 없다.

![list](/img/00_list.jpg)


그러니 제시된 fd와 fd.c를 이용해 flag를 알아내 볼 것이다. 

<br>
<br>

fd.c의 내용은 다음과 같다.


![fd.c](/img/00_fd.c.jpg)

<br>
<br>

실행시 인자로 받는 첫번째 argument에서 `0x1234` 를 뺀 값을 atoi함수를 통해 int형으로 바꾸어, `fd`, 즉 File Descriptor로 사용하고 있다.

    int fd = atoi( argv[1] ) - 0x1234;

그리고 그 친구는 `buf` 공간에 32bit만큼의 내용을... 어찌할지를 결정한다. 

> read 함수의 `fd`자리에서 0은 stdin, 즉 입력을 지시하며 1은 stdout, 2는 stderr를 의미한다. 

<br>
<br>

더 밑을 보면 if문이 나오는데, 그 안에서 우리가 원하는 `flag`를 짜잔 프린트해주게 되어있다. 

즉 이 if문을 실행되게 하는 것을 목적으로 두면 되겠다. 

    if(!strcmp("LETMEWIN\n", buf)){
                printf("good job :)\n");
                system("/bin/cat flag");
                exit(0);
        }

if문의 실행 조건은 `LETMEWIN\n`과 `buf`가 같은 문자열일 것. 

이렇게 되면 우리는 어떻게든 `buf`에 `LETMEWIN`을 담고싶어진다. 

<br>
<br>

fd로 돌아와서, 윗줄에서는 read함수가 fd와 함께 call되고있다. 

    len = read(fd, buf, 32);
    
이 때 `fd`에 stdin을 의미하는 0을 넣어준다면 read함수를 통해 우리는 `buf`에 원하는 값을 넣을 수 있을 것이다. 

그렇다. `LETMEWIN`을 집어넣을 여지가 생긴다!! 

<br>
<br>

그럼... 어떻게 fd를 0으로 만드느냐? 위에서 잠깐 지켜봤듯이 

    int fd = atoi( argv[1] ) - 0x1234;

`argv[1]`, 즉 실행시 첫번째 인자로 `0x1234`를 넣어주면 된당. 

그러면 이 프로그램은 이어서 read함수에 의해 입력값을 받아 `buf`에 저장할 것이고, 그 `buf`가 `LETMEWIN\n`과 일치한다면 우리가 필요로 하는 `flag`를 출력해 줄 것이다. 

<br>
<br>

그런데ㄱ잠깐!!! 인자는 기본적으로 string형태로 받아진다. 그니까 hex인 `0x1234`과 일치하는 string... `4660`을 넣어주면 되것다. 

![hexToString](/img/00_0x1234.jpg)

<br>
<br>

그럼 가보자고

<br>
<br>

<hr/>

<br>
<br>

    ./fd 4660

이렇게 인자로 4660을 주면서 프로그램을 실행시키면 read함수가 실행되어 입력 대기 상태가 된다. 이 때,

    LETMEWIN



를 입력해주면 된다. 뒤에 `\n`는 짜피 엔터로 입력을 종료시킬 때 들어갈거다. 

<br>
<br>

그러믄 짜잔티비. flag를 알아냈다! 

![flag](/img/00_flag.jpg)

<br>
<br>

    mommy! I think I know what a file descriptor is!!




<br>
<br>
<br>

good job :)



