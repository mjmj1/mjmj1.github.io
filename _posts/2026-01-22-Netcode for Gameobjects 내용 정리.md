---
layout: post
title: Netcode for Gameobjects 내용 정리
date: 2026-01-22 21:51:37 +0900
categories:
  - study
summary: The Zoo 프로젝트를 진행하면서 학습했던 Netcode for GameObjects 라이브러리의 내용을 정리함
project: The Zoo
tech:
  - Unity
  - Netcode for Gameobjects
awards: []
---

## NetworkManager

### 공식 문서 설명

- Netcode for GameObjects에서 네트워크 세션을 시작·종료하고 전체 네트워크 상태를 관리하는 중앙 컴포넌트
- 프로젝트에는 반드시 하나의 NetworkManager가 존재해야 하며, 서버·호스트·클라이언트 모드 전환의 기준점 역할을 수행
- Transport, PlayerPrefab, NetworkPrefabs, Scene Management 등 핵심 네트워크 설정을 총괄

### API

- `StartServer()` : 서버 모드로 네트워크 세션을 시작
- `StartHost()` : 서버와 로컬 클라이언트를 동시에 실행하는 호스트 모드로 시작
- `StartClient()` : 클라이언트 모드로 서버 또는 호스트에 연결을 시도
- `Shutdown()` : 현재 네트워크 세션을 종료
- `DisconnectClient(ulong clientId)` : 서버 측에서 특정 클라이언트를 강제로 연결 해제

### 이벤트 함수

- `OnClientConnectedCallback(ulong clientId)` : 클라이언트가 연결되었을 때 호출, 서버(호스트 포함) 측에서 호출
- `OnClientDisconnectCallback(ulong clientId)` : 클라이언트 연결이 종료되었을 때 호출, 서버(호스트 포함) 측에서 호출
- `OnClientStarted()` : 로컬 클라이언트가 준비되었을 때 호출되는 콜백
- `OnClientStopped(bool isHost)` : 로컬 클라이언트가 중지될 때 호출되는 콜백 (파라미터는 호스트 모드 여부)
- `OnConnectionEvent(NetworkManager manager, ConnectionEventData data)` : 연결 관련 이벤트를 통합적으로 전달하는 콜백
- `OnServerStarted()` : 로컬 서버가 시작되어 연결을 수신할 때 호출되는 콜백임
- `OnServerStopped(bool isHost)` : 로컬 서버가 중지될 때 호출되는 콜백 (파라미터는 호스트 모드 여부)
- `OnTransportFailure()` : NetworkTransport 실패 시 호출되는 콜백이며, 호출 이후 NetworkManager는 Shutdown 흐름으로 진행
- `OnPreShutdown()` : NetworkManager가 셧다운 중 정리 작업을 수행하기 전에 호출되는 콜백
- `OnInstantiated(NetworkManager manager)` : NetworkManager 인스턴스가 생성되었을 때 호출되는 정적(static) 이벤트
- `OnDestroying(NetworkManager manager)` : NetworkManager 인스턴스가 파괴될 때 호출되는 정적(static) 이벤트
- `OnSessionOwnerPromoted(ulong newSessionOwnerClientId)` : 분산 권한 토폴로지에서 세션 오너가 승격될 때 전체 클라이언트에 발생하는 이벤트
- `OnReanticipate(double lastRoundTripTime)` : Anticipation(예측/재시뮬레이션) 관련 값들의 재적용 이후 호출되는 콜백

### 분산 권한 설정 방법

