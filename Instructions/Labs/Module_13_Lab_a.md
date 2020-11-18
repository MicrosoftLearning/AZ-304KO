---
lab:
    title: '13: Azure Logic Apps와 Azure Event Grid의 통합 구현'
    module: '모듈 13: 애플리케이션 아키텍처 디자인'
---

# 랩: Azure Logic Apps와 Azure Event Grid의 통합 구현
# 학생 랩 매뉴얼

## 랩 시나리오
Adatum Corporation은 에이전트 기반 솔루션 및 에이전트 없는 솔루션의 조합을 사용하여 환경의 변경 사항에 대한 가시성을 제공하는 광범위한 온-프레미스 네트워크 모니터링 프레임워크를 보유하고 있습니다. 에이전트 없는 솔루션은 폴링을 사용하여 상태 변경 사항을 확인하기 때문에 상대적으로 비효율적인 경향이 있습니다. 

Adatum이 일부 워크로드를 Azure로 마이그레이션할 준비를 하고 있는 만큼 엔터프라이즈 아키텍처 팀은 이러한 비효율성을 해결하고 클라우드에서 사용할 수 있는 이벤트 기반 아키텍처의 사용을 평가하려고 합니다. 솔루션이나 애플리케이션에서 이벤트를 사용한다는 개념은 팀에 새로운 것이 아닙니다. 사실 그들은 개발자 사이에서 이벤트 중심 프로그래밍의 아이디어를 홍보해 왔습니다. 이벤트 기반 아키텍처의 핵심 원칙 중 하나는 기존 서비스가 서로 가질 수 있는 종속성을 되돌리는 것입니다. Azure는 게시자-구독자 모델을 활용하여 이벤트 라우팅을 지원하는 완전 관리형 서비스인 Event Grid에 의존하여 이 기능을 제공합니다. 핵심적으로 Event Grid는 수많은 소스 및 구독자의 이벤트 라우팅 및 배달을 관리하는 이벤트 라우팅 서비스입니다.

Blob Storage 계정, Azure 리소스 그룹 또는 Azure 구독과 같은 게시자가 이벤트를 만듭니다. 이벤트가 발생하면 Event Grid 서비스가 모든 수신 메시지를 요약하기 위해 관리하는 토픽이라는 엔드포인트에 이벤트가 게시됩니다. 이벤트 게시자는 Azure의 서비스로 제한되지 않습니다. 어디서나 실행할 수 있는 사용자 지정 애플리케이션이나 시스템에서 발생하는 이벤트를 사용할 수 있습니다. 여기에는 Event Grid 서비스에 HTTP 요청을 게시할 수 있는 경우 온-프레미스, 데이터 센터 또는 다른 클라우드에서도 호스팅되는 애플리케이션이 포함됩니다.

이벤트 처리기에는 Functions, Logic Apps 또는 Azure Automation과 같은 서버리스 기술을 비롯한 여러 Azure 서비스가 포함됩니다. 처리기는 이벤트 구독을 만들어 Event Grid에 등록됩니다. 이벤트 처리기 엔드포인트가 공개적으로 액세스할 수 있고 전송 계층 보안에 의해 암호화된 경우 Event Grid에서 메시지를 푸시할 수 있습니다.

다른 많은 Azure 서비스와 달리 프로비전하거나 관리해야 하는 Event Grid 네임스페이스는 없습니다. 기본 Azure 리소스에 대한 토픽은 사용자에게 기본 제공되며 완전히 투명하게 공개되지만, 사용자 지정 토픽은 임시로 프로비전되며 리소스 그룹에 있습니다. 이벤트 구독은 토픽과 단순 연결되어 있습니다. 이 모델은 토픽 관리를 구독으로 단순화하고 Event Grid를 다중 테넌트로 만들어 대규모 확장을 허용합니다.

