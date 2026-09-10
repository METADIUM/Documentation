# 메타디움 메인넷: 개발자 가이드 문서
2026년 9월 10일

## 목차
- [1.Introduction](#1introduction)
- [2.Metadium Blockchain Full Client (gmet)](#2metadium-blockchain-full-client-gmet)
  * [Download Source Code](#download-source-code)
  * [Install](#install)
  * [Address Format](#address-format)
  * [Coin Balance](#coin-balance)
  * [Interface (JSON RPC)](#interface-json-rpc)
  * [Network ID & Chain ID](#network-id--chain-id)
  * [Transaction Fee](#transaction-fee)
  * [EVM Version](#evm-version)
- [3.Block Explorer](#3block-explorer)
- [4.Testnet META Faucet](#4-testnet-meta-faucet)
- [5.Block Interval](#5-block-interval)
- [6.BIP32 HD wallet Rule (Reference)](#6-bip32-hd-wallet-rule-reference)
- [7.How to Connect Metamask with Metadium Testnet](#7-how-to-connect-metamask-with-metadium-testnet)
- [8.Link](#8link)
  * Source Code
  * Open API Server
  * Block Explorer
  * Test Meta Faucet
  * Gmet Install Guide
- [9.Contact](#9contact)
  * Technical Support
<br>

## 1.Introduction
본 문서는 Metadium Mainnet을 지원하려는 거래소 기술팀과 개일 개발자들을 위한 참고문서로서 Metadium Testnet/ Mainnet Integration을 위하여 작성하였다. 현 문서는 Camellia 업그레이드(Cancun EVM) 시점의 Metadium Mainnet 과 Testnet 을 기준으로 한다.
<br>
<br>

## 2.Metadium Blockchain Full Client (gmet)
In order to connect with Metadium blockchain, you must install Metadium client `gmet` and finish complete block sync.
<br>

### Download Source Code
Metadium Client는 Go-Metadium(gmet)이라고 칭하며 Ethereum의 Go-Ethereum clinet를 forking하여 만들어졌다. 해당 source code는 [Github Repository](https://github.com/METADIUM/go-metadium)에서 다운받을수있고, 배포용 바이너리는 [Releases 페이지](https://github.com/METADIUM/go-metadium/releases)에 게시된다.

<br>

### Install
**소스 빌드가 아니라 게시된 릴리스 산출물을 사용한다.** 릴리스마다 DB 엔진별로 Linux tarball 두 개가 제공된다 — `metadium-<version>-linux-leveldb.tar.gz` 와 `metadium-<version>-linux-rocksdb.tar.gz`. 운영 중인 노드의 엔진에 맞는 것을 [Releases 페이지](https://github.com/METADIUM/go-metadium/releases)에서 받아 노드 디렉터리에 덮어쓰고 재시작한다. **노드의 엔진은 바이너리 교체로 바꿀 수 없다** — 기존 chaindata가 엔진을 결정한다.

**최소 OS: Ubuntu 20.04 이상**(glibc 2.31+). 릴리스 바이너리가 그 하한으로 빌드되므로 그보다 낮은 환경에서는 기동하지 않는다.

RocksDB 빌드는 아래 공유 라이브러리가 호스트에 있어야 한다.
<pre><code>sudo apt-get install -y libjemalloc2 libsnappy1v5 liblz4-1 libzstd1</pre></code>

소스 빌드는 개발용이며 **Go 1.22 이상**이 필요하다. 빌드 방법은 [readme](https://github.com/METADIUM/go-metadium/blob/master/README.md)를 참조한다. 직접 빌드한 바이너리는 이식성이 없으므로 운영에 배포하지 않는다.

설치가 끝나면 `gmet metadium new-account` 를 실행하여 로컬 노드에 새로운 계정을 만들수있다. 이제 gmet를 실행하고 Mainnet 또는 Testnet 네트워크에 연결할 수 있다. `gmet --help` 명령을 사용하면 다른 옵션과 명령을 확인할수있다.

아래는 `gmet` node를 full data sync 모드로 실행시키는 명령이다.
<pre><code>## MAINNET
{data_folder}/bin/gmet --syncmode full --datadir {data_folder} --http

## TESTNET
{data_folder}/bin/gmet --metadium-testnet --syncmode full --datadir {data_folder} --http</pre></code>

RPC listener 는 기본값이 `127.0.0.1` 이며 그대로 두는 것을 권장한다. 외부에 노출할 때는 reverse proxy 나 출발지를 제한하는 security group 뒤에 두고, 계정을 보유한 노드에서는 `0.0.0.0` 으로 바인딩하지 않는다.

**Full sync 가 유일하게 지원되는 모드다.** 이 네트워크는 snap/fast sync 를 서브하지 않으므로 `--syncmode full` 없이 기동한 노드는 초기 동기화를 끝내지 못한다. 모든 노드에 `--syncmode full` 을 유지한다.


gmet 실행파일에는 Metadium Team에서 공식적으로 운영하는 bootnode가 들어있기 때문에 gmet을 실행할때 bootnode를 수동으로 입력할 필요는 없다. 개별적으로 운영하는 bootnode를 등록하여 사용하려면 `--bootnodes` 옵션을 사용하여 추가하는것이 가능하다.
<br>

### Address Format
Metadium의 주소체계는 Ethereum과 동일하기 때문에 Ethereum주소를 그대로 사용하는것이 가능하다 (e.g., 0x0300cef240e54a14329f2dd4bf21bc5ca394eec1). 즉, 기존에 Metadium ERC20을 지원하는 거래소는 고객 cold wallet 주소의 변경없이 동일 주소를 사용하는것이 가능하다. Metadium Team에서 출시한 공식앱인 Keepin App을 통하여[Android](https://play.google.com/store/apps/details?id=com.coinplug.metadium&hl=en_US)와 [iOS](https://itunes.apple.com/us/app/keepin-metadium-id-wallet/id1452993752?mt=8)버전 application을 설치하는것이 가능하다.
<br>

### Coin Balance
Metadium Team은 2019년 2월 22일 3시 UTC, Metadium ERC20 contract을 freezing할 예정이다. Frozen된 ERC20 token의 잔고 snapshot은 Mainnet이 시작되는 2019년 3월 20일 genesis block에 기록되었으며, 금액은 1:1 비율로 전환되었다 (i.e, 100 ERC20 Metatoken = 100 Metadium Mainnet Coin).[Mainnet Block Explorer](https://explorer.metadium.com)를 통해서 기존 주소에 ERC20과 동일한 금액의 META가 생성되었는지 확인할수있다.
<br>

### Interface (JSON RPC)
Metadium 은 표준 Ethereum JSON-RPC API([execution-apis](https://ethereum.github.io/execution-apis/))를 구현하며, 여기에 fee delegation 트랜잭션 등 Metadium 고유 기능이 더해져 있다. 특정 method 에 의존하는 응용이라면 특정 go-ethereum 릴리스와 동일하다고 가정하지 말고 노드에 직접 확인한다. 아래는 Metadium Testnet의 Open API 서버를 이용한  test용 cURL sentence이다.

Gmet의 default port는 8588이며, gmet실행시 `--http.port 8545` 옵션을 사용하여 geth와 동일한 8545로 변경할수있다.

아래는 특정주소의 META token balance를 읽어오는 명령예이다.
<pre><code>## MAINNET
curl -i -X POST -H "Content-Type: application/json" -d '{"jsonrp
c":"2.0","method":"eth_getBalance","params":["0x47f7b1714603beEBf0643e7bA5c7D7c9b7eF7D27", "latest"],"id":1}' https://api.metadium.com/prod

## TESTNET
curl -i -X POST -H "Content-Type: application/json" -d '{"jsonrp
c":"2.0","method":"eth_getBalance","params":["0x47f7b1714603beEBf0643e7bA5c7D7c9b7eF7D27", "latest"],"id":1}' https://api.metadium.com/dev
</pre></code>

### [Network ID & Chain ID](https://chainid.network)
Metadium은 동일한 네트워크 ID와 체인 ID 쌍을 사용한다. 특히 Metamask와 같은 응용 프로그램을 사용할 때는 mainnet에 11을, testnet에 12를 직접 입력하여 사용해야한다.
<br>

### Transaction Fee
Gmet의 default gas price는 80Gwei이며 이보다 낮을 경우 confirm이 늦어질수있다. Full node를 통하여 출금거래를 만들때는 `eth_gasPrice` API를 통해서 현재 네트워크의 gasPrice를 반드시 읽어서 사용해야 한다.

출금거래를 만들기위해 gas price를 gmet node에서 읽어오는 방법은 다음과 같다.
<pre><code>## MAINNET
curl -i -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"eth_gasPrice","params":[],"id":1}' https://api.metadium.com/prod

## TESTNET
curl -i -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"eth_gasPrice","params":[],"id":1}' https://api.metadium.com/dev
</pre></code>
<br>

### EVM Version
Metadium 의 EVM 은 **Cancun**(Camellia 업그레이드)이며, Mainnet 은 블록 **117,764,000**, Testnet 은 **86,449,000** 부터 활성화되어 있다. 컨트랙트는 `--evm-version cancun` 으로 컴파일한다(툴체인 기본값이 Cancun 이하를 타깃하면 그대로 두어도 된다). 해당 블록 이후로 `PUSH0`, `TLOAD`/`TSTORE`, `MCOPY`, `BLOBBASEFEE` 와 warm `COINBASE` 를 사용할 수 있으며, Cancun 보다 최신 EVM 버전으로 컴파일한 컨트랙트는 동작하지 않는다.
<br>
<br>

## 3.Block Explorer
Metadium Block Explorer는 smart contract 검증, 읽기, 쓰기 등 Etherscan의 거의 모든 기능을 가지고 있으며, 가입자에 한하여 Open API 서비스를 무료로 제공한다. 개발자들은 [Mainnet Explorer](https://explorer.metadium.com) 와 [Testnet Explorer](https://testnetexplorer.metadium.com)에서 직접 가입하여 API 서비스를 이용해보기 바란다.
<br>
<br>

## 4. Testnet META Faucet
Metadium blockchain에서 사용되는 native coin인 META는 [Testnet META Faucet](https://testnetfaucet.metadium.com)에서 받을수 있다. Faucet은 요청한 주소로 10 META coin을 즉시 전송하며 수신주소의 balance 및 송신요청 횟수에 시간제한이 있다. 또한 요청자가 동일 IP를 사용할 경우에도 시간 제약이 있다.


거래소 dev/ staging에서 사용할 다량의 META가 필요한 경우, tech@metadium.com으로 연락하여 전송받을수 있다.
<br>
<br>

## 5. Block Interval
Metadium 은 permissioned Proof-of-Authority 합의를 사용한다. 거버넌스 컨트랙트를 통해 온체인에 등록된 고정된 블록 생성자 집합이 블록을 만들고, 멤버십 조건으로 staking 을 요구한다. 블록 생성은 높이에 따라 이들 사이에서 순환한다.

|   | Testnet | Mainnet |
| -------------------------------------------- | ----------- | ----------- |
| Block interval | 2초 | 2초 |

블록 간격은 클라이언트 설정이 아니라 거버넌스 파라미터(`blockCreationTime`)이며 양 네트워크 모두 2초다. 블록은 트랜잭션 유무와 무관하게 자기 slot 이 지나면 봉인되므로, slot 중간에 제출된 트랜잭션은 평균적으로 약 한 간격 안에 확정된다 — 즉시 확정되는 것이 아니다.

<br>
<br>

## 6. BIP32 HD wallet Rule (Reference)
Metadium Mainnet은 SLIP-0044 (Registered coin types for BIP-0044)에 등록되어 있으며 mnemonic으로 생성하는 private key의 기본 path는 아래와 같다.
<pre><code>m/44'/916'/0'/0</pre></code>

https://github.com/satoshilabs/slips/blob/master/slip-0044.md

해당 basic path는 Metadium에서 사용할 private key를 직접 생성하는 앱이나 웹을 생성할때 참고하도록 한다.
<br>
<br>

## 7. How to Connect Metamask with Metadium Testnet
Metamask를 이용해서 Metadium Mainnet/ Testnet send/receive 기능을 수행할수있다. 또한 Metamask와 http://remix.ethereum.org/를 통하여 smart contract을 배포하고 동작 테스트를 진행하는 것이 가능하다.

Metamask를 통해서 send를 실행할때는 반드시 Gas Price를 80 GWei이상으로 설정해야한다.

* Step#1: 구글 크롬 브라우저의 익스텐션인 Metamak를 [Web Store](https://chrome.google.com/webstore/search/metamask?utm_source=chrome-ntp-icon)를 통해 인스톨하세요.
* Step#2: 오른쪽 위의 네트워크 설정에서 → “사용자 정의 RPC”를 선택하세요.
* Step#3:  “설정”에서  → “고급 옵션 보기”를 선택하세요.
* Step#4:  Metadium네트워크 설정 정보를 입력한 후 → 저장하신 후 → 설정을 닫아주세요.
  - 메인넷(Mainnet) 설정 정보 <pre><code>New RPC URL: https://api.metadium.com/prod
  ChainID: 11
  Symbol: META
  Nickname: META Mainnet
  </pre></code>
  - 테스트넷(Testnet)설정 정보 <pre><code>New RPC URL: https://api.metadium.com/dev
  ChainID: 12
  Symbol: META
  Nickname: META Testnet
  </pre></code>
* Step#5: 메타디움 네트워크와 연동되었습니다.(메인넷 혹은 테스트넷)
* Step#6: 잔고를 확인하세요.
* Step#7: 보내기를 선택하세요. → 받을 주소와 금액을 선택 한 후 → “고급 옵션”을 선택하세요.
* Step#8: 가스 가격(GWEI)를 80 이상으로 입력하시고 → 저장을 선택하신 후 → 다음을 선택하세요. → 내역을 확인하고 “확인” 버튼을 선택하세요.
* Step#9: 보낸 주소 혹은 받는 주소를 복사하신 후 → [Metadium Block Explorer](https://explorer.metadium.com/)로 이동하셔서 → 검색 창에 복사한 주소를 검색하시면 → [해당 주소의 거래 내역 리스트를 확인하세요.](https://explorer.metadium.com/addresses/)
<br>
<br>

## 8. 관련 링크
* [소스코드](https://github.com/metadium/go-metadium)
* 오픈 API 서버
  - [메인넷](https://api.metadium.com/prod)
  - [테스트넷](https://api.metadium.com/dev)
* 블록익스플로러
  - [메인넷](https://explorer.metadium.com)
  - [테스트넷](https://testnetexplorer.metadium.com)
* [테스트넷 META Faucet](https://testnetfaucet.metadium.com)
* [Gmet 인스톨 가이드](https://github.com/METADIUM/go-metadium/blob/master/README.md)
<br>
<br>

## 9. Contact
* Technical Support tech@metadium.com
  