[Quick Start 공식 문서](https://docs.unity3d.com/Packages/com.unity.netcode.gameobjects@2.5/manual/learn/distributed-authority-quick-start.html)

![](https://docs.unity3d.com/Packages/com.unity.netcode.gameobjects@2.5/manual/images/learn/distributed-authority-quick-start/network-manager.png){: width="400"}
_1. 빈 오브젝트에 NetworkManager 스크립트 적용_

![](https://docs.unity3d.com/Packages/com.unity.netcode.gameobjects@2.5/manual/images/learn/distributed-authority-quick-start/network-topology.png){: width="400"}
_2. 인스펙터에서 Network Topology설정 변경_

![](https://docs.unity3d.com/Packages/com.unity.netcode.gameobjects@2.5/manual/images/learn/distributed-authority-quick-start/transport.png){: width="400"}
_3. Network Transport 수정_

### 이슈 및 해결

- 씬 변경 시 네트워크 연결이 종료되는 현상
  - 원인
    - 게임 오브젝트에 다른 스크립트를 같이 적용함
  - 해결
    - `NetworkManager`를 별도의 전용 오브젝트로 분리하여 해결

---

## Scene Management (Network Scene Management)

### 공식 문서 설명

- Netcode for GameObjects는 서버 주도의 씬 동기화를 제공
- NetworkSceneManager를 통해 서버에서 씬 전환을 수행하면 모든 클라이언트가 자동 동기화

### 핵심 원칙

- 씬 전환은 반드시 서버(또는 Host)에서만 수행
- 클라이언트 단독 `SceneManager.LoadScene()` 호출은 동기화 붕괴의 원인
- NetworkObject는 씬 전환 시 자동으로 Spawn/Despawn 처리됨

### 주요 개념

- 로비 씬 / 인게임 씬 / 결과 씬 간 흐름을 서버가 제어
- Late-join 클라이언트도 현재 서버 씬 상태로 동기화됨

### 사용법

```csharp
NetworkManager.Singleton.SceneManager.LoadScene(sceneName, LoadSceneMode.Single);
```

---

## ISession

### 공식 문서 설명

- Multiplayer Services에서 세션을 표현하는 인터페이스
- 세션 상태, 참가자, 속성(Properties) 등 메타데이터 관리 계층
- 실시간 게임 상태는 NGO, 로비·매치 정보는 ISession이 담당

### 이벤트

- `Changed` : 세션 데이터 전반이 변경되었을 때 호출됨
- `StateChanged(SessionState newState)` : 세션 상태(State)가 변경되었을 때 호출
- `PlayerJoined(string playerId)` : 플레이어가 세션에 참가했을 때 호출
- `PlayerLeaving(string playerId)` : 플레이어가 세션을 떠나기 직전에 호출
- `PlayerHasLeft(string playerId)` : 플레이어가 세션을 완전히 떠난 이후 호출
- `SessionPropertiesChanged` : 세션 Properties가 변경되었을 때 호출
- `PlayerPropertiesChanged` : 플레이어 Properties가 변경되었을 때 호출
- `RemovedFromSession` : 현재 플레이어가 세션에서 제거되었을 때 호출
- `Deleted` : 세션이 삭제되었을 때 호출
- `SessionHostChanged(string newHostPlayerId)` : 세션 Host가 변경되었을 때 호출

### 파라미터

- `Type` : 세션 타입
- `Name` : 세션 이름
- `Id` : 세션 ID
- `Code` : 세션 Join Code
- `IsHost` : 현재 플레이어가 Host인지 여부
- `IsPrivate` : 비공개 세션 여부
- `IsLocked` : 세션 잠금 여부
- `HasPassword` : 비밀번호 설정 여부
- `AvailableSlots` : 남은 참가 가능 슬롯 수
- `MaxPlayers` : 최대 참가자 수
- `PlayerCount` : 현재 참가자 수
- `Players` : 세션 참가자 목록
- `Properties` : 세션 속성(Dictionary 형태의 메타데이터)
- `Host` : 현재 Host의 Player ID
- `State` : 세션 상태
- `CurrentPlayer` : 현재 플레이어 정보

### API

- `SaveCurrentPlayerDataAsync()` : 현재 플레이어 데이터 변경 사항을 세션에 저장
- `LeaveAsync()` : 세션 나가기
- `RefreshAsync()` : 세션 정보를 서버로부터 다시 동기화
- `ReconnectAsync()` : 세션 재연결을 시도
- `AsHost()` : Host 전용 기능에 접근하기 위한 `IHostSession`으로 변환

### SessionProperty

#### 개념 및 용도

- 세션 전체에 공통으로 적용되는 메타데이터
- 로비 설정, 게임 규칙, 매치 옵션 등 모든 플레이어가 공유해야 하는 정보에 사용
- `ISession.Properties`를 통해 관리됨

#### 사용 사례

- 게임 모드
- 제한 시간
- 게임 설정 등

#### 특징

- 낮은 변경 빈도를 전제로 설계됨
- 실시간 게임 상태 동기화 용도로는 부적합
- `SessionPropertiesChanged` 이벤트를 통해 변경 감지

### PlayerProperty

#### 개념 및 용도

- 세션에 속한 개별 플레이어의 메타데이터
- 플레이어의 선택 또는 로비 단계 상태를 표현하는 데 사용

#### 사용 사례

- 플레이어 닉네임
- 준비 완료 여부 등

#### 특징

- 모든 플레이어가 조회 가능
- `PlayerPropertiesChanged` 이벤트를 통해 변경 감지
- 로비 단계에서 주로 사용됨

### 이슈 및 해결

- `ISession` 오너와 `NetworkManager` 오너 불일치로 씬 변경 불가, 호스트를 변경할 때 발생
  - 해결
    - ISession 오너를 기준으로 게임 흐름을 제어하고, RPC를 통해 NetworkManager에 씬 변경 요청

---

## Ownership

### 공식 문서 설명

- 특정 NetworkObject를 누가 제어하는지를 나타내는 개념
- Netcode for GameObjects에서는 기본적으로 각 NetworkObject마다 하나의 Owner(ClientId) 가 존재함
- Owner는 입력 처리, RPC 호출 권한, Authority 판단의 기준이 됨

### 핵심 개념 정리

#### Owner

- `OwnerClientId`로 식별되는 오브젝트의 소유 클라이언트
- Owner는 해당 오브젝트에 대해 다음 권한을 가짐
  - Owner 전용 RPC 수신 (`SendTo.Owner`)
  - 입력 기반 제어 로직 수행
  - Owner Authority 설정 시 상태 변경 주체

#### Authority

- 상태를 결정하고 동기화의 기준이 되는 주체
- Authority는 설정에 따라 다음 중 하나로 결정됨
  - Server Authority (기본)
  - Owner Authority (분산 권한)

| **구분**            | **Server Authority (서버 권한)**        | **Owner Authority (분산 권한)**       |
| ------------------- | --------------------------------------- | ------------------------------------- |
| **상태 결정**       | **Server**가 결정                       | **Owner**가 결정                      |
| **입력 처리**       | Client가 입력 전송 → Server가 이동 처리 | Owner가 입력 즉시 반영 (Zero Latency) |
| **NetworkVariable** | Server만 쓰기 가능                      | Owner도 쓰기 가능 (권한 설정 필요)    |
| **주요 사용처**     | 경쟁형 게임, 보안 중요 로직             | 협동 게임, VR, 빠른 반응성 필요 게임  |

### Ownership과 RPC의 관계

- RPC는 Owner/Authority 기준으로 수신 대상이 결정
- 분산 권한 환경에서는 다음 규칙이 중요함

```csharp
// 1. Owner에게만 실행 (예: "너 피격됐어" UI 표시)
[Rpc(SendTo.Owner)]
void ShowDamageEffectRpc() { /* UI 로직 */ }

// 2. 권한자에게 실행 (예: "나 때렸어" 판정 요청)
// Server Auth라면 서버로, Owner Auth라면 해당 소유자에게 감
[Rpc(SendTo.Authority)]
void SubmitHitRpc() { /* 데미지 계산 및 체력 차감 */ }

// 3. 소유자가 아닌 모두에게 (예: 다른 사람 눈에만 보이는 이펙트)
[Rpc(SendTo.NotOwner)]
void ShowMovementTrailRpc() { /* 트레일 렌더러 */ }
```

- `SendTo.Owner` : 오브젝트의 Owner에게 전송
- `SendTo.Authority` : Authority 주체에게 전송
- `SendTo.NotOwner` / `SendTo.NotAuthority` : 나머지 인스턴스에게 전송

### Ownership 변경

런타임에 소유권을 이전해야 할 때 사용 (예: 아이템 줍기, 차량 탑승)

#### ChangeOwnership

```csharp
networkObject.ChangeOwnership(newOwnerClientId);
```

- 서버 또는 Authority에서만 호출 가능
- Owner 변경 시 다음 사항에 주의
  - 입력 제어 주체 변경
  - RPC 수신 대상 변경
  - NetworkVariable WritePermission 영향

### PlayerObject와 일반 NetworkObject의 Ownership 차이

#### PlayerObject

- 연결된 클라이언트가 자동으로 Owner가 됨
- Disconnect 시 자동 Despawn 또는 재할당

#### 일반 NetworkObject

- Spawn 시 Owner를 명시적으로 지정 가능
- AI NPC, 상호작용 오브젝트 등에 사용

### 분산 권한 토폴로지 상황별 진리표

`IsOwner`는 "내 것인가?"를 묻는 것이고, `IsServer`는 "내가 호스트/서버인가?"를 묻는 것. 두 값은 상황에 따라 일치할 수도, 다를 수도 있음

#### 내가 방장(Host)일 때

| **대상 오브젝트** | **IsLocalPlayer** | **IsOwner** | **IsServer** | **HasAuthority** | **IsSessionOwner** | **ServerIsHost** | **IsClient** | **IsHost** | **IsOwnedByServer** |
| ----------------- | ----------------- | ----------- | ------------ | ---------------- | ------------------ | ---------------- | ------------ | ---------- | ------------------- |
| **Host**          | True              | True        | False        | True             | True               | False            | True         | False      | False               |
| **Client**        | False             | False       | False        | False            | True               | False            | True         | False      | False               |
| **NPC (AI)**      | False             | True        | False        | True             | True               | False            | True         | False      | False               |

#### 내가 일반 클라이언트(Client)로 접속했을 때

| **대상 오브젝트** | **IsLocalPlayer** | **IsOwner** | **IsServer** | **HasAuthority** | **IsSessionOwner** | **ServerIsHost** | **IsClient** | **IsHost** | **IsOwnedByServer** |
| ----------------- | ----------------- | ----------- | ------------ | ---------------- | ------------------ | ---------------- | ------------ | ---------- | ------------------- |
| **Host**          | False             | False       | False        | False            | False              | False            | True         | False      | False               |
| **Client(나)**    | True              | True        | False        | True             | False              | False            | True         | False      | False               |
| **다른 Client**   | False             | False       | False        | False            | False              | False            | True         | False      | False               |
| **NPC (AI)**      | False             | False       | False        | False            | False              | False            | True         | False      | False               |

---

## Network Visibility

### 공식 문서 설명

- 특정 `NetworkObject`가 특정 클라이언트에게 네트워크 상에서 보이는지(Visible) 여부
- 클라이언트가 해당 오브젝트를 Spawn/동기화받아 업데이트를 수신할지를 결정
- **Visible**: 서버는 해당 클라이언트가 Spawn된 복제본을 갖도록 보장하고, 그 오브젝트 관련 네트워크 트래픽을 전송
- **Hidden**: 클라이언트는 해당 오브젝트를 Despawn 및 Destroy 처리하며, 더 이상 관련 네트워크 업데이트를 받지 않음

### 기본 가시성 정책

- 기본적으로 Netcode for GameObjects는 `CheckObjectVisibility`가 등록되지 않았을 때 모든 클라이언트에게 Visible로 처리
- Visibility 판단은 서버 측에서 수행
- Visibility는 Spawn 시점뿐 아니라 Spawn 이후에도 변경할 수 있음

### API

#### Visibility 판단 콜백

- `NetworkObject.CheckObjectVisibility`
  - 서버에서 클라이언트별 가시성을 판단하기 위한 콜백
  - `ulong clientId`를 인자로 받아 `bool`을 반환
    - `true` : 해당 클라이언트에게 Visible
    - `false` : 해당 클라이언트에게 Hidden

```csharp
// NetworkObject.CheckObjectVisibility += CheckVisibility;
bool CheckVisibility(ulong clientId)
{
    return true;
}
```

#### Visibility 제어 API

> 상대 클라이언트(`clientId`)로부터 내가 적용받는 것

- `NetworkObject.NetworkShow(ulong clientId)`: 특정 클라이언트에게 NetworkObject를 Visible 상태로 설정
- `NetworkObject.NetworkHide(ulong clientId)`: 특정 클라이언트에게 NetworkObject를 Hidden 상태로 설정
- `NetworkObject.NetworkShow(IEnumerable<NetworkObject>, ulong clientId)`: 여러 NetworkObject를 특정 클라이언트에게 Visible 상태로 설정
- `NetworkObject.NetworkHide(IEnumerable<NetworkObject>, ulong clientId)`: 여러 NetworkObject를 특정 클라이언트에게 Hidden 상태로 설정
- `NetworkObject.IsNetworkVisibleTo(ulong clientId)`: 특정 클라이언트에게 현재 Visible 상태인지 확인

#### Observer 없이 Spawn

- NetworkObject는 기본적으로 Spawn 시 모든 Observer(클라이언트)를 가짐
- `SpawnWithObservers = false`로 설정하면, 어떤 클라이언트에도 보이지 않는 상태로 Spawn 가능

```csharp
NetworkObject.SpawnWithObservers = false;
NetworkObject.Spawn();
```

### 사용 시 주의사항

- Visibility는 서버에서만 제어하는 것이 원칙
- `CheckObjectVisibility`는 서버에서 등록 및 해제해야 함
- Hidden 상태의 NetworkObject는 클라이언트에 존재하지 않으므로 아래 내용은 수신할 수 없음
  - 해당 오브젝트의 RPC
  - NetworkVariable 업데이트
- 특정 클라이언트에서 “RPC가 호출되지 않은 것처럼 보이는 현상”은 Visibility 정책에 의해 오브젝트가 Hidden 상태였기 때문일 수 있음
- Late-join 클라이언트에 대해서도 Visibility 정책이 일관되게 적용되도록 설계 필요

---

## RPC(Remote Procedure Call)

### 공식 문서 설명

- 네트워크 상의 다른 인스턴스(서버 또는 클라이언트)에서 메서드를 호출하기 위한 메시지 기반 메커니즘
- 전송 대상과 실행 위치가 명확히 정의되어야 함
- `[Rpc]` Attribute로 선언하며, 메서드 이름에는 `Rpc` 접미사 사용
- `SendTo` 옵션을 통해 컴파일 시점에 수신 대상이 결정됨

### SendTo 옵션 정리

- `SendTo.Server` : 서버
- `SendTo.NotServer` : 서버를 제외한 모든 클라이언트
- `SendTo.Owner` : 해당 NetworkObject의 Owner 클라이언트
- `SendTo.NotOwner` : Owner를 제외한 클라이언트
- `SendTo.Me` : 네트워크 전송 없이 로컬 인스턴스
- `SendTo.Everyone` : 서버를 포함한 모든 인스턴스
- `SendTo.ClientsAndHost` : 호스트 모드에서 서버와 클라이언트 모두
- `SendTo.SpecifiedInParams` : RpcParams로 지정된 대상

### 분산 권한에서의 SendTo

- 분산 권한에서는 오브젝트별 Owner가 Authority 역할을 수행
- `SendTo.Authority` : 해당 오브젝트의 Owner
- `SendTo.NotAuthority` : Owner를 제외한 나머지 인스턴스

### SendTo.SpecifiedInParams 사용법

```csharp
RuntimeSpecifiedRpc(RpcTarget.Single(/* clientId */, RpcTargetUse.Temp));
```

```csharp
[Rpc(SendTo.SpecifiedInParams)]
public void RuntimeSpecifiedRpc(RpcParams rpcParams = default)
{
    /* ... */
}
```

#### RpcTargetUse

- RpcTarget으로 지정한 대상 정보를 어떻게 관리할지 결정하는 옵션
- 대상 지정이 일회성인지, 재사용 가능한지에 따라 선택함
- `RpcTargetUse.Temp` : 일회성 대상 지정, 호출 이후 캐시되지 않음
- `RpcTargetUse.Persistent` : 대상 정보가 유지되며 여러 번 재사용 가능

### 사용법

- 상태 변경 및 검증 로직은 `SendTo.Server` RPC로 서버에서 처리함
- 연출, 애니메이션, UI 반영과 같은 피드백 로직은 클라이언트 대상 RPC로 분리함
- 특정 클라이언트만 대상으로 할 경우 `SendTo.SpecifiedInParams`와 `RpcParams`를 사용함
- 지속 상태 데이터는 RPC가 아닌 NetworkVariable로 관리함

### 이슈 및 해결

- AI NPC와의 상호작용을 할 때, 어디로 Rpc를 보내야하는지
  - 해결
    - AI NPC의 소유권을 가진 클라이언트에게 RPC를 전송하도록 `SendTo.SpecifiedInParams` 사용

---

## NetworkVariable

### 공식 문서 설명

- NetworkVariable은 서버와 클라이언트 간 속성을 지속적으로 동기화하기 위한 수단
- RPC나 Custom Message는 특정 시점에 한 번 전달되는 일회성 통신이며, 전송 시점에 연결되어 있지 않은 클라이언트에는 공유되지 않음
- NetworkVariable은 지속 상태를 동기화하므로, 늦게 접속한 클라이언트에게도 현재 값이 자동으로 전달됨
- 제네릭 타입 `T`를 감싸는 구조이며, 실제 값 접근은 `NetworkVariable.Value`를 통해 수행

### 동기화 시점

- 새 클라이언트가 게임에 접속했을 때
- NetworkBehaviour가 포함된 NetworkObject가 Spawn 되었을 때
- 값이 변경되었을 때, 변경 이전 값과 변경 이후 값이 `OnValueChanged` 이벤트를 통해 전달됨

### `OnValueChanged`

- 콜백에는 두 개의 파라미터가 전달됨
  - Previous: 변경 이전 값
  - Current: 변경 이후 값
- Read / Write Permission을 통해 접근 권한을 제어
- 필요 시 Custom NetworkVariable을 정의 가능

---

## NetworkBehaviour

### 공식 문서 설명

- MonoBehaviour를 상속하며 네트워크 기능을 제공하는 기본 단위
- NetworkObject와 함께 동작하며, Spawn 시점을 기준으로 네트워크 생명주기를 가짐
- 네트워크 권한 정보는 Spawn 이후 확정됨

### 생명주기 정리

- Awake → OnEnable → Start
- (네트워크 연결 및 동기화)
- OnNetworkSpawn
- 네트워크 동작
- OnNetworkDespawn
- OnDisable / OnDestroy

### 사용 시 주의사항

- Awake / Start 단계에서는 권한 정보가 확정되지 않음
- 네트워크 의존 로직은 반드시 OnNetworkSpawn 이후 실행
- Scene에 배치된 NetworkObject와 런타임에 Spawn된 NetworkObject 생명주기 흐름과 초기화 시점이 달라짐
  - 네트워크 의존 로직을 Awake / Start에 두면 잘못된 분기 가능성 존재

### 사용법

- 입력 처리, 권한 분기(IsOwner, IsServer 등)는 OnNetworkSpawn 이후에 수행
- 네트워크 상태에 따라 활성/비활성화가 필요한 로직은 Spawn/Despawn 시점에 분리
- 이벤트 구독은 OnNetworkSpawn에서 수행하고, 해제는 OnNetworkDespawn에서 처리

### 이슈 및 해결

- Awake에서 `GetComponent<T>` 호출 시 NullException 발생
  - 해결
    - 초기화 및 이벤트 구독을 `OnNetworkSpawn()`에서 수행
- `OnNetworkSpawn()`에서 이벤트가 중복 실행됨
  - 해결
    - 각 인스턴스에서 호출되므로 `IsOwner` 기준으로 분기 처리

---

## NetworkObject

### 공식 문서 설명

- Netcode for GameObjects에서 네트워크 동기화의 최소 단위
- 모든 NetworkBehaviour는 반드시 NetworkObject에 부착되어야 함
- 네트워크 상에서의 식별자(NetworkObjectId), 소유권(Owner), Spawn/Despawn 상태를 관리

### 핵심 개념

- NetworkObject는 “이 오브젝트가 네트워크에 존재하는가”를 결정하는 기준점
- Spawn되지 않은 NetworkObject는 네트워크 동기화 대상이 아님
- Owner는 해당 오브젝트의 입력·RPC·Authority 판단의 기준이 됨

### 주요 API / 속성

- `Spawn()` : 서버 또는 Authority에서 오브젝트를 네트워크에 등록
- `Despawn()` : 네트워크에서 오브젝트 제거
- `ChangeOwnership(ulong clientId)` : Owner 변경
- `IsSpawned` : 현재 네트워크에 Spawn 되었는지 여부
- `OwnerClientId` : 오브젝트의 Owner 클라이언트 ID

---

## NetworkTransform

### 공식 문서 설명

- Transform(Position, Rotation, Scale)을 네트워크를 통해 자동 동기화하는 컴포넌트
- Authority 기준으로 위치·회전 값이 결정되며, 비권한 인스턴스는 수신만 수행

### 동작 원리

- Authority 인스턴스에서 Transform 변경
- 변경 값이 네트워크를 통해 비권한 인스턴스로 전파
- 비권한 인스턴스는 보간(Interpolation)을 통해 시각적 부드러움 유지

### 주요 옵션 개념

- Server Authority / Owner Authority 설정
- Interpolation / Extrapolation
- Position / Rotation / Scale 동기화 선택

---

## NetworkRigidbody

### 공식 문서 설명

- Rigidbody 기반 물리 오브젝트를 네트워크에서 안전하게 동기화하기 위한 컴포넌트
- 비권한(non-authoritative) 인스턴스의 Rigidbody를 자동으로 Kinematic 모드로 설정

### 동작 원리

- Authority를 가진 인스턴스만 물리 시뮬레이션 수행
- 물리 결과(Position/Rotation)만 네트워크로 전파
- 비권한 인스턴스는 로컬 물리 간섭 없이 동기화 결과만 반영

### Use Rigidbody For Motion

- 활성화 시 NetworkTransform이 FixedUpdate에서 PhysX 결과를 기준으로 동기화
- Rigidbody 기반 이동이 표준인 환경에서 사용 권장

### 이슈 및 해결

- 구 형태의 맵에서 다른 오브젝트 이동이 툭툭 끊기듯 보이는 현상 발생
  - 원인
    - `Use Rigidbody For Motion` 옵션을 사용하면 동기화가 FixedUpdate로 적용됨
  - 해결
    - `Use Rigidbody For Motion` 옵션 비활성화