Azure Event Grid는 언어 또는 플랫폼 기반이 아닙니다. 기본적으로 Azure 서비스와 통합되지만 HTTP 프로토콜을 지원하는 모든 서비스에서 쉽게 활용할 수 있으므로 매우 편리하고 혁신적인 서비스입니다.

이 기능을 탐색하기 위해 Adatum 아키텍처 팀은 Azure Logic Apps와 Event Grid의 통합을 테스트하여 다음을 수행하려고 합니다.

-  지정된 Azure VM의 상태가 변경된 시기를 감지합니다.

-  이벤트에 대한 응답으로 자동으로 이메일 알림을 생성합니다.


## 목표
  
이 랩을 완료하면 다음을 수행할 수 있습니다.

-  Azure Logic Apps와 Event Grid 통합

-  리소스 그룹 내의 리소스 변경을 나타내는 이벤트에 대한 응답으로 Logic Apps 실행 트리거


## 랩 환경
  
Windows 서버 관리자 자격 증명

-  사용자 이름: **Student**

-  암호: **Pa55w.rd1234**

예상 시간: 60분


## 랩 파일

-  \\\\AZ303\\AllFiles\\Labs\\04\\azuredeploy30304suba.json

-  \\\\AZ303\\AllFiles\\Labs\\04\\azuredeploy30304rga.json

-  \\\\AZ303\\AllFiles\\Labs\\04\\azuredeploy30304rga.parameters.json

## 지침

### 연습 0: 랩 환경 준비

이 연습의 주요 작업은 다음과 같습니다.

 1. Azure Resource Manager 템플릿을 사용하여 Azure VM 배포


#### 작업 1: Azure Resource Manager 템플릿을 사용하여 Azure VM 배포

