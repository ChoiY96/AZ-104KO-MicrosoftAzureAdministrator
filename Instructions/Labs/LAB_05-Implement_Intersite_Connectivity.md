---
lab:
    title: '05 - 사이트 간 연결 구현'
    module: '모듈 05 - 사이트 간 연결'
---

# 랩 05 - 사이트 간 연결 구현
# 학생 랩 매뉴얼

## 랩 시나리오

Contoso는 보스턴, 뉴욕 및 시애틀 사무소에 광역 네트워크 링크로 연결된 데이터 센터가 있으며, 이들 간의 완벽한 연결이 가능합니다. Contoso 온-프레미스 네트워크의 토폴로지를 반영하고 그 기능을 확인하는 랩 환경을 구현해야 합니다. 

## 목적

이 랩에서는 다음 작업을 수행합니다.

+ 작업 1: 랩 환경 프로비전
+ 작업 2: 로컬 및 글로벌 가상 네트워크 피어링 구성
+ 작업 3: 사이트 간 연결 테스트 

## 예상 시간: 30분

### 지침

#### 작업 1: 랩 환경 프로비전

이 작업에서는 세 개의 가상 머신을 각각 별도의 가상 네트워크에 배포하고 그 중 두 개는 동일한 Azure 지역에, 세 번째는 다른 Azure 지역에 배포합니다. 

