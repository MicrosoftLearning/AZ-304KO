---
lab:
    title: '4: Azure AD 인증 및 권한 부여 관리'
    module: '모듈 4: 인증 및 권한 부여 디자인'
---

# 랩: Azure AD 인증 및 권한 부여 관리
# 학생 랩 매뉴얼

## 랩 시나리오

Adatum Corporation은 Azure로 마이그레이션하는 것의 일환으로 ID 전략을 정의해야 합니다. Adatum은 adatum.com라는 단일 도메인 Active Directory 포리스트를 가지고 있으며 그에 대해 공개적으로 등록된 DNS 도메인을 소유하고 있습니다. Adatum 엔터프라이즈 아키텍처 팀은 일부 온-프레미스 워크로드를 Azure로 전환하는 옵션을 탐색하고 있으므로 AD DS(Active Directory Domain Services) 환경과 대상 Azure 구독과 연결된 Azure Active Directory(Azure AD) 테넌트 간의 통합을 장기적인 인증 및 권한 부여 모델의 핵심 구성 요소로 평가할 계획입니다.

새 모델은 Azure AD의 다단계 인증 기능을 활용하는 애플리케이션 단계별 인증과 함께 단일 로그인을 용이하게 해야 합니다. 단일 로그인을 구현하기 위해 아키텍처 팀은 Azure AD Connect를 배포하고 암호 해시 동기화를 위해 구성하여 두 ID 저장소의 사용자 개체를 일치시킬 계획입니다. 최적의 인증 방법을 선택하는 일은 클라우드로 이동하려는 조직의 첫 번째 관심사입니다. Azure AD 암호 해시 동기화는 Azure AD 통합 리소스에 액세스할 때 온-프레미스 사용자를 위한 단일 로그인 인증을 구현하는 가장 간단한 방법입니다. 이 메서드는 ID 보호와 같은 일부 프리미엄 Azure AD 기능에서도 필요합니다.

단계별 인증을 구현하기 위해 Adatum 엔터프라이즈 아키텍처 팀은 Azure AD 조건부 액세스 정책을 활용할 계획입니다. 조건부 액세스 정책은 액세스 중인 애플리케이션 또는 리소스 유형에 따라 다단계 인증의 적용을 지원합니다. 조건부 액세스 정책은 1단계 인증이 완료된 후 적용됩니다. 조건부 액세스는 다음을 포함한 다양한 요소를 기반으로 할 수 있습니다.

- 사용자 또는 그룹 구성원. 정책은 특정 사용자 및 그룹을 대상으로 지정하여 관리자에게 액세스에 대한 세부적인 제어를 제공할 수 있습니다.
- IP 위치 정보. 조직은 신뢰할 수 있는 IP 주소 범위를 만들 수 있으며 정책 결정을 내릴 때 이를 사용할 수 있습니다. 관리자는 트래픽을 차단하거나 허용하기 위해 전체 국가/지역의 IP 범위를 지정할 수 있습니다.
- 디바이스. 조건부 액세스 정책을 시행할 때 특정 플랫폼의 디바이스가 있거나 특정 상태로 표시된 사용자를 사용할 수 있습니다.
- 애플리케이션. 특정 애플리케이션에 액세스하려는 사용자는 다양한 조건부 액세스 정책을 트리거할 수 있습니다.
- 실시간 및 계산된 위험 탐지. Azure AD ID 보호와의 신호 통합을 통해 조건부 액세스 정책이 위험한 로그인 동작을 식별할 수 있습니다. 그런 다음 정책은 사용자가 암호 변경 또는 다단계 인증을 수행하여 위험 수준을 줄이거나 관리자가 수동 작업을 수행할 때까지 액세스 권한을 차단하도록 강제할 수 있습니다.
- MCAS(Microsoft Cloud App Security). 사용자 애플리케이션 액세스 및 세션을 실시간으로 모니터링하고 제어할 수 있으므로 클라우드 환경에서 수행되는 액세스 및 활동에 대한 가시성과 제어가 향상됩니다.

이러한 목표를 달성하기 위해 Adatum 엔터프라이즈 아키텍처 팀은 Azure AD(Azure Active Directory) 테넌트와 AD DS(Active Directory Domain Services) 포리스트의 통합을 테스트하고 파일럿 사용자의 조건부 액세스 기능을 평가할 계획입니다.

## 목표
  
이 랩을 완료하면 다음을 수행할 수 있습니다.

 - AD DS 도메인 컨트롤러를 호스트하는 Azure VM 배포

 - Azure AD 테넌트 만들기 및 구성.

 - Azure AD 포리스트와 Azure AD 테넌트 통합


## 랩 환경
  
