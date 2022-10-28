# HW05) pwnable random


## week9 

<hr/>

[pwnable](https://pwnable.kr/play.php)

<br>
<br>

random! 


![cardIMG](https://pwnable.kr/img/random.png)


설명은 다음과 같다. 

    Daddy, teach me how to use random value in programming!

> ssh random@pwnable.kr -p2222 (pw:guest)


<br>

***


## 슝슝 들어가보자~


<br>
<br>

일단 C코드.

![random.c](/img/05_random.c.jpg)





과제 하느라 random을 사용해봤을 때 유의할 점이, time 관련 함수를 사용해 `seed = time(NULL);` 요런식으로? 시드를 리셋해주지 않으면 한번 정해진 랜덤값은 바뀌지 않는 것 같았다. 이걸 기억한다면 random이라고 해서 겁낼 필요는 없을 것 같다. 코드에 시드 초기화하는 부분은 안 나오니까 한번만 알아내면 된다는 소리임. 


이걸 기억한 채 코드를 찬찬히 보자. 코드 자체는 간단하다. 

    if( (key ^ random) == 0xdeadbeef ){
        printf("Good!\n");
        system("/bin/cat flag");
        return 0;
    }

`key`를 정수형으로 입력받고, 그 값과 랜덤하게 생성된 `random`을 xor연산한 뒤 그 결과와 `0xdeadbeef`를 비교. 같다면 if문 내부의 쉘코드가 실행된다. 

구럼 key에 뭘 입력해주면 좋을지를 알아내고 싶어진다.

어떻게? 

xor연산의 특성을 알면 문제는 간단해진다. 

    A xor B = C일 때,
    B xor C = A.
    A xor C = B.

그렇다구 한다. 이말인즉슨 `key ^ random == 0xdeadbeef` 얘를 조금 비틀어서...

    key ^ random == 0xdeadbeef
    0xdeadbeef ^ random == key

이런 결과를 기대해볼 수 있게 만드는 것이다. 

이제 우리에게 필요한 건 random에 뭐가 들어있는지 뿐이다. 

<br>

돌격.

<br>


![disas main](/img/05_disasMain.jpg)





random값이 담겨진 주소를 찾아내기 위해, `random = rand();`연산이 일어나는 순간을 찾아보자. 

    0x0000000000400601 <+13>:    call   0x400500 <rand@plt>
    0x0000000000400606 <+18>:    mov    DWORD PTR [rbp-0x4],eax

여기가 아닐까 살포시 예상해본다. 

선언순서상 제일 먼저 선언된 random의 위치는 base pointer, rbp에서 따악 한 칸 앞일테니까 rbp-0x4에 들어가고있다는 점이 아다리가 맞는다. 

즉, 거기 들어가고있는 값의 주소... eax레지스터에 담긴게 random이겠다. 그럼 eax의 값을 내놓으라 해보자. 

    (gdb) i r eax
    eax            0x6b8b4567       1804289383

주소는 0x400760, 값은 4196192!

참고로 eax 레지스터는 외부함수를 call할 때 리턴값이 담기는 곳이라고 한당. 그니까 `rand()`를 call했을 때 임시로 그 반환값을 가지고 있음을 유추해볼 수 있고, 이것도 하나의 구하는 방법이 되겠다. 




하튼 그렇게 구한 eax를 이용해 xor연산까지 깔끔하게 해보면... 


    (gdb) p/d 0x6b8b4567^0xdeadbeef
    $1 = 3039230856




`3039230856`. 이놈이 그놈이 아닐까 싶어진다. 

<br><hr><br>
두구두구.

<br>

![good](/img/05_success!!.jpg)


<br>

    Mommy, I thought libc random is unpredictable...




<br>
<br>
<br>



:)






 

