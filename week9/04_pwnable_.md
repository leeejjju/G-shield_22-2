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

우선 100칸짜리 char 배열에 name을 입력받고 있다. 딱봐도 여기에 뭔가 해줘야 할 것 같다. 일단 킵. 


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






<br>
<br>

푸는중!!



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




