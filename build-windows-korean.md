비트코인 Windows 빌드 설명서
====================

	번역: cryptotetsuya (Twitter@akapola)
	편집자 주석은 [주: 내용] 참고

Windows용 Bitcoin Core를 빌드하는 방법에 대한 참고사항.

Windows용 Bitcoin Core를 빌드할때 제대로 동작한다 알려진 옵션은 아래와 같아:

* Linux에서 빌드할경우 [Mingw-w64](https://mingw-w64.org/doku.php) 크로스 컴파일러 툴체인을 설치. Windows에서 동작하는 Bitcoin Core바이너리를 빌드하기 위해선 Ubuntu Trusty 14.04 사용을 권장한다.
* Windows에서 [Windows Subsystem for Linux (WSL)](https://msdn.microsoft.com/commandline/wsl/about)와 Mingw-w64 크로스 컴파일러 툴체인을 설치.

아래의 다른 옵션들도 아마 동작하겠지만 실험된 샘플이 없는관계로 해본사람은 기여를 부탁한다:

* Windows용 POSIX 호환성계층 앱 [cygwin](http://www.cygwin.com/)이나 [msys2](http://www.msys2.org/)을 이용.
* Windows용 네이트브 컴파일러 툴체인 [Visual Studio](https://www.visualstudio.com) 같은거.

Windows Subsystem for Linux 설치
---------------------------------------

Microsoft가 Windows 10 에서 [Windows Subsystem for Linux (WSL)](https://msdn.microsoft.com/commandline/wsl/about)라는걸 출시했어. 
이것은 Ubuntu 환경을 베이스로 bash셸을 실행가능하기에 여기서 크로스 컴파일러를 이용하면 다른 리눅스 환경을 따로 구성할 필요는 없을거야.
물론 OpenSUSE와 같은 몇가지 다른 리눅스 시스템을 설치할수도 있지만 Ubuntu에서만 테스트되었으니 그걸 기술할거야.

이 기능은 Windows 10 64비트에서만 동작하며 자세한건 (https://msdn.microsoft.com/en-us/commandline/wsl/install_guide) 참고.

WSL 설치과정에 대한 자세한 설명은 위의 링크에도 설명되있어.
먼저 Windows 10에 WSL을 설치하려면 Fall Creators Update가 설치되있어야 한다.(cmd 열었을때 보이는 버전이 10.0.16215.0 이상):

1. WSL기능을 활성화 하려면
  * Win+R키로 실행창을 열어서 (`OptionalFeatures.exe`)를 입력하여 실행시켜.
  * Linux용 Windows 하위시스템을 체크하고 확인을 눌러.
  * 재부팅 하라고 나오면 해주면 끝나.
2. Ubuntu를 설치하려면
  * cmd에서 lxrun /install을 입력하고 y로 넘겨줘
  * 자동으로 스토어에서 Ubuntu 이미지를 받아 설치할거야 [주: 지울때는 lxrun /uninstall /full로 지우면 된다.]
3. 설치완료하기
  * 로컬설정을 en-us로 놔둘꺼냐 물어보는데 오래걸리니 n으로 넘겨버리자.
  * 새로운 UNIX 사용자 계정을 생성하자. (이건 윈도우 계정과는 별개로 돌아간다.)

cmd 창에서 bash를 입력해서 셸을 활성화 하였다면, 아래의 "크로스 컴파일" 섹션의 내용을 따라가면 될거야.
64비트 버전으로 컴파일이 권장되는데 32비트버전의 컴파일도 가능하긴해.

Ubuntu and Windows Subsystem for Linux 크로스 컴파일
------------------------------------------------------------

지금 WSL에 설치된 Ubuntu의 버전은 Ubuntu Xenial 16.04 일거야. Ubuntu Xenial 버전에서의 Mingw-w64 패키지는 Bitcoin Core에서 몇가지 실행파일은 빌드가 안되지만
Ubuntu Xenial 일지라도 Ubuntu Zesty로부터 크로스 컴파일러 패키지를 설치하면 제대로 빌드할수 있을거야, 아래 과정을 참고하자.
확실한건 Ubuntu Zesty 17.04 에서 17.10까지의 버전은 작동이 보장되어있어.

아래 과정은 그냥 Ubuntu (VM포함) 혹은 WSL에서 공통적으로 수행가능해. 물론 다른 리눅스 배포판에서도 사용가능하지만 툴체인을 설치하기위한 명령어는 조금 다를거야.

먼저 일반적인 필수 의존성 패키지를 설치해보자:

    sudo apt update
    sudo apt upgrade
    sudo apt install build-essential libtool autotools-dev automake pkg-config bsdmainutils curl git

여기서 (`build-essential`) 이라는 패키지는 의존성 패키지를 위해 꼭 필요해 예를들면 (`protobuf`)는 호스트 유틸리티를 빌드해야되는데 빌드과정에서 필요하게 된다.

나머진 이곳을 참고: [dependencies.md](build-unix-한국어번역.md#의존성패키지-dependencies).

## 64비트 윈도우용으로 빌드하기

먼저 mingw-w64 크로스 컴파일러 툴체인을 설치해. Ubuntu 배포판마다 설치방법이 조금 틀린관계로 (특히 Xenial 문제에 대해서는) 따로 설명을 붙일거야.

정석대로 mingw32 크로스 컴파일러 툴체인을 설치하는 방법이라면:

    sudo apt install g++-mingw-w64-x86-64 #[주: Xenial(WSL)에서 테스트결과 이걸 설치하고 아래과정을 수행하면 의존성 충돌이 없어진다.]

Ubuntu Trusty 14.04에서는:

    따로 해줄게 없어.

Ubuntu Xenial 16.04와 Windows Subsystem for Linux(WSL) <sup>[1](#footnote1),[2](#footnote2)</sup>에서는:

    sudo apt install software-properties-common
    sudo add-apt-repository "deb http://old-releases.ubuntu.com/ubuntu zesty universe" #[주: archive.ubuntu.com에는 zesty패키지가 내려간상태라 임의로 변경.]
    sudo apt update
    sudo apt upgrade
    sudo update-alternatives --config x86_64-w64-mingw32-g++ # mingw32 g++ compiler의 기본옵션을 posix로 설정하자.

Ubuntu Zesty 17.04 <sup>[2](#footnote2)</sup>:

    sudo update-alternatives --config x86_64-w64-mingw32-g++ # mingw32 g++ compiler의 기본옵션을 posix로 설정하자.

툴체인들이 제대로 설치되었다면 정상적인 빌드과정을 따라가자:

WSL안에 Bitcoin Core 소스의 경로는 **반드시** 기본으로 마운트된 파일시스템 안에 위치해 있어야해, 예를들면 /usr/src/bitcoin
그리고 /mnt/d/ 안의 경로에 위치해선 안돼. 위 사항을 지키지 않을경우 autoconf 스크립트가 에러를 뿜을거야.
다시말하자면 윈도우 파일시스템 내부의 경로를 빌드를 위해 직접 사용하는것은 불가능하단 이야기였어.

평범하게 소스를 가져와보자:

    git clone https://github.com/bitcoin/bitcoin.git

소스코드의 빌드준비가 완료되었다면 아래의 과정을 따라보자.

    PATH=$(echo "$PATH" | sed -e 's/:\/mnt.*//g') # 문제될수있는 윈도우의 %PATH% 환경변수를 제거하는 과정이야.
    cd depends
    make HOST=x86_64-w64-mingw32
    cd ..
    ./autogen.sh # tarball가져온걸 빌드할땐 필요없는 과정이야.
    CONFIG_SITE=$PWD/depends/x86_64-w64-mingw32/share/config.site ./configure --prefix=/
    make
    [주: make -j 스레드수 옵션을 붙여야 멀티스레드로 컴파일이 가능하다.]

## 32비트 윈도우용으로 빌드하기

32비트 윈도우용 실행파일을 빌드하려면 다음의 의존성 패키지를 설치해야해:

    sudo apt install g++-mingw-w64-i686 mingw-w64-i686-dev

Ubuntu Xenial 16.04와 Ubuntu Zesty 17.04, Windows Subsystem for Linux의 경우 <sup>[2](#footnote2)</sup>:

    sudo update-alternatives --config i686-w64-mingw32-g++  # mingw32 g++ compiler의 기본옵션을 posix로 설정하자.

WSL안에 Bitcoin Core 소스의 경로는 /usr/src/bitcoin과 같이 **반드시** 기본으로 마운트된 파일시스템 안에 위치해 있어야해.
절대로 /mnt/d/ 안의 경로에 위치해선 안돼. 위 사항을 지키지 않을경우 autoconf 스크립트가 에러를 뿜을거야.
다시말하자면 윈도우 파일시스템 내부의 경로를 빌드를 위해 직접 사용하는것은 불가능하단 이야기였어.

평범하게 소스를 가져와보자:

    git clone https://github.com/bitcoin/bitcoin.git

이제 빌드하자면:

    PATH=$(echo "$PATH" | sed -e 's/:\/mnt.*//g') # 문제될수있는 윈도우의 %PATH% 환경변수를 제거하는 과정이야.
    cd depends
    make HOST=i686-w64-mingw32
    cd ..
    ./autogen.sh # tarball가져온걸 빌드할땐 필요없는 과정이야.
    CONFIG_SITE=$PWD/depends/i686-w64-mingw32/share/config.site ./configure --prefix=/
    make
    [주: make -j 스레드수 옵션을 붙여야 멀티스레드로 컴파일이 가능하다.]

## 다른시스템의 경우

다른 시스템의 설명을 보고싶다면 [README.md](https://github.com/bitcoin/bitcoin/blob/master/depends/README.md)를 참고해.

설치
-------------

Windows subsystem을 이용한 빌드가 끝났다면 컴파일 된 실행 파일을 Windows파일시스템 경로안에 배포판의 `.zip` 아카이브와 동일한 디렉토리구조로
복사하는 것이 유용할 수 있어. make할때 요 과정만 추가하면되는데 아래예시는 `c:\workspace\bitcoin` 에 넣는 과정이야:

    make install DESTDIR=/mnt/c/workspace/bitcoin

각주
---------

<a name="footnote1">1</a>: WSL/Ubuntu Xenial 16.04에 포함된 64비트 Mingw-w64 패키지의 크로스컴파일러는 비트코인과 관련된 두가지의 실행파일에서 크래쉬가 발생하는 버그가 있어. 이 버그는 -fstack-protector-all 이라는 g++ 컴파일러 플래그와 관련있는데, 버퍼오버플로우 공격을 완화하는 역할을 하는 플래그야.
Ubuntu 17 배포판의 Mingw-w64패키지를 설치하면 요문제는 해결되는데, 아무튼 이건 공식적으로 지원되는 해결방법이 아니니깐 뭔가 고장났을경우 WSL/Ubuntu를 재설치할 각오가 있는경우에만 사용하길 바랄게.

<a name="footnote2">2</a>: Ubuntu Xenial 16.04에서 32비트와 64비트 Mingw-w64 패키지의 크로스컴파일러 옵션에서 두가지의 옵션을 확인할수 있을거야. 바로 posix와 win32 스레드인데, 기본옵션인 win32 스레드를 쓰는것이 Windows의 kernel32.lib와 직접링크하는 기계어를 생성하므로 더 효율적인 방법이야.
하지만 win32 스레드를 지원하기 위한 헤더는 C++11 표준 라이브러리의 몇몇 클래스들, 특히 std::mutex같은 것과 충돌해서 빌드가 제대로 되지않아.
따라서 bincoin core의 헤더 소스코드를 고치지않는한 Mingw-w64 크로스 컴파일러의 win32 버전을 사용할수는 없어. [주: 에러내용은 (https://pastebin.com/9Fuhq61J) 참고]
