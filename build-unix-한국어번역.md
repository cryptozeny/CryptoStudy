비트코인 UNIX 빌드 설명서
====================

	번역: cryptozeny (Twitter@9werty60y)
	편집자 주석은 [주: 내용] 참고

Bitcoin Core를 Unix에서 빌드하는 방법에 대한 참고사항.

(OpenBSD 에서의 방법은 [build-openbsd.md](build-openbsd.md) 및 [build-netbsd.md](build-netbsd.md) 참고)

주의
---------------------
반드시 절대경로(absolute paths)를 사용하여 비트코인과 의존성패키지를 구성(configure) 및 컴파일한다, 예를들면, 의존성패키지의 경로를 지정할때 :

	../dist/configure --enable-cxx --disable-shared --with-pic --prefix=$BDB_PREFIX

여기서 BDB_PREFIX 는 반드시 절대경로 - 절대경로의 사용을 보장하는 $(pwd)를 사용하여 정의한다.

빌드하기 (BUILD)
---------------------

```bash
./autogen.sh
./configure
make
make install # 선택사항
```

의존성패키지의 요구사항이 충족된다면 자동으로 bitcoin-qt도 빌드된다.

의존성패키지 (Dependencies)
---------------------

다음과 같은 의존성패키지들이 필요 :

 라이브러리명 | 목적              | 설명
 ------------|------------------|----------------------
 libssl      | 암호화            | 난수발생, 타원곡선 암호화
 libboost    | 유틸리티          | 스레딩 및 데이터 구조를 위한 라이브러리
 libevent    | 네트워킹          | OS 독립적 비동기 네트워킹

선택적 의존성패키지 (Optional dependencies) [주: 설치하는것이 좋다] :

 라이브러리명   | 목적              | 설명
 ------------|------------------|----------------------
 miniupnpc   | UPnP 지원         | 방화벽-점핑 지원
 libdb4.8    | Berkeley DB      | 지갑 저장공간 (지갑 활성화시)
 qt          | GUI              | GUI toolkit (GUI 활성화시)
 protobuf    | GUI에서 지불       | 지불 프로토콜에 사용되는 데이터 교환형식 (GUI 활성화시)
 libqrencode | GUI에서 QR코드     | 선택사항, QR코드생성 (GUI 활성화시)
 univalue    | 유틸리티           | JSON 구문분석 및 인코딩 (만약 configure에서 --with-system-univalue 를 사용하지 않으면 기존의 번들버젼을 사용한다)
 libzmq3     | ZMQ 알림          | 선택사항, ZMQ 알림생성 가능 (ZMQ 버젼 >= 4.x)

사용된 의존성패키지 목록은 다음을 참고 [dependencies.md](dependencies.md)

메모리 요구사항
--------------------
C++ 컴파일러는 메모리를 많이 사용한다. 비트코인 코어를 컴파일시 적어도 1.5 GB 이상의 메모리를 확보할것. 메모리가 부족한 시스템에서는 gcc에서 다음의 추가적인 CXXFLAGS를 사용한다 :

    ./configure CXXFLAGS="--param ggc-min-expand=1 --param ggc-min-heapsize=32768"

<!--Dependency Build Instructions: Ubuntu & Debian-->
의존성패키지 빌드 설명: 우분투 & 데비안
----------------------------------------------
<!--Build requirements:-->
빌드 요구사항 :

    sudo apt-get install build-essential libtool autotools-dev automake pkg-config libssl-dev libevent-dev bsdmainutils python3

<!--Options when installing required Boost library files:-->
Boost 라이브러리 파일을 설치할때 필요한 옵션 :

<!--1. On at least Ubuntu 14.04+ and Debian 7+ there are generic names for the
individual boost development packages, so the following can be used to only
install necessary parts of boost:-->
1. 우분투 14.04+ 및 데비안 7+ 에서는 개별 개발패키지에 대한 일반적인 이름이 있으므로 다음을 사용하여 필요한 부분만 설치하면 됩니다 :

        sudo apt-get install libboost-system-dev libboost-filesystem-dev libboost-chrono-dev libboost-program-options-dev libboost-test-dev libboost-thread-dev

