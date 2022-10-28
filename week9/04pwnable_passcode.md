# HW04) pwnable passcode


## week6 

<hr/>

[pwnable](https://pwnable.kr/play.php)

<br>
<br>

passcode. 

![cardIMG](https://pwnable.kr/img/passcode.png)


<br>
<br>

카드를 쨘 클릭하면 이렇게 안내가 뜬다. 

    Mommy told me to make a passcode based login system.
    My initial C code was compiled without any error!
    Well, there was some compiler warning, but who cares about that?

플래그같이 말하는 것 봐라 이거... 딱봐도 취약점과 관련된 힌트이다. 


> ssh passcode@pwnable.kr -p2222 (pw:guest)




<br>

***


## 드가보자!!


<br>
<br>


c코드부터 보면 다음과 같다.

![passcode.c](/img/04_passcode.c.jpg)

길다 길어... 

우리의 목적은 권한이 부족해 cat으로는 접근이 불가한 flag를 읽어내는 것. 


<br>
<br>


흐름부터 보자. main함수는 안내 문구를 출력하고, `welcome()`함수와 `login()`함수를 차례로 호출한다.

welcome에서는 무슨 일이 일어나냐?

    void welcome(){
        char name[100];
        printf("enter you name : ");
        scanf("%100s", name);
        printf("Welcome %s!\n", name);
    }

우선 100칸짜리 char 배열에 name을 입력받고 있다. 누가봐도 여기에 뭔가 해줘야 할 것 같다. 일단 킵. 

<br>

다음은 login으로 들어가보자. 

    void login(){
        int passcode1;
        int passcode2;

        printf("enter passcode1 : ");
        scanf("%d", passcode1);
        fflush(stdin);

        // ha! mommy told me that 32bit is vulnerable to bruteforcing :)
        printf("enter passcode2 : ");
        scanf("%d", passcode2);

        printf("checking...\n");
        if(passcode1==338150 && passcode2==13371337){
                printf("Login OK!\n");
                system("/bin/cat flag");
        }
        else{
                printf("Login Failed!\n");
                exit(0);
        }
    }

int형태의 변수 `passcode1`과 `passcode2`가 선언되며 시작한다. 그리고 int형으로 passcode1과 2를 차례로 입력받는다.

여기서 특이한 것은, scanf()함수를 쓰면서 인자 부분에 passcode들의 주소(&첨가)가 아닌... 걍 생짜 거시기를 넣어버렸다. 이래도 되는거임? 

인자의 자리는... 입력값이 저장될 곳의 주소자나? 근데 거기 &처리를 안해줬다는건 passcode들의 값을 주소삼아 거기에 입력값을 저장하겠다는 소리가 된다. 

그말인즉슨 passcode의 값을 우리가 원하는 곳의 주소로 조작하면 scanf함수를 통해 그 주소에 입력값을 넣을 수 있다는 뜻이다... 


더하여 주석은 `32bit is vulnerable to bruteforcing` 이라며, 브루트포싱... 무차별대입?떡밥을 던진다.  

<hr>

그으리고 그 사이에 call되는 `fflush()`. 얘 첨보는놈이라 구글링해봤다. 

정의는 다음과 같고, 

    int fflush(FILE *stream)


사용은 "C 라이브러리 함수로서 출력 버퍼를 비운다." 라고 한다.즉 출력 스트림 버퍼에 남아 있는 데이터를 꺼내어 모두 출력해 버리는 함수이며 return 값은 작업에 성공하면 0, 버퍼에 아무것도 없는 상태라면 EOF, 실패하면 그 실패에 대한 errno 값이다. 

여기서는 stdin을 받았으니 입력스트림을 받은 모양새. 입력버퍼를 비우기 위해 받은 것 같은데... 사실 이게 굉장히 위험한 짓이라고 한다. 

fflush는 엄연히 출력버퍼를 비우는 친구이다. 찾아본 [레퍼런스](https://8ublictip.tistory.com/6)에 의하면 "fflush() 함수의 매개변수로 입력 스트림을 전달하는 것은 정의되지 않은 방법입니다. 즉, 어떻게 작동할지 정의돼 있지 않기 때문에 그 누구도 결과를 예측할 수 없고, 경우에 따라 치명적인 결과를 초래할 수도 있습니다."라고 한당. 아마 pwnable꼬맹이에게 떴다던 경고문구가 이 관련된게 아닐까 싶음. 그렇다는건 우리가 파고들어야 할 취약점도 이와 관련되었다는 뜻이겠지? 

또다른 [레퍼런스](http://andyader.blogspot.com/2013/09/fflush.html)에 의하면 fflush에 인자로 들어가는 stream의 값은 출력 스트림에 한정되고, 입력 스트림이 들어갈 경우는 C 표준에 정의되어 있지 않기 때문에 컴파일러마다, OS 마다 동작하는 양상이 전혀 다르다고 한다. 

그리고 Linux에서 fflush에 stdin이 들어와버리면... 추후에 합당한 동작이 정의될 때까지 stdin에 대한 fflush 기능은 **일단 정의하지 않는다**는 의미로? 해석하고있다. 뭔가 파고들 여지가 많아보이는 부분 발견이다. 

<br><hr><br>

이어서 내려가보면 나오는 if문. 

    printf("checking...\n");
    if(passcode1==338150 && passcode2==13371337){
            printf("Login OK!\n");
            system("/bin/cat flag");
    }
    else{
            printf("Login Failed!\n");
            exit(0);
    }

passcode1이 338150이고 passcode2가 13371337일 때 쉘코드를 실행시킨다. 

그냥 이 두 값들을 scanf를 통해 각각 위치에 쇽 담아주면 참으로 스무스할텐데, 앞서 살펴봤듯 이 친구가 코드를 아주 난해하게 짜놨다. 

scanf에는 &사인을 뺴놓지를 않나 정의되지도 않은 fflush(stdin)을 쓰지를 않나. 

하지만 생각해보자. 어케해야할까. 어케해야 두 공간에 저 값들을 집어넣고 무사히 다음 스텝으로 실행되게 하지? 




<br>
<br>



푸는중...

<br>
<br>



<br>
<br>





<br>
<br>





<br>
<br>





<br>
<br>





<br>





<br>
<hr>
<br>



<br>




<br>
<br>

<hr/>

<br>
<br>



![success](/)

<br>

    blah blah



<br>
<br>
<br>



:)




