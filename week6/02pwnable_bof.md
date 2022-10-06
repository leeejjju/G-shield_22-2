# HW02) pwnable bof


## week6 

<hr/>

[pwnable](https://pwnable.kr/play.php)

<br>
<br>

이번엔 방학에 찍먹해본 bof! 

![cardIMG](https://pwnable.kr/img/bof.png)


얘 역시 짱귀엽다. 
<hr>

<br>
<br>

## 문제풀기를 시작하기 위한... 세팅. 

<br>
<br>

카드를 쨘 클릭하면 이렇게 안내가 뜬다. 

    Nana told me that buffer overflow is one of the most common software vulnerability.Is that true?

    Download : http://pwnable.kr/bin/bof
    Download : http://pwnable.kr/bin/bof.c

    Running at : nc pwnable.kr 9000


원래 ssh 써서 간편하게 접속했는데 이거는 뭐가 좀 더 많음. 

일단 위 링크를 브라우저에 입력해 실행파일과 c 파일을 다운받고, 걔를 분석해 페이로드를 짠 뒤... 

터미널에서 아래 명령어를 쳐서 접속한 서버에 쏴서 flag를 따내는 듯!  

<br>

근데 서버 접속을 위한 nc 명령어(Netcat)가 window환경에서는 기본 제공되지 않는다. 




그래서 따로 깔아줘야 함. 

[[다운로드 링크]](https://eternallybored.org/misc/netcat/)

<br>

근데 윈도우 보안 어쩌구 때문에 다운이 잘 안 받아질것이다. 

설정 잠깐 들어가서 쇽 보안 중지하고, 쇽 다운받고, 거시기 하면 된다. 

[[참조: Netcat 다운로드]](https://ndb796.tistory.com/85)

[[참조: 보안이슈 해결]](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=govlaos3444&logNo=221421842360)

<br>

***

## 그러면 어케 푸냐??


<br>
<br>


일단 다운받은 c 파일이랑 실행파일을 뜯어봐야 쓰것다. 



근데 실행파일 저기 링크대로 다운받으니 제대로 실행이 안 되더라. 또 윈도우 이슈인가. 

걍 gcc써서 다시 컴파일했당. 
<br>
<br>


접근성이 좋은 high level 코드부터 보자.

bof.c의 내용은 다음과 같다.


![bof.c](/img/02_bof.c.jpg)


<br>
<br>


main함수를 보면, 우선 인자를 필요로 한다는 것을 확인 가능하다. 

그리고 `0xdeadbeef`를 파라미터로 주며 `func` 함수를 실행한다. 벌써 수상한 애가 나왔다. `0xdeadbeef`... 이게몰까. 

<br>

일단 함수를 call했으니 우리도 따라가보자. `func`는 뭐 하는 친구인가... 

    void func(int key){

        char overflowme[32];
        printf("overflow me : ");
        gets(overflowme);       // smash me!
        if(key == 0xcafebabe){
                system("/bin/sh");
        }
        else{
                printf("Nah..\n");
        }
    }


보자보자.

일단 int형태의 파라미터를 `key`라는 변수로 받아온다. 

그리고 gets 함수를 통해 입력값을 받아 `overflowme`에 저장한다. 32byte의 char배열로 이루어진 문자열이다. 

그리고 파라미터로 받은 key와 `0xcafebabe`라는 value를 비교하여 둘이 동일하다면 쉘코드를 실행한다. 

그럼 서버의 해당 디렉토리에서 파일들을 열람할 수 있을 것이고, 아마 거기에 flag가 있지 않을까 싶다! 



<br>
<br>


근데말이다. 우리가 원하는 것을 실행시키기 위한 조건 부분은 `key`와 상수 변수를 비교한다. 그리고 `key`는 함수를 call할 때 받아와진다. 

우리가 접근 가능한 여지가 없다는 것이다. 

우리의 입력값이 영향을 미치는 `overflowme`는 실제 중요한 부분에 어떤 관련도 없음을 확인할 수 있었다.

그럼... 우짜나?

문자열 변수명부터 힌트가 있다. overflow me! 오버플로우를 일으켜봐라~ 막 이러고 있지 않은가. 

`key`의 값과 상수값을 비교해 원하는 실행이 일어난다는 것은, `key`의 값을 조작해야 한다는 것. `key`의 값을 조작하기 위한 방법이 overflowme를 이용한 bof인 것이다. 


<br>
<br>




이쯤에서 bof, buffer overflow가 뭐하는 친구인지 간략하게 정리해보자.

배웠던 바로는... 뭐더라?

메모리 속 stack의 구조를 이용한 공격. 메모리에서 프로그램 실행시 할당되는 segment의 구조는 다음과 같은데... 대충 이렇게 무지개떡마냥 쌓여있다 치면

    (High address; 0xffffffff)

    ~ ~ ~ ~ ~

    stack 영역: 지역변수, 매개변수 공간. 선언된 순서대로 아래로 쌓임.  리틀 엔디안 방식이라 하위 바이트부터 들어간당. 

    heap 영역: 동적할당한 애들이 여기에 위로 쌓임. 

    data 영역: static, global 변수들

    code 영역: 어셈블리 형태

    ~ ~ ~ ~ ~

    (Low address; 0x00000000)

그렇다구 한당. 

여기서 high level 코드에서 선언된 변수들은 맨 위의 stack영역에 차곡차곡 자리를 배정 받게 된다.  

차곡차곡이 중요하다. 선언 순서대로 쌓이므로 bof.c에서 func 함수로 들어가면, 여기의 stack은 이런 구조로 되어있을 것이다. 

일단 파라미터로 들어오는 key(지역변수 취급되니까 얘도 변수임), 그리고 선언된 overflowme. 

    (High address)

    RET: 반환 주소값. 다음에 실행할 명령어가 위치한 메모리 주소값이 저장됨. 함수의 경우 return될 때 돌아갈 곳의 주소

    SFP: stack 베이스 값. 스택 주소값 계산시 기준으로 쓰이는 프레임 포인트. 

    key

    overflowme

    (Low address)


이 때 배열 특인데, 배정된 공간을 넘게 담아도 입력된다고 한다. 정확히 말하면 흘러 넘쳐서 위쪽 공간까지 침범한다. 

이걸 이용하는게 bof 공격이다. 버퍼오버플로우... 

<br>
<br>

이제 위 구조도의 overflowme 위치를 보자. 우리가 조작하고싶은 key의 아래쪽이다. 

그 말인즉슨 **overflowme에 배정된 공간 32byte를 넘게 뭘 넣으면 그게 흘러넘쳐 key까지 도달시킬 수 있다는 뜻**!! 

여기서 정확히 key 값을 원하는 값으로 덮기 위해서는 메모리상 두 변수들의 거리를 정확히 알아야 할 필요가 있다.

 그 사이 거리만큼의 dummy값과 key를 엎을 원하는 값을 연속된 문자열로 overflowme에 넣어주면, 목적을 달성 할 수 있을 것이기 때문 





<br>
<br>


그러니! 일단 key와 overflowme 사이의 거리를 계산해보자. 

gdb를 이용해 어셈블리 영역을 확인한다면... 


main은 이렇게
![disasMain](/img/02_disasMain.jpg)

func함수는 이렇게
![disasFunc](/img/02_disasFunc.jpg)

되어있는 것을 확인 가능하다. 그럼 key와 overflowme가 다루어지는 부분들을 보자... 


<br>
<br>


gets 함수가 call되는 지점이다.

    0x00401478 <+24>:    call   0x403ad0 <gets>

보면 마치 gets함수로 받아와진 친구가 들어갈 자리라는 듯 주소가 하나 쨘 있지 않은가. 혹시 이게 overflowme 주소...? 물론 추측이다. 

일단 킵. `0x403ad0`

<br>


이건 key가 `0xcafebabe`와 비교되고있는 coml문 부분이다. 

    0x0040147d <+29>:    cmpl   $0xcafebabe,0x8(%ebp)

첫번째 operand는 어디서 많이 본 상수값, 두번째 operand는 0x8(%ebp)이다. 

예상컨대 현시점 ebp 값에 0x8을 더한 것이 아닐까... 그니까 걔가 key가 아닐까 라는 작은 소망을 품어본다. 그럼 ebp값을 알아야겠다. 


레지스터들의 값을 확인해보자. 
![infoReg](/img/02_infoReg.jpg)

이 부분. 

    ebp            0x61ff08 0x61ff08

`0x61ff08`라는 주소가 나왔다. 그럼 위의 instuction에서 key로 추정되는 주소는, 

    (gdb) p/x 0x61ff08 + 0x8
    $3 = 0x61ff10


<br>
<br>

그으러어며언... 

아까 추정한 overflowme의 주소는 `0x403ad0`

그리고 방금 추정한 key의 주소는 `0x61ff10`

더 높은 주소에 있는 key에서 overflowme를 빼보자... 

    (gdb) p/d 0x61ff10 - 0x403ad0
    $5 = 2212928

사이의...ㄱㅓ리가... 2212928...?

<br>

이겠냐?

<br>

실패.









<br>
<br>

<hr/>

<br>
<br>



![none-success]()

<br>

    DOUMM.....



<br>
<br>
<br>


:(