<!--2. If that doesn't work, you can install all boost development packages with:-->
2. 만약 전자가 실패하면 다음을 사용하여 모든 Boost 개발패키지를 설치할수 있습니다. [주 : 후자를 사용하여 전부 설치하는것을 추천한다]

        sudo apt-get install libboost-all-dev

<!--BerkeleyDB is required for the wallet.-->
버클리DB는 지갑에 필요합니다. [주 : 이것이 없으면 지갑이 작동하지 않는다]

<!--**For Ubuntu only:** db4.8 packages are available [here](https://launchpad.net/~bitcoin/+archive/bitcoin).
You can add the repository and install using the following commands:-->
**우분투에만 해당하는 내용 :** 버클리DB(db4.8) 패키지는 [여기](https://launchpad.net/~bitcoin/+archive/bitcoin). 다음과 같이 저장소를 추가하여 설치할수 있다.

    sudo apt-get install software-properties-common
    sudo add-apt-repository ppa:bitcoin/bitcoin
    sudo apt-get update
    sudo apt-get install libdb4.8-dev libdb4.8++-dev

<!--Ubuntu and Debian have their own libdb-dev and libdb++-dev packages, but these will install
BerkeleyDB 5.1 or later, which break binary wallet compatibility with the distributed executables which
are based on BerkeleyDB 4.8. If you do not care about wallet compatibility,
pass `--with-incompatible-bdb` to configure.-->
우분투 및 데비안은 기본적으로 libdb-dev 및 libdb++-dev 패키지가 있는데, 그 이유로 버클리DB 5.1 혹은 그 이상을 설치할것이다. 그런데 이 최신버젼이 분산 실행 바이너리 호환성을 손상시키기에 우리는 버클리DB 4.8 버젼(구버젼)을 설치할것이다. 만약 지갑 호환성에 신경쓰지 않는다면 `--with-incompatible-bdb` 를 사용해서 구성(configure)할것. [주 : BerkeleyDB 최신버젼에 문제가 있으니 예전 구버젼을 쓰겠다는 것이다]

<!--See the section "Disable-wallet mode" to build Bitcoin Core without wallet.-->
지갑기능을 사용하지 않고 비트코인 코어를 빌드하려면 "지갑-사용중지 모드" 참고.
<!--Optional (see --with-miniupnpc and --enable-upnp-default):-->
선택사항 (--with-miniupnpc 및 --enable-upnp-default 참고) :
    sudo apt-get install libminiupnpc-dev

<!--ZMQ dependencies (provides ZMQ API 4.x):-->
ZMQ 의존성패키지 (ZMQ API 4.x 제공) :
    sudo apt-get install libzmq3-dev

<!--Dependencies for the GUI: Ubuntu & Debian-->
GUI를 위한 의존성패키지 : 우분투 및 데비안
-----------------------------------------

<!--If you want to build Bitcoin-Qt, make sure that the required packages for Qt development
are installed. Either Qt 5 or Qt 4 are necessary to build the GUI.
If both Qt 4 and Qt 5 are installed, Qt 5 will be used. Pass `--with-gui=qt4` to configure to choose Qt4.
To build without GUI pass `--without-gui`.-->
Bitcoin-Qt를 빌드하려면 Qt 개발에 필요한 패키지가 설치되어 있는지 확인할것. GUI를 빌드하려면 Qt 5 또는 Qt 4가 필요하다. 만약 Qt 4 및 Qt 5 모두 설치되면 Qt 5가 사용된다. `--with-gui=qt4` 을 사용하면 Qt4를 선택할수 있다. GUI 없이 빌드하려면 `--without-gui` 를 사용한다.

<!--To build with Qt 5 (recommended) you need the following:-->
Qt 5 (권장사항)으로 빌드하려면 다음이 필요하다 [주 : Qt 5를 쓰는것이 좋다] :

    sudo apt-get install libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools libprotobuf-dev protobuf-compiler

<!--Alternatively, to build with Qt 4 you need the following:-->
만약, Qt 4로 빌드하려면 다음이 필요하다 :

    sudo apt-get install libqt4-dev libprotobuf-dev protobuf-compiler

<!--libqrencode (optional) can be installed with:-->
libqrencode (선택사항) 다음으로 설치가능 :

    sudo apt-get install libqrencode-dev

<!--Once these are installed, they will be found by configure and a bitcoin-qt executable will be
built by default.-->
이것들을 모두 설치하고 제대로 구성한다면 기본적으로 bitcoin-qt 실행파일이 빌드될것이다.

<!--Dependency Build Instructions: Fedora-->
의존성패키지 빌드 설명: 페도라
-------------------------------------
<!--Build requirements:-->
빌드 요구사항 :

    sudo dnf install gcc-c++ libtool make autoconf automake openssl-devel libevent-devel boost-devel libdb4-devel libdb4-cxx-devel python3

<!--Optional:-->
선택사항 :

    sudo dnf install miniupnpc-devel

<!--To build with Qt 5 (recommended) you need the following:-->
Qt 5 (권장사항)으로 빌드하려면 다음이 필요하다 [주 : Qt 5를 쓰는것이 좋다] :

    sudo dnf install qt5-qttools-devel qt5-qtbase-devel protobuf-devel

<!--libqrencode (optional) can be installed with:-->
libqrencode (선택사항) 다음으로 설치가능 :

    sudo dnf install qrencode-devel

<!-- Notes -->
참고
-----
<!-- The release is built with GCC and then "strip bitcoind" to strip the debug symbols, which reduces the executable size by about 90%. -->
이 릴리즈는 GCC로 빌드되고 "strip bitcoind"을 사용하여 디버그 심볼을 제거하였다. 그래서 실행파일의 크기를 약 90% 줄일수 있었다.

miniupnpc
---------

<!-- [miniupnpc](http://miniupnp.free.fr/) may be used for UPnP port mapping.  It can be downloaded from [here](http://miniupnp.tuxfamily.org/files/).  UPnP support is compiled in and turned off by default.  See the configure options for upnp behavior desired: -->
[miniupnpc](http://miniupnp.free.fr/)는 UPnP 포트 매핑에 사용할수 있다. [여기](
http://miniupnp.tuxfamily.org/files/)에서 다운로드 가능하다. UPnP 지원은 기본적으로 지원되고 해제가 가능하다. 원하는 UPnP 동작에 대한 구성을 확인할것.

	<!-- --without-miniupnpc      No UPnP support miniupnp not required -->
	<!-- --disable-upnp-default   (the default) UPnP support turned off by default at runtime -->
	<!-- --enable-upnp-default    UPnP support turned on by default at runtime -->
	--without-miniupnpc      요구되지 않는경우 UPnP 지원을 사용하지 않는다
	--disable-upnp-default   (기본값) 런타임시 UPnP 지원을 사용하지 않는다
	--enable-upnp-default    런타임시 기본적으로 UPnP 지원을 사용한다

<!-- Berkeley DB -->
버클리 데이터베이스
-----------
<!-- It is recommended to use Berkeley DB 4.8. If you have to build it yourself, you can use [the installation script included in contrib/](/contrib/install_db4.sh) like so -->
버클리 데이터베이스 4.8을 사용하는것을 추천한다. 스스로 구축하는 경우에는 [contrib에 포함된 설치 스크립트/](/contrib/install_db4.sh)를 다음과 같이 사용한다 :

```shell
./contrib/install_db4.sh `pwd`
```

<!-- from the root of the repository. -->
저장소의 루트에서.

<!-- **Note**: You only need Berkeley DB if the wallet is enabled (see the section *Disable-Wallet mode* below). -->
**참고**: 지갑이 활성화되어 있으면 버클리 데이터베이스만 필요하다 (하단의 *지갑-사용중지 모드* 참고)

부스트(Boost)
-----
<!-- If you need to build Boost yourself: -->
스스로 부스트를 구축하는 경우 :

	sudo su
	./bootstrap.sh
	./bjam install


<!-- Security -->
보안
--------
<!-- To help make your bitcoin installation more secure by making certain attacks impossible to exploit even if a vulnerability is found, binaries are hardened by default. This can be disabled with: -->
취약점이 발견된 경우에도 악용할수 없는 특정 공격을 방지하여 비트코인 설치의 보안을 강화하기위해 기본적으로 바이너리 파일이 강화된다. 이 기능은 다음과 같이 비활성화할수 있다.

<!-- Hardening Flags: -->
강화(hardening) 플래그 :

	./configure --enable-hardening
	./configure --disable-hardening


<!-- Hardening enables the following features: -->
강화기능을 사용하면 다음 기능을 사용할수 있다.

<!-- * Position Independent Executable -->
		<!-- Build position independent code to take advantage of Address Space Layout Randomization offered by some kernels. Attackers who can cause execution of code at an arbitrary memory location are thwarted if they don't know where anything useful is located. The stack and heap are randomly located by default but this allows the code section to be randomly located as well. -->
* 독립적으로 실행하는 포지션
		일부 커널에서 제공하는 주소 공간 레이아웃 임의 설정을 활용하려면 위치 독립 코드를 작성합니다. 임의 메모리 위치에서 코드를 실행할 수 있는 공격자는 유용한 위치를 모를 경우 방해를 받는다. 스택과 힙은 기본적으로 임의의 위치에 배치되지만 이를 통해 코드 섹션도 임의로 배치할 수 있습니다.

    <!-- On an AMD64 processor where a library was not compiled with -fPIC, this will cause an error such as: "relocation R_X86_64_32 against '......' can not be used when making a shared object;" -->
		AMD64 프로세서에서 -fPIC 로 라이브러리를 컴파일하지 않으면 다음과 같은 오류가 발생합니다 : "relocation R_X86_64_32 against '......' can not be used when making a shared object;"

    <!-- To test that you have built PIE executable, install scanelf, part of paxutils, and use: -->
		PIE 실행파일을 빌드했는지 테스트하려면, paxutils 의 일부인 scanelf 를 설치하고 다음을 사용하라.

    	scanelf -e ./bitcoin

    <!-- The output should contain: -->
		출력결과는 다음이 포함되어야 한다 :

		TYPE ET_DYN

<!-- * Non-executable Stack -->
    <!-- If the stack is executable then trivial stack based buffer overflow exploits are possible if vulnerable buffers are found. By default, bitcoin should be built with a non-executable stack but if one of the libraries it uses asks for an executable stack or someone makes a mistake and uses a compiler extension which requires an executable stack, it will silently build an executable without the non-executable stack protection. -->
* 비 실행 스택
		스택이 실행 가능한 경우 취약한 버퍼가 발견되면 작은 스택 기반 버퍼 오버 플로 악용이 가능하다. 기본적으로 비트코인은 비 실행 스택으로 구축해야 하지만 이 패키지를 사용하는 라이브러리 중 하나에서 실행 파일 스택을 요청하거나 다른 사람이 오류를 일으켜 실행 파일 보호 기능이 필요하지 않는 컴파일러 확장을 사용하는 경우 비 실행 스택의 보호 없이 자동으로 실행파일을 작성한다.

    <!-- To verify that the stack is non-executable after compiling use: -->
		스택을 컴파일 한 후에 비 실행 여부를 확인하려면 다음을 실행한다.
    `scanelf -e ./bitcoin`

    <!-- the output should contain: -->
		출력결과는 다음이 포함되어야 한다 : STK/REL/PTL RW- R-- RW-

    <!-- The STK RW- means that the stack is readable and writeable but not executable. -->
		STK RW- 은 이것이 스택이 읽고 쓰기가 되지만 실행이 불가능하다는것을 의미한다.

<!-- Disable-wallet mode -->
지갑-사용중지 모드
--------------------
<!-- When the intention is to run only a P2P node without a wallet, bitcoin may be compiled in disable-wallet mode with: -->
지갑이 없이 P2P 노드 만 실행하는 경우, 비트코인은 다음과 같이 지갑-사용중지 모드로 컴파일이 가능하다.

    ./configure --disable-wallet

<!-- In this case there is no dependency on Berkeley DB 4.8. -->
이 경우 버클리 데이터베이스에 대한 의존성은 없다.

<!-- Mining is also possible in disable-wallet mode, but only using the `getblocktemplate` RPC call not `getwork`. -->
채굴은 지갑-사용중지 모드에서도 가능하지만, `getwork`가 아닌 오직 `getblocktemplate` RPC 호출을 사용해야한다.

<!-- Additional Configure Flags -->
추가 구성 플래그
--------------------------
<!-- A list of additional configure flags can be displayed with: -->
추가 구성 플래그 목록은 다음을 사용하여 확인이 가능하다.

    ./configure --help


<!-- 여기부터 아직 번역안함 -->
Setup and Build Example: Arch Linux
-----------------------------------
This example lists the steps necessary to setup and build a command line only, non-wallet distribution of the latest changes on Arch Linux:

    pacman -S git base-devel boost libevent python
    git clone https://github.com/bitcoin/bitcoin.git
    cd bitcoin/
    ./autogen.sh
    ./configure --disable-wallet --without-gui --without-miniupnpc
    make check

Note:
Enabling wallet support requires either compiling against a Berkeley DB newer than 4.8 (package `db`) using `--with-incompatible-bdb`,
or building and depending on a local version of Berkeley DB 4.8. The readily available Arch Linux packages are currently built using
`--with-incompatible-bdb` according to the [PKGBUILD](https://projects.archlinux.org/svntogit/community.git/tree/bitcoin/trunk/PKGBUILD).
As mentioned above, when maintaining portability of the wallet between the standard Bitcoin Core distributions and independently built
node software is desired, Berkeley DB 4.8 must be used.


ARM Cross-compilation
-------------------
These steps can be performed on, for example, an Ubuntu VM. The depends system
will also work on other Linux distributions, however the commands for
installing the toolchain will be different.

Make sure you install the build requirements mentioned above.
Then, install the toolchain and curl:

    sudo apt-get install g++-arm-linux-gnueabihf curl

To build executables for ARM:

    cd depends
    make HOST=arm-linux-gnueabihf NO_QT=1
    cd ..
    ./configure --prefix=$PWD/depends/arm-linux-gnueabihf --enable-glibc-back-compat --enable-reduce-exports LDFLAGS=-static-libstdc++
    make


For further documentation on the depends system see [README.md](../depends/README.md) in the depends directory.

Building on FreeBSD
--------------------

(Updated as of FreeBSD 11.0)

Clang is installed by default as `cc` compiler, this makes it easier to get
started than on [OpenBSD](build-openbsd.md). Installing dependencies:

    pkg install autoconf automake libtool pkgconf
    pkg install boost-libs openssl libevent
    pkg install gmake

You need to use GNU make (`gmake`) instead of `make`.
(`libressl` instead of `openssl` will also work)

For the wallet (optional):

    ./contrib/install_db4.sh `pwd`
    setenv BDB_PREFIX $PWD/db4

Then build using:

    ./autogen.sh
    ./configure BDB_CFLAGS="-I${BDB_PREFIX}/include" BDB_LIBS="-L${BDB_PREFIX}/lib -ldb_cxx"
    gmake

*Note on debugging*: The version of `gdb` installed by default is [ancient and considered harmful](https://wiki.freebsd.org/GdbRetirement).
It is not suitable for debugging a multi-threaded C++ program, not even for getting backtraces. Please install the package `gdb` and
use the versioned gdb command e.g. `gdb7111`.
