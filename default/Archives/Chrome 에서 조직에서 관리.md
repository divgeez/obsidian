
https://bbs.ruliweb.com/pc/board/300006/read/2224486

**[chrome://policy](chrome://policy/)** 들어가시면, (엣지는 chrome 대신 edge)

![79c7d2e3a64888.png](https://i3.ruliweb.com/img/21/01/16/1770a93354750f55a.png)

**SSLVersionMin**이란 것을 볼 수 있습니다.

이건 은행 같은 곳의 EXE 보안프로그램이 설치하는 레지스트리 같은 겁니다.

**저게 있으면 크롬에서 '조직에서 관리' 부분이 계속 나타납니다.**

이건 구라 제거기로도 없어지지 않습니다.

레지스트리 편집기(regedit)를 열고,

**HKEY_LOCAL_MACHINE\SOFTWARE\Policies → \Chrome** (엣지는 \Edge)

**만약 'SSLVersionMin' 값이 있을때 이걸 삭제**하면 사라집니다.

하지만 구글 계정 자체가 G Suite 계정이면 없어지진 않을 겁니다.