1. 랩 컴퓨터에서 웹 브라우저를 시작하여 [Azure Portal](https://portal.azure.com)로 이동하고, 이 랩에서 사용할 구독에서 Owner 역할을 가진 사용자 계정의 자격 증명을 제공하여 로그인합니다.

1. Azure Portal에서 검색 텍스트 상자의 오른쪽에 있는 도구 모음 아이콘을 직접 선택하여 **Cloud Shell** 창을 엽니다.

1. **Bash** 또는 **PowerShell**을 선택하라는 메시지가 표시되면 **PowerShell**을 선택합니다. 

    >**참고**: **Cloud Shell**을 처음 시작하고 **탑재된 스토리지가 없음** 메시지를 받으면, 이 랩에서 사용하는 구독을 선택하고 **스토리지 만들기**를 선택합니다. 

1. Cloud Shell 창에서 다음을 실행하여 구독에 **Microsoft.EventGrid** 공급자를 등록합니다.

   ```powershell
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.EventGrid'
   ```

1. Cloud Shell 창의 도구 모음에서 **파일 업로드/다운로드** 아이콘을 선택한 다음, 드롭다운 메뉴에서 **업로드**를 선택하여 **\\\\AZ303\\AllFiles\Labs\\04\\azuredeploy30304suba.json** 파일을 Cloud Shell 홈 디렉터리에 업로드합니다.

1. Cloud Shell 창에서 다음을 실행하여 리소스 그룹을 만듭니다(`<Azure region>` 자리 표시자를 구독에 Azure VM을 배포할 수 있고 랩 컴퓨터 위치에 가장 가까운 Azure 지역의 이름으로 대체).

   ```powershell
   $location = '<Azure region>'
   New-AzSubscriptionDeployment `
     -Location $location `
     -Name az30304subaDeployment `
     -TemplateFile $HOME/azuredeploy30304suba.json `
     -rgLocation $location `
     -rgName 'az30304a-labRG'
   ```

      > **참고**: Azure VM을 프로비전할 수 있는 Azure 지역을 확인하려면 [**https://azure.microsoft.com/ko-kr/regions/offers/**](https://azure.microsoft.com/ko-kr/regions/offers/)를 참고하세요.

1. Cloud Shell 창에서 Azure Resource Manager 템플릿 **\\\\AZ303\\AllFiles\Labs\\04\\azuredeploy30304rga.json**을 업로드합니다.

1. Cloud Shell 창에서 Azure Resource Manager 매개 변수 파일 **\\\\AZ303\\AllFilesLabs\\04\\azuredeploy30304rga.parameters.json** 을 업로드합니다.

1. Cloud Shell 창에서 다음을 실행하여 이 랩에서 사용할 Windows 서버 2019를 실행하는 Azure VM을 배포합니다.

   ```powershell
   New-AzResourceGroupDeployment `
     -Name az30304rgaDeployment `
     -ResourceGroupName 'az30304a-labRG' `
     -TemplateFile $HOME/azuredeploy30304rga.json `
     -TemplateParameterFile $HOME/azuredeploy30304rga.parameters.json `
     -AsJob
   ```

    > **참고**: 배포가 완료될 때까지 기다리지 말고 다음 연습을 진행하세요. 배포는 5분 미만이 소요됩니다.

1. Azure Portal에서 **Cloud Shell** 창을 닫습니다. 


### 연습 1: Azure 논리 앱에 대한 인증 및 권한 부여 구성

1. Azure Active Directory 서비스 주체 만들기

1. Azure AD 서비스 주체에 Reader 역할 할당 


#### 작업 1: Azure Active Directory 서비스 주체 만들기

1. Azure Portal 에서 **Cloud Shell**내의 **PowerShell** 세션을 시작합니다. 

1. Cloud Shell 창에서 다음을 실행하여 이 작업의 후속 단계에서 만드는 서비스 주체와 연결할 새 Azure AD 애플리케이션을 만듭니다.

   ```powershell
   $password = 'Pa55w.rd1234.@z304'
   $securePassword = ConvertTo-SecureString -Force -AsPlainText -String $password
   $az30304aadapp = New-AzADApplication -DisplayName 'az30304aadsp' -HomePage 'http://az30304aadsp' -IdentifierUris 'http://az30304aadsp' -Password $securePassword
   ```

1. Cloud Shell 창에서 다음을 실행하여 이전 단계에서 만든 애플리케이션과 연결된 새 Azure AD 서비스 주체를 만듭니다.

   ```powershell
   New-AzADServicePrincipal -ApplicationId $az30304aadapp.ApplicationId.Guid
   ```

1. **New-AzADServicePrincipal** 명령의 출력에서 **ApplicationId** 속성의 값을 기록합니다. 이 연습의 뒷부분에서 필요할 것입니다.

1. Cloud Shell 창에서 다음을 실행하여 현재 Azure 구독의 **ID** 속성 값과 해당 구독과 연결된 Azure AD 테넌트의 **TenantId** 속성 값을 식별합니다(이 랩의 다음 실습에도 필요합니다).

   ```powershell
   Get-AzSubscription
   ```

1. Cloud Shell 창을 닫습니다.


#### 작업 2: Azure AD 서비스 주체에 대한 액세스 권한 부여 

1. Azure Portal에서 **리소스 그룹**을 검색 및 선택하고, **리소스 그룹** 블레이드에서 **az30304a-labRG**을 선택합니다.

1. **az30304a-labRG** 블레이드에서 **액세스 제어(IAM)** 를 선택합니다.

1. **az30304a-labRG | 액세스 제어(IAM)** 블레이드에서 **+ 추가**를 선택하고, **역할 할당 추가**를 선택합니다. 

1. **역할 할당 추가** 블레이드에서 다음 설정을 지정하고 **저장**을 클릭합니다.

    | 설정 | 값 | 
    | --- | --- |
    | 역할 | **읽기 권한자** |
    | 액세스 권한을 할당할 대상 | **Azure AD 사용자, 그룹 또는 서비스 주체** |
    | 선택 | **az30304aadsp** |


### 연습 2: Azure 논리 앱 구현
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure 논리 앱 만들기

1. Azure 논리 앱에 트리거 추가

1. Azure 논리 앱에 조건 추가

1. Azure 논리 앱에 작업 추가


#### 작업 1: Azure 논리 앱 만들기

1. Azure Portal에서 **논리 앱**을 검색 및 선택하고, **논리 앱** 블레이드에서 **+ 추가**를 선택합니다.

1. **논리 앱** 블레이드의 **기본** 탭에서 다음 설정을 지정합니다(다른 설정은 기본값으로 유지).

    | 설정 | 값 | 
    | --- | --- |
    | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
    | 리소스 그룹 | 새 리소스 그룹 **az30304b-labRG**의 이름 |
    | 논리 앱 이름 | **az30304b-logicapp1** | 
    | 위치 선택 | **지역** |
    | 위치 | 이전 실습에서 선택한 Azure 지역의 이름 |
    | Log Analytics | **꺼짐** |

1. **검토 + 만들기**를 선하고 **만들기**를 클릭합니다. 

    >**참고**: 논리 앱이 생성될 때까지 기다립니다. 프로비전은 약 2분이 소요됩니다. 


#### 작업 2: Azure 논리 앱에 트리거 추가

1. Azure Portal에서 **논리 앱**을 검색 및 선택하고 **논리 앱** 블레이드에서 **az30304b-logicapp1**을 선택합니다.

1. **논리 앱 디자이너** 블레이드에서 **빈 논리 앱**을 선택합니다. 이렇게 하면 빈 디자이너 작업 영역이 표시됩니다.

1. **커넥터 및 트리거 검색** 텍스트 상자를 사용하여 **Event Grid**를 찾고 결과 목록의 **트리거** 열에서 **리소스 이벤트가 발생할 때**를 선택하여 Azure Event Grid 트리거를 디자이너 작업 영역에 추가합니다.

1. **Azure Event Grid** 타일에서 **서비스 주체로 연결** 링크를 선택하고 다음 설정을 지정한 다음 **만들기**를 선택합니다.

    | 설정 | 값 | 
    | --- | --- |
    | 연결 이름 | **az30304egconnection** |
    | Client ID | 이 실습에서 이전에 식별한 **ApplicationId** 속성의 값 |
    | 클라이언트 암호 | **Pa55w.rd1234.@z304** |
    | 테넌트 | 이 실습에서 이전에 식별한 **TenantId** 속성의 값 |

1. **리소스 이벤트가 발생할 때** 타일에서 다음 설정을 지정합니다.

    | 설정 | 값 | 
    | --- | --- |
    | 구독 | 이 실습에서 이전에 식별한 구독 **Id** 속성의 값 |
    | 리소스 유형 | **Microsoft.Resources.resourceGroups** |
    | 리소스 이름 | **az30304a-LabRG** |
    | 이벤트 유형 항목 - 1 | **Microsoft.Resources.ResourceWriteSuccess** |
    | 이벤트 유형 항목 - 2 | **Microsoft.Resources.ResourceDeleteSuccess** |

1. **리소스 이벤트가 발생할 때** 타일에서 **새 매개 변수 추가**를 선택하고, **구독 이름**을 선택합니다.

1. **구독 이름** 텍스트 상자에서 **event-subscription-az30304b**를 입력하고 **저장**을 선택합니다.


#### 작업 3: Azure 논리 앱에 조건 추가

1. Azure Portal의 새로 프로비전된 Azure 논리 앱의 논리 앱 디자이너 블레이드에서 **+ 새 단계**를 선택합니다. 

1. 작업 타일 선택에서 **커넥터 및 트리거 검색** 텍스트 상자를 이용하여 **조건**을 검색하고, 결과 목록의 **작업** 열에서 **조건**을 선택하여 디자이너 작업 영역에 추가합니다.

1. **조건** 타일의 오른쪽 상단 모서리에 있는 줄임표 기호를 선택한 다음 팝업 메뉴에서 **이름 바꾸기**를 선택하고, **조건**을 **리소스 그룹의 가상 머신이 변경된 경우** 텍스트로 대체합니다. 

1. 조건의 왼쪽에 있는 **값 선택** 텍스트 상자를 선택하고, 팝업 창의 **식** 탭에서 다음 식을 입력하고 **OK**를 선택합니다.

   ```
   triggerBody()?['data']['operationName']
   ```

1. 조건의 중간 요소에서 **다음과 같음**이 나타나는지 확인하고, 오른쪽의 **값 선택** 텍스트 상자에 모니터링하려는 작업을 나타내는 값을 입력하세요.

   ```
   Microsoft.Compute/virtualMachines/write
   ```

1. **논리 앱 디자이너** 블레이드에서 **저장**을 선택합니다. 


#### 작업 4: Azure 논리 앱에 작업 추가

1. Azure Portal 내 새로 프로비전된 Azure 논리 앱의 논리 앱 디자이너 블레이드의 **참** 타일에서 **작업 추가**을 선택합니다. 

1. **작업 선택** 창의 **커넥터 및 작업 검색** 텍스트 상자에서 **Outlook**을 입력합니다.

1. 결과 목록에서 **Outlook.com**을 클릭합니다. 

1. **Outlook.com** 작업 목록에서 **이메일 보내기(V2)** 를 클릭합니다.

1. **Outlook.com** 창에서 **로그인**을 선택합니다. 

1. 메시지가 표시되면 이 랩에서 사용 중인 Microsoft 계정을 사용하여 인증합니다. 

1. Outlook 리소스에 액세스할 수 있는 Azure Logic App 권한을 부여하는 데 동의하라는 메시지가 표시되면 **예**를 선택합니다.

1. **Outlook.com** 창에서 **이메일 보내기(V2)** 타일의 오른쪽 상단 모서리에 있는 생략 부호를 선택하고, 팝업 메뉴에서 **이름 바꾸기**를 선택한 다음, **이메일 보내기(v2)** 를 텍스트 **이메일 보내기**로 교체합니다. 

1. **이메일 보내기** 창에서 다음 설정을 지정하고 **저장**을 선택합니다.

    | 설정 | 값 | 
    | --- | --- |
    | 대상 | Microsoft 계정의 기본 전자 메일 주소 |
    | 제목 | 제목: **리소스 업데이트:** 를 입력하고 **전자 메일 보내기** 창 오른쪽에 있는 **동적 콘텐츠** 열에서 **제목**을 선택합니다. |
    | 본문 | **리소스 그룹 유형:** 을 입력하고 **전자 메일 보내기** 창의 오른쪽에 있는 **동적 콘텐츠** 열 아래의 검색 텍스트 상자에 **토픽**을 입력 및 선택한 다음, **본문** 텍스트 상자로 돌아가서 새 줄에 **이벤트 유형:** 을 입력하고 **전자 메일 보내기** 창의 오른쪽에 있는 **동적 콘텐츠** 창 아래의 검색 텍스트 상자에 **이벤트 유형**을 입력 및 선택합니다. **본문** 텍스트 상자로 돌아가서 새 줄에 **이벤트 ID:** 를 입력하고 **전자 메일 보내기** 창의 오른쪽에 있는 **동적 콘텐츠** 열 아래의 검색 텍스트 상자에 **ID**를 입력 및 선택한 후, **본문** 텍스트 상자로 돌아가서 새 줄에 **이벤트 시간:** 을 입력하고 **전자 메일 보내기** 창의 오른쪽에 있는 **동적 콘텐츠** 열 아래의 검색 텍스트 상자에 **이벤트 시간**을 입력 및 선택합니다. |

    **사서함에 들어가 전화 번호를 입력하여 계정을 확인해야 할 수 있습니다.**

1. **로직 앱 디자이너** 블레이드에서 **저장**을 선택합니다. 


### 연습 3: 이벤트 구독 구현
  
이 연습의 주요 작업은 다음과 같습니다.

1. 이벤트 구독 구성

1. Azure 논리 앱의 기능 검토

1. 랩에 배포된 Azure 리소스 제거


#### 작업 1: 이벤트 구독 구성

1. Azure Portal에서 **az30304b-logicapp1** 블레이드로 이동한 후 **요약** 섹션에서 **트리거 기록 보기**를 선택합니다. 

1. **When_a_resource_event_occurs** 블레이드에서 **콜백 url [POST]** 텍스트 상자의 값을 복사합니다.

1. Azure Portal에서 **az30304a-LabRG** 리소스 그룹으로 이동한 다음 세로 메뉴에서 **이벤트**를 선택합니다.

1. **az30304a-LabRG**에서 **| 이벤트** 블레이드에서 **+ 이벤트 구독**을 선택합니다.

1. **이벤트 구독 만들기** 블레이드에서 다음 설정을 지정하고 **만들기**를 선택합니다.

    | 설정 | 값 | 
    | --- | --- |
    | 이름 | **event-subscription-az30304a-LabRG** |
    | 이벤트 스키마 | **Event Grid 스키마** |
    | 시스템 토픽 이름 | **event-topic-az30304a-LabRG** |
    | 이벤트 유형 필터링 | **리소스 쓰기 성공**, **리소스 삭제 성공**, **리소스 작업 성공** |
    | 엔드포인트 유형 | **웹 후크** | 
    | 엔트포인트 | 이 작업의 시작 부분에서 복사한 URL 문자열 |

1. **만들기**를 선택합니다.


#### 작업 2: Azure 논리 앱의 기능 검토

1. Azure Portal에서 **az30304a-labRG** 리소스 그룹으로 이동하여 리소스 목록에서 **az30304a-vm0** Azure VM을 나타내는 항목을 선택합니다.

1. **az30304a-vm0** 블레이드의 **설정** 섹션에서 **크기**를 선택합니다.

1. **az30304a-vm0 | 크기** 블레이드에서, 현재 설정된 크기와 다른 크기를 선택하고 **크기 조정**을 선택한 후 크기 조정 작업이 완료되었는지 확인합니다. 

1. **az30304b-logicapp1** 블레이드로 다시 이동한 후  **새로 고침**을 선택합니다. **실행 기록**에 Azure VM 상태 변경에 해당하는 항목이 포함됩니다.

1. **실행 기록** 목록에서 기간이 가장 긴 항목을 선택합니다. 이 항목은 Azure VM에서 크기 조정이 수행되었음을 나타냅니다.

1. **논리 앱 실행** 블레이드에서 논리 앱 실행 워크플로를 나타내는 다이어그램을 검토합니다.

1. **논리 앱 실행** 블레이드에서 **리소스 이벤트가 발생할 때** 사각형을 선택하여 확장하고 **출력**섹션에서 **원시 출력 표시**를 선택합니다.

1. **출력** 블레이드에서 이벤트의 세부 정보 및 사용자 계정의 ID와 Azure VM 크기를 조정하려는 요청이 시작된 IP 주소와 같은 세부 정보가 포함된 참고 사항을 검토합니다. 


1. 이전 연습에서 지정한 이메일 계정의 받은 편지함으로 이동하여 논리 앱으로 생성한 이메일이 포함되어 있는지 확인합니다.


#### 작업 3: 랩에 배포된 Azure 리소스 제거

1. Cloud Shell 창에서 다음을 실행하여 이 연습에서 만든 리소스 그룹을 나열합니다.

   ```powershell
   Get-AzResourceGroup -Name 'az30304*'
   ```

    > **참고**: 이 랩에서 만든 리소스 그룹만 출력에 포함되어 있는지 확인합니다. 이 작업에서는 이러한 그룹을 삭제할 것입니다.

1. Cloud Shell 창에서 다음을 실행하여 이 랩에서 만든 리소스 그룹을 삭제합니다.

   ```powershell
   Get-AzResourceGroup -Name 'az30304*' | Remove-AzResourceGroup -Force -AsJob
   ```

1. Cloud Shell 창을 닫습니다.
