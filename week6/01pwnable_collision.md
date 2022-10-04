# HW00) pwnable fd


## week6 

<hr/>

[pwnable](https://pwnable.kr/play.php)

<br>
<br>

혼자 돌진해보는 첫 문제! 

![cardIMG](https://pwnable.kr/img/collision.png)


얘도 짱귀엽다. 
<hr>

<br>
<br>

## 배경지식을 공부해보고 접근하자. 

pwnable은 

    "Daddy told me about cool MD5 hash collision today.
    I wanna do something like that too!"

라고 힌트를 준다. 예상컨대 키워드는 `MD5 hash collision`. 

구글링 가보자고. 

<br>
<br>

열심히 한국어 먼저 찾다보니 [나무위키](https://namu.wiki/w/MD5)에 도달했다. 이 설명에 의하면...

<br>
<br>


## Message-Digest algorithm 5
1. 임의의 길이의 값을 입력받아서 128비트 길이의 해시값을 출력하는 알고리즘. 

2. 단방향 암호화이므로 출력값으로 입력값 찾는건 사실상 불가능

3. 같은 입력값에서는 반드시 같은 출력값이 나오고, 서로 다른 입력값이 같은 출력값을 내놓을 확률은 극히 낮다고 한다. 

4. 그래서 패스워드에 암호화에 자주 사용되었음

<br>
<br>

2번 특징 때문에 원문을 찾으려 드는건 바부짓인 것 같다. 

그래서 어케든 뚫어보려면... collision(충돌), 즉 같은 MD5값을 갖는 문자열을 알아내는 것이 방법이라고 함. 

어쨌든 MD5값이 같으면 같은 문자열이라고 판단하기 때문.

글구 시간이 지나며 이 collision을 발생시키는 기법들이 개발되었고, 때문에 현재는 보안 용도로 권장되지는 않는다구 한당. 굳이 쓰려면 꼭 salt를 붙여서 쓰는 게 안전하다구 하는디 salt는 또 뭐람. 

<br >

> salt: 소금이 기본 양념이듯 원문에 가미하여 암호문을 다른 값으로 만드는 것. hash에서 자주 쓰임. 이를 더 안전하게 만들기 위해 출력값과 다른 공간에 비밀스럽게 저장하면 후추(pepper)라고 한다... 라고 나무위키가 그랬음.
>
> 입력된 값에 임의의 값을 붙인다든지, 암호화 한 값에 또다른 값을 붙여서 암호화 한다든지...


<br>
<br>






***

## 이제 풀어보쟈

<br>
<br>

일단 powershell에서 

    ssh col@pwnable.kr -p2222 

을 입력하여 접속하자. (pw:guest)

<br>
<br>

이번에도 flag는 권한이 딸리니, 제시된 col과 col.c를 이용해 flag를 알아내야 한다.

<br>
<br>

col.c의 내용은 다음과 같다.


![col.c](/img/01_col.c.jpg)


<br>
<br>


main함수를 보면, 우선 인자의 조건을 제한하는 부분이 있다. 

20byte 이상의 첫번째 인자를 요구하는 것을 알 수 있다. 

<br>

눈여겨봐야 할 부분은 이어지는 여기. 


    if(hashcode == check_password( argv[1] )){
                system("/bin/cat flag");
                return 0;
        }


원하는 flag를 출력하기 위해서는 unsigned long type인 `hash` 변수가, 우리가 첫번째 인자로 줄 argv[1]을 파라미터로 받은 `check_password()`함수의 리턴값과 같아야 한다. 

그럼 이 함수가 뭐하는 놈인지 위로 올라가보자. 

<br>
<br>

이게무ㅓ고..... 

    unsigned long check_password(const char* p){
        int* ip = (int*)p;
        int i;
        int res=0;
        for(i=0; i<5; i++){
                res += ip[i];
        }
        return res;
}

return type은 `hash`와 같은 unsigned long. 파라미터로 문자열을 받는다. 

그리고 내부에서 지역 변수로 정의된 `ip`라는 int 포인터 타입(아마 array가 아닐까)의 변수는 파라미터로 받은 친구의 int* 형변환 버전으로 초기화된다. 

그리고 for문이 돌며 `res`에는 `ip`의 맨 앞 다섯 글자가 담겨지고... 

그 `res`가 return되는 것을 볼 수 있다. 



<br>
<br>


그러므로! 이 함수에서 return된 값이 `hashcode`, 즉 `0x21DD09EC`와 같도록 하는 것이 목적이 되겠다.

참고로 `0x21DD09EC`의 decimal 버전은 `568134124`다. 

즉, 함수의 리턴값이 일치하게 하려면은 `res`가 `568134124`가 되어야 하는거고. `res`는 `ip`의 맨 앞 다섯개 원소의 합... 

그러니께 더해서 `568134124`가 되는 다섯개의 정수, 그놈이 이 hashcode의 collision이라 추측해볼 수 있겠다. 

그럼 그걸 어케 만드느냐?? 


<br>
<br>


함수를 다시 살펴보면, 파라미터로 들어가는... 우리가 첫번째 인자로 넣어준 값을 어케어케 해서 리턴값을 만들어낸다. 

그 어케어케를 거꾸로 가서 원하는 값이 나오도록 하는 input값을 찾아보자. 



1. char배열인 input(p)을 int배열로 형변환시킨다. 
2. 앞에서부터 다섯개 원소를 res에 누적합산한다. 

근데 보자보자. char는 하나에 1byte다. int는 4byte다. 그니까 네개의 문자가 하나의 int로 변환되는 것 아닐까? 

결국 다섯개의 int를 얻기 위해서는 20개의 char을 입력해줘야 하는 것.

입력 조건인 20byte와 맞아떨어지는 걸 보니 이 가설의 신뢰도가 올라간다.

<br>
<br>

그럼 네개씩 모아 하나의 int가 되는 그 친구들의 합이 568134124가 되게 하기 위해... 일단 한 int당 뭐가 되게 하면 좋을지 직관적으로 나누어보자. 

    >>> 568134124//5
    113626824
    >>> 568134124%5
    4
파이썬이 계산해줌 히히. 

보면 개당 113626824의 값을 갖고, 한놈은 4를 더 가지면 될 듯. 

    (int)(p[0]~p[3]) = 113626824
    (int)(p[4]~p[7]) = 113626824
    (int)(p[8]~p[11]) = 113626824
    (int)(p[12]~p[15]) = 113626824
    (int)(p[16]~p[19]) = 113626828

일케 나오게끔... 


저걸 되게 하기 위해서는 char 4개가 int가 되는 규칙성을 알아야 한다. 

<br>
<br>

그리고 시험삼아 짜본 코드에서 실마리를 얻을 수 있었다!

![testCode](/img/01_testCode.jpg)

보면 가설대로 char입력값 네개당 하나의 int로 변환됨을 볼 수 있다. 

그리고 변환된 인트는 예시코드상 다음과 같은데...

    aaaa = 1633771873
    bbbb = 1650614882
    cccc = 1667457891
    dddd = 1684300900
    eeee = 1701143909

이건 뭔 규칙성인가 이리저리 변환기를 돌려보니, 

char의 ascii값인 정수형을, decimal to hex변환하고,
그 hex값 네 개를 단순 나열하여 다시 decimal로 변환하면 이 값이 나온다!

예를 들자면, 

- aaaa를 ascii 즉 decimal로 바꾸면 61616161
- 61616161을 hex로 취급한 채 decimal로 다시 변환하면 1633771873가 나온다!! 

역시 내부적으로 16진수를 쓴다는 점이 열쇠였던 듯. 일단 16진수 변환해서 차곡차곡 밀어넣은 다음 네개를 하나로 취급해서 다시 변환하는 방식이다. 


<br>
<br>


이제 규칙성을 알았으니 역으로 풀어볼 차례다. 아까 그놈을 데려와보자. 


    (int)(p[0]~p[3]) = 113626824
    (int)(p[4]~p[7]) = 113626824
    (int)(p[8]~p[11]) = 113626824
    (int)(p[12]~p[15]) = 113626824
    (int)(p[16]~p[19]) = 113626828


근데... 얘를 앞의 규칙성에 완전히 역으로 해보려 했는데, 막 문제가 많더라. 

![sapzil](/img/01_sapzil.jpg)

삽질의 흔적들... 


걍 인자를 16진수 형태로 넣어주는게 젤 깔끔할 것 같다. 전에 뭐냐 little endian형식으로 넣어주면 된다고 배운 것 같은데... 

    //일단 hex로
    0x06C5CEC8
    0x06C5CEC8
    0x06C5CEC8
    0x06C5CEC8
    0x06C5CECC
    
    //little endian으로 
    \xC8\xCE\xC5\x06
    \xC8\xCE\xC5\x06
    \xC8\xCE\xC5\x06
    \xC8\xCE\xC5\x06
    \xCC\xCE\xC5\x06


됐당.... 이래도 되나 근데? 

\xC8\xCE\xC5\x06\xC8\xCE\xC5\x06\xC8\xCE\xC5\x06\xC8\xCE\xC5\x06\xCC\xCE\xC5\x06

안되네 젠장 

<br>
<br>

그럼 가보자고

<br>
<br>

<hr/>

<br>
<br>





<br>
<br>





<br>
<br>






<br>
<br>
<br>