1. [Azure Portal](https://portal.azure.com)에 로그인합니다.

1. Azure Portal 오른쪽 상단의 아이콘을 클릭하여 **Azure Cloud Shell**을 엽니다.

1. **Bash**또는 **PowerShell** 중 하나를 선택하라는 메시지가 표시되면 **PowerShell**을 선택합니다.      

    >**참고**: **Cloud Shell**을 처음 시작할 때 **탑재된 스토리지가 없음** 메시지가 나타나면 이 랩에서 사용 중인 구독을 선택하고 **스토리지 만들기**를 클릭합니다. 

1. Cloud Shell 창 도구 모음에서 **파일 업로드/다운로드** 아이콘을 클릭하고, 드롭다운 메뉴에서 **업로드**를 클릭하여 **\\Allfiles\\Labs\\05\\az104-05-vnetvm-template.json** 및 **\\Allfiles\\Labs\\05\\az104-05-vnetvm-parameters.json** 파일을 Cloud Shell 홈 디렉터리에 업로드합니다.

1. Cloud Shell 창에서 다음을 실행하여 첫 번째 가상 네트워크와 가상 머신 쌍을 호스팅할 첫 번째 리소스 그룹을 만듭니다.(Azure 가상 머신을 배포할 Azure 지역 이름으로 `[Azure_region_1]` 자리 표시자를 바꿉니다.)

   ```pwsh
   $location = '[Azure_region_1]'

   $rgName = 'az104-05-rg0'

   New-AzResourceGroup -Name $rgName -Location $location
   ```
   >**참고**: Cloud Shell의 PowerShell 세션에서 Azure 지역을 식별하려면 **(Get-AzLocation).Location**을 실행합니다.
   
1. Cloud Shell 창에서 다음을 실행하여 첫 번째 가상 네트워크를 만들고 업로드한 템플릿 및 매개 변수 파일을 사용하여 가상 머신을 배포합니다.

   ```pwsh
   New-AzResourceGroupDeployment `
      -ResourceGroupName $rgName `
      -TemplateFile $HOME/az104-05-vnetvm-template.json `
      -TemplateParameterFile $HOME/az104-05-vnetvm-parameters.json `
      -nameSuffix 0 `
      -AsJob
   ```
1. Cloud Shell 창에서 다음을 실행하여 두 번째 가상 네트워크 및 두 번째 가상 머신을 호스팅할 두 번째 리소스 그룹을 만듭니다.

   ```pwsh
   $rgName = 'az104-05-rg1'

   New-AzResourceGroup -Name $rgName -Location $location
   ```
1. Cloud Shell 창에서 다음을 실행하여 두 번째 가상 네트워크를 만들고 업로드한 템플릿 및 매개 변수 파일을 사용하여 가상 머신을 배포합니다.

   ```pwsh
   New-AzResourceGroupDeployment `
      -ResourceGroupName $rgName `
      -TemplateFile $HOME/az104-05-vnetvm-template.json `
      -TemplateParameterFile $HOME/az104-05-vnetvm-parameters.json `
      -nameSuffix 1 `
      -AsJob
   ```
1. Cloud Shell 창에서 다음을 실행하여 세 번째 가상 네트워크와 세 번째 가상 머신을 호스팅할 세 번째 리소스 그룹을 만듭니다.(다른 두 배포에서 사용한 Azure 지역과 달리 Azure 가상 머신을 배포할 수 있는 다른 Azure 지역 이름으로 `[Azure_region_2]` 자리 표시자를 바꿉니다.)

   ```pwsh
   $location = '[Azure_region_2]'

   $rgName = 'az104-05-rg2'

   New-AzResourceGroup -Name $rgName -Location $location
   ```
1. Cloud Shell 창에서 세 번째 가상 네트워크를 만들고 업로드한 템플릿 및 매개 변수 파일을 사용하여 가상 머신을 배포합니다.

   ```pwsh
   New-AzResourceGroupDeployment `
      -ResourceGroupName $rgName `
      -TemplateFile $HOME/az104-05-vnetvm-template.json `
      -TemplateParameterFile $HOME/az104-05-vnetvm-parameters.json `
      -nameSuffix 2 `
      -AsJob
   ```
    >**참고**: 다음 작업을 진행하기 전에 배포가 완료될 때까지 기다립니다. 약 2분이 소요됩니다.

    >**참고**: 배포 상태를 확인하기 위해 이 작업에서 만든 리소스 그룹의 속성을 검사할 수 있습니다.

1. Cloud Shell 창을 닫습니다.

#### 작업 2: 로컬 및 글로벌 가상 네트워크 피어링 구성

이 작업에서는 이전 작업에서 배포된 가상 네트워크 간의 로컬 및 글로벌 피어링을 구성합니다.

1. Azure Portal에서 **가상 네트워크**를 검색하고 선택합니다.

1. 이전 작업에서 만든 가상 네트워크를 검토하고 처음 두 개가 동일한 Azure 지역에 있고 세 번째가 다른 Azure 지역에 있는지 확인합니다. 

    >**참고**: 세 개의 가상 네트워크 배포에 사용된 템플릿은 가상 네트워크 세 개의 IP 주소 범위가 겹치지 않도록 합니다.

1. 가상 네트워크 목록에서 **az104-05-vnet0**을 클릭합니다.

1. **az104-05-vnet0** 가상 네트워크 블레이드의 **설정** 섹션에서 **피어링**을 클릭한 다음 **+ 추가**를 클릭합니다.

1. 다음 설정으로 피어링을 추가합니다(다른 설정은 기본값을 그대로 둡니다.)

    | 설정 | 값|
    | --- | --- |
    | az104-05-vnet0에서 원격 가상 네트워크로 피어링 이름 | **az104-05-vnet0_to_az104-05-vnet1** |
    | 가상 네트워크 배포 모델 | **리소스 관리자** |
    | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
    | 가상 네트워크 | **az104-05-vnet1 (az104-05-rg1)** |
    | az104-05-vnet1에서 az104-05-vnet0으로 피어링 이름 | **az104-05-vnet1_to_az104-05-vnet0** |
    | az104-05-vnet0에서 az104-05-vnet1로 가상 네트워크 액세스 허용 | **사용** |
    | az104-05-vnet1에서 az104-05-vnet0으로 가상 네트워크 액세스 허용 | **사용** |
    | az104-05-vnet1에서 az104-05-vnet0으로 전달되는 트래픽 허용 | **사용 안 함** |
    | az104-05-vnet0에서 az104-05-vnet1로 전달되는 트래픽 허용 | **사용 안 함** |
    | 게이트웨이 전송 허용 | **(확인란 선택 취소)** |

    >**참고**: 이 단계에서 하나는 az104-05-vnet0에서 az104-05-vnet1로, 다른 하나는 az104-05-vnet1에서 az104-05-vnet0으로 두 개의 로컬 피어링을 설정합니다.

1. **az104-05-vnet0** 가상 네트워크 블레이드의 **설정** 섹션에서 **피어링**을 클릭한 다음 **+ 추가**를 클릭합니다.

1. 다음 설정으로 피어링을 추가합니다(다른 설정은 기본값을 그대로 둡니다.)

    | 설정 | 값|
    | --- | --- |
    | az104-05-vnet0에서 원격 가상 네트워크로 피어링 이름 | **az104-05-vnet0_to_az104-05-vnet2** |
    | 가상 네트워크 배포 모델 | **리소스 관리자** |
    | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
    | 가상 네트워크 | **az104-05-vnet2 (az104-05-rg2)** |
    | az104-05-vnet2에서 az104-05-vnet0으로 피어링 이름 | **az104-05-vnet2_to_az104-05-vnet0** |
    | az104-05-vnet0에서 az104-05-vnet2 가상 네트워크 액세스 허용 | **사용** |
    | az104-05-vnet2에서 az104-05-vnet0으로 가상 네트워크 액세스 허용 | **사용** |
    | az104-05-vnet2에서 az104-05-vnet0으로 전달된 트래픽 허용 | **사용 안 함** |
    | az104-05-vnet0에서 az104-05-vnet2로 전달된 트래픽 허용 | **사용 안 함** |
    | 게이트웨이 전송 허용 | **(확인란 선택 취소)** |

    >**참고**: 이 단계에서 하나는 az104-05-vnet0에서 az104-05-vnet2로, 다른 하나는 az104-05-vnet2에서 az104-05-vnet0으로 두 개의 글로벌 피어링을 설정합니다.

1. **가상 네트워크** 블레이드로 돌아가 가상 네트워크 목록에서 **az104-05-vnet1**을 클릭합니다.

1. **az104-05-vnet1** 가상 네트워크 블레이드의 **설정** 섹션에서 **피어링**과 **+추가**를 클릭합니다.

1. 다음 설정으로 피어링을 추가합니다(다른 설정은 기본값을 그대로 둡니다.)

    | 설정 | 값|
    | --- | --- |
    | az104-05-vnet1에서 원격 가상 네트워크로 피어링 이름 | **az104-05-vnet1_to_az104-05-vnet2** |
    | 가상 네트워크 배포 모델 | **리소스 관리자** |
    | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
    | 가상 네트워크 | **az104-05-vnet2 (az104-05-rg2)** |
    | az104-05-vnet2에서 az104-05-vnet1로 피어링 이름 | **az104-05-vnet2_to_az104-05-vnet1** |
    | az104-05-vnet1에서 az104-05-vnet2로 가상 네트워크 액세스 허용 | **사용** |
    | az104-05-vnet2에서 az104-05-vnet1로 가상 네트워크 액세스 허용 | **사용** |
    | az104-05-vnet2에서 az104-05-vnet1로 전달된 트래픽 허용 | **사용 안 함** |
    | az104-05-vnet1에서 az104-05-vnet2로 전달된 트래픽 허용 | **사용 안 함** |
    | 게이트웨이 전송 허용 | **(확인란 선택 취소)** |

    >**참고**: 이 단계에서 하나는 az104-05-vnet1에서 az104-05-vnet2로, 다른 하나는 az104-05-vnet2에서 az104-05-vnet1로 두 개의 전역 피어링을 설정합니다.

#### 작업 3: 사이트 간 연결 테스트 

이 작업에서는 이전 작업의 로컬 및 글로벌 피어링을 통해 연결된 세 개의 가상 네트워크에서 가상 머신 간의 연결을 테스트합니다.

1. Azure Portal 에서 **가상 머신**을 찾아서 선택합니다.

1. 가상 머신 목록에서 **az104-05-vm0**을 클릭합니다.

1. **az104-05-vm0**블레이드에서 **연결**을 클릭하고 드롭다운 메뉴에서 **RDP**를 클릭한 다음 **RDP로 연결** 블레이드에서 **RDP 파일 다운로드**를 클릭하고 메시지를 따라 원격 데스크톱 세션을 시작합니다.

    >**참고**: 이 단계는 Windows 컴퓨터에서 원격 데스크톱을 통해 연결하는 것을 말합니다. Mac에서는 Mac App Store의 원격 데스크톱 클라이언트를 사용할 수 있으며 Linux 컴퓨터에서는 오픈 소스 RDP 클라이언트 소프트웨어를 사용할 수 있습니다.

    >**참고**: 대상 가상 머신에 연결할 때 경고 메시지를 무시할 수 있습니다.

1. 메시지가 표시되면 사용자 이름 **Student**과 암호 **Pa55w.rd1234**를 사용하여 로그인합니다.

1. **az104-05-vm0**에 대한 원격 데스크톱 세션 내에서 **시작** 버튼을 마우스 오른쪽 버튼으로 클릭하고 마우스 오른쪽 단추 클릭 메뉴에서 **Windows PowerShell(관리자)** 를 클릭합니다.

1. Windows PowerShell 콘솔 창에서 다음 명령을 실행하여 TCP 포트 3389를 통해 **az104-05-vm1**(개인 IP 주소가 **10.51.0.4**)에 대한 연결을 테스트합니다.

   ```pwsh
   Test-NetConnection -ComputerName 10.51.0.4 -Port 3389 -InformationLevel 'Detailed'
   ```
    >**참고**: 테스트에서는 운영 체제 방화벽에 의해 기본적으로 허용되는 포트 TCP 3389를 사용합니다. 

1. 명령 출력을 검사하여 연결되었는지 확인합니다.

1. Windows PowerShell 콘솔 창에서 다음 명령을 실행하여 **az104-05-vm2**에 대한 연결을 테스트합니다(개인 IP 주소가 **10.52.0.4**입니다).

   ```pwsh
   Test-NetConnection -ComputerName 10.52.0.4 -Port 3389 -InformationLevel 'Detailed'
   ```
1. 랩 컴퓨터의 Azure Portal 로 다시 전환하여 **가상 머신** 블레이드로 다시 이동합니다. 

1. 가상 머신 목록에서 **az104-05-vm1**을 클릭하십시오.

1. **az104-05-vm1** 블레이드에서 **연결**을 클릭하고 드롭다운 메뉴에서 **RDP**를 클릭하며 **RDP로 연결** 블레이드에서 **RDP 파일 다운로드**를 클릭한 다음 메시지를 따라 원격 데스크톱 세션을 시작합니다.

    >**참고**: 이 단계는 Windows 컴퓨터에서 원격 데스크톱을 통해 연결하는 것을 말합니다. Mac에서는 Mac App Store의 원격 데스크톱 클라이언트를 사용할 수 있으며 Linux 컴퓨터에서는 오픈 소스 RDP 클라이언트 소프트웨어를 사용할 수 있습니다.

    >**참고**: 대상 가상 머신에 연결할 때 경고 메시지를 무시할 수 있습니다.

1. 메시지가 표시되면 사용자 이름 **Student**과 암호 **Pa55w.rd1234**를 사용하여 로그인합니다.

1. **az104-05-vm1**에 대한 원격 데스크톱 세션 내에서 **시작 버튼**을 마우스 오른쪽 버튼으로 클릭하고 오른쪽 버튼을 클릭한 메뉴에서 **Windows PowerShell(관리자)** 을 클릭합니다.

1. Windows PowerShell 콘솔 창에서 다음 명령을 실행하여 TCP 포트 3389에서 **az104-05-vm2**에 대한 연결을 테스트합니다(개인 IP 주소가 **10.52.0.4**).

   ```pwsh
   Test-NetConnection -ComputerName 10.52.0.4 -Port 3389 -InformationLevel 'Detailed'
   ```
    >**참고**: 테스트에서는 운영 체제 방화벽에 의해 기본적으로 허용되는 포트 TCP 3389를 사용합니다. 

1. 명령 출력을 검사하여 연결되었는지 확인합니다.

#### 리소스 정리

   >**참고**: 더 이상 사용하지 않는 새로 만든 Azure 리소스를 제거해야 합니다. 사용하지 않는 리소스를 제거하면 해당 비용이 발생하지 않습니다.

1. Azure Portal 에서 **Cloud Shell** 창 내부의 **PowerShell** 세션을 엽니다.

1. 다음 명령을 실행하여 본 모듈의 랩 전체에서 만든 모든 리소스 그룹을 나열합니다.

   ```pwsh
   Get-AzResourceGroup -Name 'az104-05*'
   ```

1. 다음 명령을 실행하여 본 모듈의 랩 전체에서 만든 모든 리소스 그룹을 삭제합니다.

   ```pwsh
   Get-AzResourceGroup -Name 'az104-05*' | Remove-AzResourceGroup -Force -AsJob
   ```

    >**참고**: 명령은 비동기적으로 실행(-AsJob 매개 변수에 의해 결정)되므로 동일한 PowerShell 세션 내에서 즉시 다른 PowerShell 명령을 실행할 수 있지만 리소스 그룹이 실제로 제거되기까지 몇 분 정도 걸릴 수 있습니다.

#### 리뷰

이 랩에서는 다음 작업을 했습니다.

- 랩 환경 프로비전 
- 로컬 및 글로벌 가상 네트워크 피어링 구성
- 사이트 간 연결 테스트 