Windows 서버 관리자 자격 증명

-  사용자 이름: **Student**

-  암호: **Pa55w.rd1234**

예상 시간: 120분


## 랩 파일

-  \\\\AZ304\\AllFiles\\Labs\\10\\azuredeploy30410suba.json


## 지침

### 연습 0: 랩 환경 준비

이 연습의 주요 작업은 다음과 같습니다.

1. Azure VM 배포에 사용할 수 있는 DNS 이름 식별

1. Azure Resource Manager 빠른 시작 템플릿을 사용하여 Active Directory 도메인 컨트롤러를 호스트하는 Azure VM 배포


#### 작업 1: Azure VM 배포에 사용할 수 있는 DNS 이름 식별

1. 랩 컴퓨터에서 웹 브라우저를 시작하여 [Azure Portal](https://portal.azure.com)로 이동하고, 이 랩에서 사용할 구독에서 Owner 역할을 가진 사용자 계정의 자격 증명을 제공하여 로그인합니다.

1. Azure Portal에서 검색 텍스트 상자의 오른쪽에 있는 도구 모음 아이콘을 직접 선택하여 **Cloud Shell** 창을 엽니다.

1. **Bash** 또는 **PowerShell**을 선택하라는 메시지가 표시되면 **PowerShell**을 선택합니다. 

    >**참고**: **Cloud Shell**을 처음 시작하고 **탑재된 스토리지가 없음** 메시지를 받으면, 이 랩에서 사용하는 구독을 선택하고 **스토리지 만들기**를 선택합니다. 

1. Cloud Shell 창에서 다음 명령을 실행하여 다음 작업에서 제공해야 할 사용 가능한 DNS 이름을 식별합니다. 또한 자리 표시자 `<custom-label>`를 전역적으로 고유한 유효 DNS 호스트 이름으로 대체하며 자리 표시자 `<Azure region>`을 Active Directory 도메인 컨트롤러를 호스트할 Azure VM을 배포하고자 하는 Azure 지역의 이름으로 변경합니다.

    ```powershell
    Test-AzDnsAvailability -DomainNameLabel <custom-label> -Location '<location>'
    ```
      > **참고**: Azure VM을 프로비전할 수 있는 Azure 지역을 식별하려면 [https://azure.microsoft.com/ko-kr/regions/offers/](https://azure.microsoft.com/ko-kr/regions/offers/)를 참고하세요. **Powershell cmdlet**을 사용하여 지역의 목록을 가져올 수도 있습니다
      ```powershell
      Get-AzLocation | FT
      ```

1. 이 명령이 **True**를 반환하는지 확인합니다. True가 반환되지 않으면 **True**가 반환될 때까지 `<custom-label>`의 다른 값을 사용해 같은 명령을 다시 실행합니다.

1. 성공적인 결과가 반환된 `<custom-label>`의 값을 기록합니다. 이 값은 다음 작업에 필요합니다.


#### 작업 2: Azure Resource Manager 빠른 시작 템플릿을 사용하여 AD DS 도메인 컨트롤러를 호스트하는 Azure VM 배포

1. Azure Portal의 Cloud Shell 창의 도구 모음에서 **파일 업로드/다운로드** 아이콘을 선택하고 드롭다운 메뉴에서 **업로드**를 선택하고 파일 **\\\\AZ304\\AllFiles\Labs\\10\\azuredeploy30410suba.json** 을 Cloud Shell 홈 디렉터리에 업로드합니다.

1. Cloud Shell 창에서 다음 명령을 실행하여 리소스 그룹을 만듭니다. 여기서 `<Azure region>` 자리 표시자는 이전 작업에서 지정한 Azure 지역의 이름으로 바꿉니다.

   ```powershell
   $location = '<Azure region>'
   ```
   ```powershell
   New-AzSubscriptionDeployment `
     -Location $location `
     -Name az30410subaDeployment `
     -TemplateFile $HOME/azuredeploy30410suba.json `
     -rgLocation $location `
     -rgName 'az30410a-labRG'
   ```

1. Azure Portal에서 **Cloud Shell** 창을 닫습니다.

1. 랩 컴퓨터에서 다른 브라우저 탭을 열고 [https://github.com/Azure/azure-quickstart-templates/tree/master/active-directory-new-domain](https://github.com/Azure/azure-quickstart-templates/tree/master/active-directory-new-domain)로 이동합니다. 

1. **Create a new Windows VM and create a new AD Forest, Domain and DC** 페이지에서 **Deploy to Azure**를 선택합니다. 이렇게 하면 Azure Portal의 **Create an Azure VM with a new AD Forest** 블레이드로 브라우저가 자동으로 리디렉션됩니다.

1. **Create an Azure VM with a new AD Forest** 블레이드에서 **Edit parameters**을 선택합니다.

1. **매개 변수 편집** 블레이드에서 **파일 로드**를 선택하고 **\\\\AZ303\\AllFiles\Labs\\10\\azuredeploy30310rga.parameters.json**을 선택하고 **열기** 를 선택한 다음 **저장**을 선택합니다. 

1. **Create an Azure VM with a new AD Forest** 블레이드에서 다음 설정을 지정합니다(기존 값은 유지).

    | 설정 | 값 | 
    | --- | --- |
    | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
    | 리소스 그룹 | **az30410a-LabRG** |
    | DNS 접두사 | 이전 작업에서 확인한 DNS 호스트 이름| 

1. **Create an Azure VM with a new AD Forest** 블레이드에서 **검토+만들기** 확인란을 선택하고 다음 화면에서 **만들기**를 선택합니다.

    > **참고**: 배포가 완료될 때까지 기다리지 말고 다음 연습을 진행하세요. 배포에는 약 15분이 소요될 수 있습니다. 이 랩의 세 번째 연습에서이 작업에 배포된 가상 머신을 사용합니다.


### 연습 1: Azure AD 테넌트 만들기 및 구성.
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure AD 테넌트 만들기

1. Azure AD 사용자 생성 및 구성

1. Azure AD Premium P2 라이선스 정품 인증 및 할당


#### 작업 1: Azure AD 테넌트 만들기

1. Azure Portal에서 **Azure Active Directory**를 검색 및 선택하고 Azure Active Directory 블레이드에서 **+ 테넌트 만들기**를 선택합니다.

1. **디렉터리 만들기** 블레이드의 **기본** 탭에서 **Azure Active Directory** 옵션을 선택하고 **다음 : 구성 >** 을 선택합니다.

1. **디렉터리 만들기** 블레이드의 **구성** 탭에서 다음 설정을 지정합니다(다른 설정을 기존 값으로 남겨둡니다).

    | 설정 | 값 |
    | --- | --- |
    | 조직 이름: | **Adatum Lab** |
    | 초기 도메인 이름 | 소문자와 숫자로 구성된 유효하며 문자로 시작하는 DNS 이름 | 
    | 국가/지역 | **미국** |

   > **참고**: **초기 도메인 이름** 텍스트 상자의 녹색 확인 표시는 입력한 도메인 이름이 유효하고 고유한지 여부를 나타냅니다.

1. **다음: 검토 + 만들기를** 선택하고 **만들기**를 선택합니다.

1. Azure Potal을 표시하는 브라우저 페이지를 새로 고치고 **Azure Active Directory**를 검색 및 선택하고 Azure Active Directory 블레이드에서 **테넌트 전환**을 선택합니다.

1. **디렉터리 + 구독** 블레이드의 **Adatum Lab** 카드에서 **전환을** 클릭합니다.


#### 작업 2: Azure AD 사용자 생성 및 구성

1. **Adatum Lab | 개요** 섹션의 Azure Active Directory 블레이드 **관리**아래 **사용자**선택합니다. **사용자 | 모든 사용자** 블레이드에서 로그인한 사용자 계정을 선택하여 **프로필** 설정을 표시합니다. 

1. 사용자 계정의 프로필 블레이드의 **설정** 섹션에서 상단 메뉴에서 **편집**을 선택하고 **사용량 위치**를 **United States**으로 설정하고 변경 사항을 저장합니다.

    >**참고**: 이것은 이 랩의 후반부에서 사용자 계정에 Azure AD Premium P2 라이선스를 할당하려면 필요합니다.

1. **사용자|모든 사용자(미리 보기)** 블레이드로 다시 이동한 다음, 상단 메뉴에서 **+ 새 사용자**를 선택합니다.

1. **새 사용자** 블레이드에서 다음 설정을 지정합니다(다른 설정을 기본값으로 남겨둡니다).

    | 설정 | 값 |
    | --- | --- |
    | 사용자 이름 | **az30410-aaduser1** |
    | 이름 | **az30410-aaduser1** |
    | 암호 자동 생성 | 선택 |
    | 암호 표시 | 체크 |
    | 역할 | **전역 관리자('전역 관리자'로 검색해서 선택)** |
    | 사용 위치 | **미국** |

    >**참고**: 전체 사용자 이름(도메인 이름 포함)과 자동 생성된 암호를 기록합니다. 이 작업의 후반에서 필요할 것입니다.

1. **새 사용자** 블레이드에서 **만들기**를 선택합니다.

1. 랩 컴퓨터에서는 Microsoft 엣지브라우저의 **InPrivate** 브라우저 창을 열고 새로 만든 **az30410-aaduser1** 사용자 계정을 사용하여 [Azure Portal](https://portal.azure.com)에 로그인합니다. 암호를 업데이트하라는 메시지가 표시되면 암호를 **Pa55w.rd1234**로 변경합니다. 

1. Azure Portal에서 **az30410-aaduser1** 사용자로 로그아웃하고 InPrivate 브라우저 창을 닫습니다.


#### 작업 3: Azure AD Premium P2 라이선스 정품 인증 및 할당

1. Azure Portal을 표시하는 브라우저 창으로 돌아가 **Adatum Lab** Azure AD 테넌트의 **개요** 블레이드로 이동한 다음 **관리** 섹션에서 **라이선스**를 선택합니다.

1. **라이선스 | 개요** 블레이드에서 **모든 제품**을 선택하고 **+ 사용/구매**를 선택합니다

1. **정품 인증** 블레이드의 **Azure AD Premium P2** 섹션에서 **무료 평가판**을 선택한 다음 **활성화**를 선택합니다. 

1. **라이선스 | 모든 제품** 블레이드를 표시하는 브라우저 창을 새로 고치고 정품 인증이 성공했는지 확인합니다. 

1. **라이선스 - 모든 제품** 블레이드에서 **Azure Active Directory Premium P2** 항목을 선택합니다. 

1. **Azure Active Directory Premium P2 | 허가된 사용자** 블레이드에서 **+ 할당**을 선택합니다. 

1. **라이선스 할당** 블레이드에서 **사용자**를 선택하고 **사용자** 블레이드에서 로그인한 계정과 **az30410-aaduser1** 사용자 계정을 모두 선택합니다.

1. **라이선스 할당** 블레이드로 돌아가서 **할당 옵션**을 선택하고 **라이선스 옵션** 블레이드에 나열된 옵션을 검토한 후 **확인**을 선택합니다.

1. **라이선스 할당** 블레이드에서 **할당**을 선택합니다. 


### 연습 2: Azure AD 포리스트와 Azure AD 테넌트 통합
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure AD 테넌트에 사용자 지정 도메인 이름 할당

1. Azure VM에서 AD DS 구성

1. Azure AD Connect 설치

1. 동기화된 사용자 계정의 속성 구성


#### 작업 1: Azure AD 테넌트에 사용자 지정 도메인 이름 할당

1. Azure Portal에서 **Azure Active Directory Adatum Lab | 개요** 블레이드로 이동합니다.

1. **Adatum Lab | 개요** 블레이드에서 **사용자 지정 도메인 이름**을 선택합니다.

1. **Adatum Lab | 사용자 지정 도메인 이름** 블레이드에서 Azure AD 테넌트와 연결된 기본 DNS 도메인 이름을 식별합니다. 

    >**참고**: Azure AD 테넌트의 기본 DNS 이름 값을 기록합니다. 이 값은 다음 작업에 필요합니다.

1. **Adatum Lab | 사용자 지정 도메인 이름** 블레이드에서 **+ 사용자 지정 도메인 추가**를 선택합니다.

1. **사용자 지정 도메인 이름** 블레이드의 **사용자 지정 도메인 이름** 텍스트 상자에 **adatum.com**을 입력하고 **도메인 추가**를 선택합니다. 

1. **adatum.com** 블레이드에서 Azure AD 도메인 이름 확인을 수행하는 데 필요한 정보를 검토하고 블레이드를 닫습니다.

    > **참고**: **adatum.com** DNS 도메인 이름이 없으므로 유효성 검사 프로세스를 완료할 수 없습니다. 따라서 **adatum.com** Active Directory 도메인을 Azure AD 테넌트와 동기화할 수 *없습니다*. 이를 위해 이 작업의 앞에서 식별한 Azure AD 테넌트의 기본 DNS 이름(**onmicrosoft.com** 접미사로 끝나는 이름 예:hahaysh.onmicrosoft.com)을 사용합니다. 그러나 결과적으로 Active Directory 도메인의 DNS 도메인 이름과 Azure AD 테넌트의 DNS 이름이 다를 수 있다는 점을 기억해 두세요. 이 경우, Adatum 사용자는 Active Directory 도메인에 로그인할 때와 Azure AD 테넌트에 로그인할 때 각기 다른 이름을 사용해야 합니다.


#### 작업 2: Azure VM에서 AD DS 구성

> **참고**: 이 연습을 시작하기 전에 랩 시작 시 했었던 Azure VM의 배포가 완료되었는지 확인합니다.

1. Azure Portal에서 **"가상 머신"** 을 검색하고 선택한 다음 **"가상 머신"** 블레이드에서 **az30410a-vm1**를 선택합니다.

1. **az30410a-vm1** 블레이드의 드롭다운 메뉴에서 **연결을** 선택합니다. **az30410a-vm1 | 연결** 블레이드의 **RDP** 탭 아래 드롭다운 메뉴에서 **RDP**를 선택합니다. **IP 주소** 드롭다운 목록에서 **부하 분산 장치 공용 IP 주소** 항목을 선택하고 **RDP 파일 다운로드를** 선택합니다.

1. 메시지가 표시되면 다음 자격 증명으로 로그인합니다.

    | 설정 | 값 | 
    | --- | --- |
    | 사용자 이름 | **Student** |
    | 암호 | **Pa55w.rd1234** |

1. **az30410a-vm1**에 대한 원격 데스크톱 내 Server Manager 창에서 왼쪽에 항목 중 **Local Server**를 선택하고 **IE Enhanced Security Configuration** 레이블 옆에 있는 **On** 링크를 선택한 다음, **IE Enhanced Security Configuration** 대화 상자에서 둘 다 **Off** 옵션을 선택합니다.

1. **az30410a-vm1**에 대한 원격 데스크톱 세션 내에서 Server Manager창 오른쪽 위쪽에 메뉴 **Tools**를 선택하고 드롭다운 메뉴에서 **Active Directory Administrative Center**를 선택합니다.

1. **Active Directory Administrative Center**에서 **adatum(로컬)** 을 선택하고 오른쪽 레이어에 **Tasks** 창에서 **New**를 선택한 후 계단식 메뉴에서 **Organization Unit**를 선택합니다.

1. **Create Organization Unit** 창의 **Name** 텍스트 상자에 **ToSync**를 입력하고 **OK**을 선택합니다.

1. 새로 만든 **ToSync** Organization Unit 를 두 번 클릭하여 해당 콘텐츠가 Active Directory Administrative Center 콘솔의 세부 정보 창에 표시도록 합니다. 

1. **Tasks** 창의 **ToSync** 섹션 내에서 **New**를 선택하고 계단식 메뉴에서 **User**를 선택합니다.

1. **Create User** 창에서 다음 설정이 있는 새 사용자 계정을 만들고(다른 설정 값은 그대로 유지한 채) **OK**을 선택합니다.

    | 설정 | 값 | 
    | --- | --- |
    | Full Name | **aduser1** |
    | User UPN logon | **aduser1** |
    | User SamAccountName logon | **aduser1** |
    | Password | **Pa55w.rd1234** | 
    | Other password options | **Password never expires** |


#### 작업 3: Azure AD Connect 설치

1. **az30410a-vm1**에 대한 원격 데스크톱 세션 내에서 Internet Explorer를 시작하고, [Azure portal](https://portal.azure.com)로 이동하여 이전 연습에서 만든 **az30410-aaduser1** 사용자 계정을 통해 로그인합니다. 메시지가 표시되면 기록한 전체 사용자 이름과 **Pa55w.rd1234** 암호를 지정합니다.

1. Azure Portal에서 **Azure Active Directory**를 검색하여 선택하고 **Adatum Lab | 개요** 블레이드에서 **Azure AD Connect**를 선택합니다.

1. **Adatum Lab | Azure AD Connect** 블레이드에서 **Azure AD Connect 다운로드** 링크를 선택합니다. **Microsoft Azure Active Directory Connect** 다운로드 페이지로 리디렉션됩니다.

1. **Microsoft Azure Active Directory Connect** 다운로드 페이지에서 **다운로드**를 선택합니다.

1. 메시지가 표시되면 **실행**을 선택하여 **Microsoft Azure Active Directory Connect** 마법사를 시작합니다.

1. **Microsoft Azure Active Directory Connect** 마법사의 **Welcome to Azure AD Connect** 페이지에서 **I agree to the license terms and privacy notice**라는 확인란을 선택하고 **continue**을 선택합니다.

1. **Microsoft Azure Active Directory Connect** 마법사의 **Express Settings** 페이지에서 **Customize** 옵션을 선택합니다.

1. **Install required components** 페이지에서 모든 선택적 구성 옵션의 선택을 취소하고 **Install**를 선택합니다.

1. **User sign-in** 페이지에서 **Password Hash Synchronization**만 사용하도록 설정되었는지 확인하고 **Next**을 선택합니다.

1. **Connect to Azure AD** 페이지에서 이전 연습에서 만든 **az30410-aaduser1**(예 : az30410-aaduser1@hahaysh.onmicrosoft.com) 사용자 계정의 자격 증명을 사용하여 인증하고 **Next**을 선택합니다. 

1. **Connect your directories** 페이지에서 **adatum.com** 포리스트 항목 오른쪽에 있는 **Add Directory** 버튼을 선택합니다.

1. **AD forest account** 창에서 **Create new AD account** 옵션이 선택되었는지 확인하고 다음 자격 증명을 지정한 후 **OK**을 선택합니다:

    | 설정 | 값 | 
    | --- | --- |
    | 사용자 이름 | **ADATUM\Student** |
    | 암호 | **Pa55w.rd1234** |

1. **Connect your directories** 페이지에서 **adatum.com** 항목이 configured directory로 표시되는지 확인하고 **Next**을 선택합니다

1. **Azure AD sign-in configuration** 페이지에서 **Users will not be able to sign-in to Azure AD with on-premises credentials if the UPN suffix does not match a verified domain name**라는 경고를 확인하고, **Continue without matching all UPN suffixes to verified domain** 확인란에 체크한 후 **다음**을 선택합니다.

    > **참고**: 앞서 설명했듯이 사용자 지정 Azure AD DNS 도메인 **adatum.com**수업 용으로 검증/확인할 수 없기 때문에 이러한 결과가 예상됩니다.

1. **Domain and OU filtering** 페이지에서 **Sync selected domains and OUs** 옵션을 선택하고 모든 확인란을 선택 취소한 후 **ToSync** OU 옆에 있는 확인란만 선택하고 **Next**를 선택합니다.

1. **Uniquely identifying your users** 페이지에서 기본 설정을 수락하고 **Next**을 선택합니다.

1. **Filter users and devices** 페이지에서 기본 설정을 수락하고 **Next**를 선택합니다.

1. **Optional features** 페이지에서 기본 설정을 수락하고 **Next**을 선택합니다.

1. **Ready to configure** 페이지에서 **Start the synchronization process when configuration completes** 확인란이 선택되었는지 확인하고 **Install**를 선택합니다.

    > **참고**: 설치에는 약 2분이 소요됩니다.

1. **Configuration complete** 페이지의 정보를 검토하고 **Exit**를 선택하여 **Microsoft Azure Active Directory Connect** 창을 닫습니다.


#### 작업 4: 동기화된 사용자 계정의 속성 구성

1. **az30410a-vm1** 원격 데스크톱 세션 내에 있는 Azure Portal을 표시하는 Internet Explorer 창에서 Adatum Lab Azure AD 테넌트의 **사용자 | 모든 사용자(미리 보기)** 블레이드로 이동합니다.

1. **사용자 | 모든 사용자(미리 보기)** 블레이드에서, 사용자 개체 목록에 **aduser1** 계정이 포함되며 **디렉토리가 동기화됨**이 **Yes** 로 표시되는 것을 확인합니다.

    > **참고**: **aduser1** 사용자 계정이 표시되도록 하려면 잠시 기다렸다가 **새로 고침**을 선택합니다.

1. **사용자 | 모든 사용자(미리 보기)** 블레이드에서 **aduser1** 항목을 선택합니다.

1. **aduser1 | 프로필** 블레이드는 사용자 계정의 전체 이름을 기록합니다.

    > **참고**: 전체 사용자 이름을 기록합니다. 다음 연습에서 필요합니다.

1. **aduser1 | 프로필** 블레이드의 **작업 정보** 섹션에서 **부서** 특성이 설정되지 않음에 유의합니다.

1. **az30410a-vm1**에 대한 원격 데스크톱 세션에서 **Active Directory Administrative Center**로 전환하고, **ToSync** OU의 개체 목록에서 **aduser1** 항목을 선택한 후 **Tasks** 창의 **Properties**을 선택합니다.

1. **aduser1** 창에 있는 **Organization** 섹션의 **Department** 텍스트 상자에 **Sales**을 입력하고 **Ok**을 선택합니다.

1. **az30410a-vm1**에 대한 원격 데스크톱 세션 내에서, 관리자 권한으로 **Windows PowerShell**을 시작합니다.

1. **Administrator: Windows PowerShell** 콘솔은 다음을 실행하여 Azure AD Connect 델타 동기화를 시작합니다:

   ```powershell
   Import-Module -Name 'C:\Program Files\Microsoft Azure AD Sync\Bin\ADSync\ADSync.psd1'

   Start-ADSyncSyncCycle -PolicyType Delta
   ```

1. **aduser1 | 프로필** 블레이드를 표시하는 Internet Explorer 창으로 전환하고 페이지상단에 **새로 고침** 후에 **부서** 속성이 **Sales**으로 설정되어 있음을 유의하십시오.

    > **참고**: **부서** 특성이 설정되지 않은 상태로 유지되면 몇 분 동안 기다렸다가 페이지를 다시 새로 고침하세요.

1. **aduser1 | 프로필** 블레이드에서 **편집**을 선택합니다.

1. **aduser1 | 설정** 섹션의 **프로필** 블레이드에 있는 **사용량 위치** 드롭다운 목록에서 **United States**을 선택한 다음 **저장**을 선택합니다.

1. **aduser1 | 프로필** 블레이드에서 **라이선스**를 선택합니다.

1. **aduser1 | 라이선스** 블레이드에서 **+ 할당**을 선택합니다.

1. **라이선스 할당 업데이트** 블레이드에서 **Azure Active Directory Premium P2** 확인란을 선택하고 **저장**을 선택합니다.


### 연습 3: Azure AD 조건부 액세스 구현
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure AD 보안 기본값을 사용하지 않도록 설정.

1. Azure AD 조건부 액세스 정책 만들기

1. Azure AD 조건부 액세스 확인

1. 랩에 배포된 Azure 리소스 제거


#### 작업 1: Azure AD 보안 기본값을 사용하지 않도록 설정.

1. **az30410a-vm1**에 대한 원격 데스크톱 세션 내, Azure Portal을 표시하는 Internet Explorer 창에서 **Adatum Lab | 개요** 블레이드로 이동합니다.

1. **Adatum Lab | 개요** 블레이드의 **관리** 섹션에서 **속성**을 선택합니다.

1. **Adatum Lab | 속성** 블레이드에서 페이지 하단에 있는 **보안 기본값 관리** 링크를 선택합니다.

1. **보안 기본값 실행** 블레이드에서 **보안 기본값 실행** 스위치를 **아니요**로 설정하고 **내 조직에서 조건부 액세스를 사용 중임** 확인란을 선택한 후 **저장**을 선택합니다. 


#### 작업 2: Azure AD 조건부 액세스 정책 만들기

1. **Adatum Lab | 속성** 블레이드의 **관리** 섹션에서 **보안**을 선택합니다.

1. **보안 | 시작** 블레이드에서 **조건부 액세스**를 선택합니다.

1. **조건부 액세스 | 정책** 블레이드에서 **+ 새 정책**을 선택합니다.

1. **새로 만들기** 블레이드의 **이름** 텍스트 상자에 **Azure portal MFA enforcement**을 입력합니다. 

1. **새로 만들기** 블레이드의 **할당** 섹션에서 **사용자 및 그룹**을 선택하고 **포함** 탭에서 **사용자 및 그룹 선택**을 선택한 후 **사용자 및 그룹** 확인란을 선택하고 **선택** 블레이드에서 **aduser1**을 선택하고 **선택**을 클릭하여 선택 사항을 확인합니다.

1. **새로 만들기** 블레이드로 돌아가 **할당** 섹션에서 **클라우드 앱 또는 작업**을 선택하고 **포함** 탭에서 **앱 선택**을 선택한 후 **선택**을 클릭하고 **선택** 블레이드에서 **Microsoft Azure Management** 확인란을 선택한 후 **선택**을 클릭하여 선택 사항을 확인합니다.

1. **새로 만들기** 블레이드로 돌아가 **액세스 제어** 섹션에서 **허용**를 선택하고 **허용** 블레이드에서 **액세스 허용** 옵션이 선택되었는지 확인한 후 **다단계 인증 기능 필요**를 선택하고 **선택**을 클릭하여 선택 사항을 확인합니다.

1. **새로 만들기** 블레이드로 돌아가 **정책 사용** 스위치를 **설정**으로 설정하고 **만들기**를 선택합니다.


#### 작업 3: Azure AD 조건부 액세스 확인

1. **az30410a-vm1**에 대한 원격 데스크톱 세션 내에서 새로운 **InPrivate 브라우징** Internet Explorer 창을 시작하고 액세스 패널 애플리케이션 포털([https://account.activedirectory.windowsazure.com](https://account.activedirectory.windowsazure.com))로 이동합니다.

1. 메시지가 표시되면 이전 연습에 기록된 전체 사용자 이름과 **Pa55w.rd1234** 암호를 사용하여 **aduser1** (예 : aduser1@hahaysh.onmicrosoft.com)의 동기화된 Azure AD 계정을 통해 로그인합니다. 

1. 액세스 패널 애플리케이션 포털에 성공적으로 로그인했는지 확인합니다. 

1. 동일한 브라우저 창에서 [Azure portal](https://portal.azure.com)로 이동합니다.

1. 이번에는 **More information required** 메시지가 표시됩니다. 메시지를 표시하는 페이지 내에서 **Next**를 선택합니다. 

1. 이 시점에서 **추가 보안 확인** 페이지로 리디렉션되어 다단계 인증 구성에 대해 안내 받습니다.

    > **참고**: 다단계 인증 구성을 완료하는 것은 선택 사항입니다. 계속하면 모바일 디바이스를 인증 전화로 지정하거나 모바일 앱을 실행하는 데 사용해야 합니다.


#### 작업 4: 랩에 배포된 Azure 리소스 제거

1. **az30410a-vm1**에 대한 원격 데스크톱 세션 내에서 Internet Explorer를 시작하고 [https://go.microsoft.com/fwlink/p/?LinkId=286152](https://go.microsoft.com/fwlink/p/?LinkId=286152)에서 Microsoft Online Services Sign-In Assistant for IT Professionals RTW로 이동합니다. 

1. Microsoft Online Services Sign-In Assistant for IT Professionals RTW 다운로드 페이지에서 **다운로드**를 선택하고 **원하는 다운로드 선택**에서 **en\msoidcli_64.msi**를 선택한 후 **다음**을 선택합니다. 

1. 기본 옵션으로 다운로드 후에 **Microsoft Online Services Sign-in Assistant Setup**을 설치합니다.

1. 설치가 완료되면 **az30410a-vm1**에 대한 원격 데스크톱 세션 내에서 **Windows PowerShell** 콘솔을 관리자 권한으로 시작합니다.

1. **Administrator: Windows PowerShell** 창은 다음을 실행하여 필요한 PowerShell 모듈을 설치합니다:

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force
   Install-Module MSOnline -Force
   ```
1. **관리자:**에서**Windows PowerShell** 창은 **Adatum Lab** Azure AD 테넌트에 인증하기 위해 다음을 실행합니다:

   ```powershell
   Connect-MsolService
   ```

1. 인증하라는 메시지가 표시되면 **az30410-aaduser1**(예 : az30410-aaduser1@hahaysh.onmicrosoft.com) 사용자 계정의  자격 증명을 제공합니다.

1. **관리자:**에서**Windows PowerShell** 창은 다음을 실행하여 Azure AD Connect 동기화를 사용하지 않도록 설정합니다:

   ```powershell
   Set-MsolDirSyncEnabled -EnableDirSync $false -Force
   ```

1. 랩 컴퓨터에서 Azure Portal을 표시하는 브라우저 창에서 Azure Active Directory에서 라이선스, 모든제품, **Azure Active Directory Premium P2 - 허가된 사용자** 블레이드로 이동하여 이 랩에서 라이선스를 할당한 사용자 계정을 선택하고 **라이선스 제거**를 선택하고 확인하라는 메시지가 표시되면 **확인**을 선택합니다.

1. Azure Portal에서 Azure Active Directory에서 **사용자 - 모든 사용자(미리 보기)** 블레이드로 이동하여 이 랩에서 만든 모든 사용자 계정에 **원본** 열에 **Azure Active Directory** 항목이 있는지 확인합니다. 

1. **사용자 | 모든 사용자(미리 보기)** 블레이드에서 이 랩에서 만든 각 사용자 계정을 선택하고 도구 모음에서 **삭제**를 선택합니다. 

1. Azure Active Directory에서  **Adatum Lab - 개요** 블레이드로 이동하여 **테넌트 삭제**를 선택하고 **디렉터리 'Adatum Lab' 삭제** 블레이드에서 **Azure 리소스 링크를 삭제할 수 있는 권한 가져오기**를 선택하고 Azure Active Directory의 **속성** 블레이드에서 **Azure 리소스 액세스 관리**를 **예**로 설정하고 **저장**을 선택합니다.

1. Azure Portal에서 로그아웃하고 다시 로그인합니다. 

1. **디렉터리 'Adatum Lab' 삭제** 블레이드로 다시 이동하여 **삭제**를 선택합니다.

1. **az30410a-vm1**에 대한 원격 데스크톱 세션 내 Azure Portal을 표시하는 브라우저 창의 Cloud Shell 창 내에서 PowerShell 세션을 시작합니다.

1. Cloud Shell 창에서 다음을 실행하여 이 연습에서 만든 리소스 그룹을 나열합니다.

   ```powershell
   Get-AzResourceGroup -Name 'az30410*'
   ```

    > **참고**: 이 랩에서 만든 리소스 그룹만 출력에 포함되어 있는지 확인합니다. 이 작업에서는 이러한 그룹을 삭제할 것입니다.

1. Cloud Shell 창에서 다음을 실행하여 이 랩에서 만든 리소스 그룹을 삭제합니다.

   ```powershell
   Get-AzResourceGroup -Name 'az30410*' | Remove-AzResourceGroup -Force -AsJob
   ```

1. Cloud Shell 창을 닫습니다.
