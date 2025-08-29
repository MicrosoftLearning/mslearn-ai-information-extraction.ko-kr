---
lab:
  title: 콘텐츠 이해 클라이언트 응용 프로그램 개발
  description: Azure AI 콘텐츠 이해 REST API를 사용하여 분석기용 클라이언트 앱을 개발합니다.
---

# 콘텐츠 이해 클라이언트 응용 프로그램 개발

이 연습에서는 Azure AI 콘텐츠 이해를 사용하여 명함에서 정보를 추출하는 분석기를 만듭니다. 그런 다음 분석기를 사용하여 스캔한 명함에서 연락처 세부 정보를 추출하는 클라이언트 애플리케이션을 개발합니다.

이 연습에는 약 **30**분이 소요됩니다.

## Azure AI 파운드리 허브 및 프로젝트 만들기

이 연습에서 사용할 Azure AI 파운드리의 기능에는 Azure AI 파운드리 *허브* 리소스를 기반으로 하는 프로젝트가 필요합니다.

1. 웹 브라우저에서 [Azure AI 파운드리 포털](https://ai.azure.com)(`https://ai.azure.com`)을 열고 Azure 자격 증명을 사용하여 로그인합니다. 처음 로그인할 때 열리는 팁이나 빠른 시작 창을 닫고, 필요한 경우 왼쪽 위에 있는 **Azure AI 파운드리** 로고를 사용하여 다음 이미지와 유사한 홈페이지로 이동합니다(**도움말** 창이 열려 있는 경우 닫습니다).

    ![Azure AI Foundry 포털의 스크린샷.](./media/ai-foundry-home.png)

1. 브라우저에서 `https://ai.azure.com/managementCenter/allResources`로 이동하여 **새로 만들기**를 선택합니다. 그런 다음 새 **AI 허브 리소스**를 만드는 옵션을 선택합니다.
1. **프로젝트 만들기** 마법사에서 유효한 프로젝트 이름을 입력하고 새 허브를 만드는 옵션을 선택합니다. 그런 다음 **허브 이름 바꾸기** 링크를 사용하여 새 허브의 유효한 이름을 지정하고 **고급 옵션**을 확장한 후 프로젝트에 대해 다음 설정을 지정합니다.
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: ‘리소스 그룹 만들기 또는 선택’
    - **허브 이름**: 허브에서 유효한 이름
    - **위치**: 다음 위치 중 하나를 선택합니다.\*
        - 오스트레일리아 동부
        - 스웨덴 중부
        - 미국 서부

    > \*작성 시점에 Azure AI 콘텐츠 이해는 이러한 지역에서만 사용할 수 있습니다.

    > **팁**: **만들기** 단추가 여전히 사용하지 않도록 설정된 경우 허브 이름을 고유한 영숫자 값으로 바꿔야 합니다.

1. 프로젝트가 만들어질 때까지 기다린 다음 프로젝트 개요 페이지로 이동합니다.

## REST API 사용하여 Content 이해 분석기 만들기

REST API 사용하여 명함 이미지에서 정보를 추출할 수 있는 분석기를 만듭니다.

1. 새 브라우저 탭을 엽니다(Azure AI 파운드리 포털을 기존 탭에서 열어 두기). 그런 다음 새 탭에서 [Azure Portal](https://portal.azure.com)(`https://portal.azure.com`)을 열고 메시지가 나타나면 Azure 자격 증명을 사용하여 로그인합니다.

    Azure Portal 홈페이지를 보려면 환영 알림을 닫습니다.

1. 페이지 상단의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 구독에 저장소가 없는 ***PowerShell*** 환경을 선택합니다.

    Cloud Shell은 Azure Portal 하단의 창에서 명령줄 인터페이스를 제공합니다. 보다 쉽게 작업할 수 있도록 이 창의 크기를 조정하거나 최대화할 수 있습니다.

    > **팁**: 대부분 클라우드 셸에서 작업할 수 있지만 Azure Portal 페이지에서 키 및 엔드포인트를 계속 볼 수 있도록 창의 크기를 조정합니다. 이를 코드에 복사해야 합니다.

1. Cloud Shell 도구 모음의 **설정** 메뉴에서 **클래식 버전으로 이동**을 선택합니다(코드 편집기를 사용하는 데 필요).

    **<font color="red">계속하기 전에 Cloud Shell의 클래식 버전으로 전환했는지 확인합니다.</font>**

1. Cloud Shell 창에서 다음 명령을 입력하여 이 연습의 코드 파일이 포함된 GitHub 리포지토리를 복제합니다(명령을 입력하거나 클립보드에 복사한 다음 명령줄을 마우스 오른쪽 단추로 클릭하고 일반 텍스트로 붙여넣기).

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **팁**: CloudShell에 명령을 입력하면 출력이 화면 버퍼의 많은 부분을 차지할 수 있습니다. `cls` 명령을 입력해 화면을 지우면 각 작업에 더 집중할 수 있습니다.

1. 리포지토리가 복제된 후 앱의 코드 파일이 들어 있는 폴더로 이동합니다.

    ```
   cd mslearn-ai-info/Labfiles/content-app
   ls -a -l
    ```

    폴더에는 앱을 빌드하는 데 필요한 Python 코드 파일뿐만 아니라 두 개의 스캔된 명함 이미지가 포함되어 있습니다.

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 사용할 라이브러리를 설치합니다.

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt
    ```

1. 제공된 구성 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 코드 파일에서 **YOUR_ENDPOINT** 및 **YOUR_KEY** 자리 표시자를 Azure AI 서비스 엔드포인트와 해당 키 중 하나(Azure Portal에서 복사됨)로 바꾸고 **ANALYZER_NAME**이 `business-card-analyzer`로(으로) 설정되어 있는지 확인합니다.
1. 자리 표시자를 바꾼 후 코드 편집기에서 **CTRL+S** 명령을 사용하여 변경 내용을 저장한 다음 **CTRL+Q** 명령을 사용하여 Cloud Shell 명령줄을 열어둔 채 코드 편집기를 닫습니다.

    > **팁**: 이제 Cloud shell 창을 최대화할 수 있습니다.

1. Cloud shell 명령줄에서 다음 명령을 입력하여 제공된 **biz-card.json** JSON 파일을 확인합니다.

    ```
   cat biz-card.json
    ```

    Cloud shell 창을 스크롤하여 명함의 분석기 스키마를 정의하는 파일의 JSON을 확인합니다.

1. 분석기의 JSON 파일을 확인한 후 다음 명령을 입력하여 제공된 **create-analyzer.py** Python 코드 파일을 편집합니다.

    ```
   code create-analyzer.py
    ```

    Python 코드 파일은 코드 편집기에서 열립니다.

1. 다음 코드를 검토합니다.
    - **biz-card.json** 파일에서 분석기 스키마를 로드합니다.
    - 환경 구성 파일에서 엔드포인트, 키 및 분석기 이름을 검색합니다.
    - 현재 구현되지 않은 **create_analyzer**라는 함수를 호출합니다.

1. **create_analyzer** 함수에서 **콘텐츠 이해 분석기 만들기** 설명을 찾아 다음 코드를 추가합니다(올바른 들여쓰기를 유지하도록 주의하세요).

    ```python
   # Create a Content Understanding analyzer
   print (f"Creating {analyzer}")

   # Set the API version
   CU_VERSION = "2025-05-01-preview"

   # initiate the analyzer creation operation
   headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"}

   url = f"{endpoint}/contentunderstanding/analyzers/{analyzer}?api-version={CU_VERSION}"

   # Delete the analyzer if it already exists
   response = requests.delete(url, headers=headers)
   print(response.status_code)
   time.sleep(1)

   # Now create it
   response = requests.put(url, headers=headers, data=(schema))
   print(response.status_code)

   # Get the response and extract the callback URL
   callback_url = response.headers["Operation-Location"]

   # Check the status of the operation
   time.sleep(1)
   result_response = requests.get(callback_url, headers=headers)

   # Keep polling until the operation is no longer running
   status = result_response.json().get("status")
   while status == "Running":
        time.sleep(1)
        result_response = requests.get(callback_url, headers=headers)
        status = result_response.json().get("status")

   result = result_response.json().get("status")
   print(result)
   if result == "Succeeded":
        print(f"Analyzer '{analyzer}' created successfully.")
   else:
        print("Analyzer creation failed.")
        print(result_response.json())
    ```

1. 다음의 추가한 코드를 검토합니다.
    - REST 요청에 적합한 헤더 만들기
    - HTTP *DELETE* 요청을 제출하여 분석기가 이미 있는 경우 삭제합니다.
    - HTTP *PUT* 요청을 제출하여 분석기 만들기를 시작합니다.
    - 응답을 확인하여 *Operation-Location* 콜백 URL을 검색합니다.
    - 콜백 URL에 HTTP *GET* 요청을 반복적으로 제출하여 더 이상 실행되지 않을 때까지 작업 상태를 확인합니다.
    - 사용자에게 작업의 성공(또는 실패)을 확인합니다.

    > **참고**: 이 코드에는 서비스에 대한 요청 속도 제한을 초과하지 않도록 몇 가지 의도적인 시간 지연이 포함됩니다.

1. **CTRL+S** 명령을 사용하여 코드 변경 내용을 저장하지만 코드의 오류를 수정해야 하는 경우 코드 편집기 창을 열어 둡니다. 명령줄 창을 명확하게 볼 수 있도록 창의 크기를 조정합니다.
1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 Python 코드를 실행합니다.

    ```
   python create-analyzer.py
    ```

1. 프로그램의 출력을 검토합니다. 이 출력은 분석기가 만들어졌음을 나타냅니다.

## REST API를 사용하여 콘텐츠 분석

이제 분석기를 만들었으므로 콘텐츠 이해 REST API를 통해 클라이언트 응용 프로그램에서 사용할 수 있습니다.

1. Cloud Shell 명령줄에서 다음 명령을 입력하여 제공된 **read-card.py** Python 코드 파일을 편집합니다.

    ```
   code read-card.py
    ```

    Python 코드 파일은 코드 편집기에서 열립니다.

1. 다음 코드를 검토합니다.
    - 분석할 이미지 파일을 식별하며 기본값은 **biz-card-1.png**입니다.
    - 프로젝트에서 Azure AI 서비스 리소스의 엔드포인트 및 키를 검색합니다(현재 Cloud shell 세션의 Azure 자격 증명을 사용하여 인증).
    - 현재 구현되지 않은 **analyze_card**라는 함수를 호출합니다.

1. **analyze_card** 함수에서 **콘텐츠 이해를 사용하여 이미지 분석** 설명을 찾아 다음 코드를 추가합니다(올바른 들여쓰기를 유지하도록 주의하세요).

    ```python
   # Use Content Understanding to analyze the image
   print (f"Analyzing {image_file}")

   # Set the API version
   CU_VERSION = "2025-05-01-preview"

   # Read the image data
   with open(image_file, "rb") as file:
        image_data = file.read()
    
   ## Use a POST request to submit the image data to the analyzer
   print("Submitting request...")
   headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/octet-stream"}
   url = f'{endpoint}/contentunderstanding/analyzers/{analyzer}:analyze?api-version={CU_VERSION}'
   response = requests.post(url, headers=headers, data=image_data)

   # Get the response and extract the ID assigned to the analysis operation
   print(response.status_code)
   response_json = response.json()
   id_value = response_json.get("id")

   # Use a GET request to check the status of the analysis operation
   print ('Getting results...')
   time.sleep(1)
   result_url = f'{endpoint}/contentunderstanding/analyzerResults/{id_value}?api-version={CU_VERSION}'
   result_response = requests.get(result_url, headers=headers)
   print(result_response.status_code)

   # Keep polling until the analysis is complete
   status = result_response.json().get("status")
   while status == "Running":
        time.sleep(1)
        result_response = requests.get(result_url, headers=headers)
        status = result_response.json().get("status")

   # Process the analysis results
   if status == "Succeeded":
        print("Analysis succeeded:\n")
        result_json = result_response.json()
        output_file = "results.json"
        with open(output_file, "w") as json_file:
            json.dump(result_json, json_file, indent=4)
            print(f"Response saved in {output_file}\n")

        # Iterate through the fields and extract the names and type-specific values
        contents = result_json["result"]["contents"]
        for content in contents:
            if "fields" in content:
                fields = content["fields"]
                for field_name, field_data in fields.items():
                    if field_data['type'] == "string":
                        print(f"{field_name}: {field_data['valueString']}")
                    elif field_data['type'] == "number":
                        print(f"{field_name}: {field_data['valueNumber']}")
                    elif field_data['type'] == "integer":
                        print(f"{field_name}: {field_data['valueInteger']}")
                    elif field_data['type'] == "date":
                        print(f"{field_name}: {field_data['valueDate']}")
                    elif field_data['type'] == "time":
                        print(f"{field_name}: {field_data['valueTime']}")
                    elif field_data['type'] == "array":
                        print(f"{field_name}: {field_data['valueArray']}")
    ```

1. 다음의 추가한 코드를 검토합니다.
    - 이미지 파일의 내용을 읽습니다.
    - 사용할 콘텐츠 이해 REST API 버전을 설정합니다.
    - 콘텐츠 이해 엔드포인트에 HTTP *POST* 요청을 제출하여 이미지를 분석하도록 지시합니다.
    - 작업의 응답을 확인하여 분석 작업의 ID를 검색합니다.
    - 더 이상 실행되지 않을 때까지 작업 상태를 확인하기 위해 콘텐츠 이해 엔드포인트에 HTTP *GET* 요청을 반복적으로 제출합니다.
    - 작업이 성공하면 JSON 응답을 저장한 다음 JSON을 구문 분석하고 각 형식별 필드에 대해 검색된 값을 표시합니다.

    > **참고**: 간단한 명함 스키마에서 모든 필드는 문자열입니다. 이 코드는 더 복잡한 스키마에서 다양한 형식의 값을 추출할 수 있도록 각 필드의 형식을 확인해야 하는 필요성을 보여 줍니다.

1. **CTRL+S** 명령을 사용하여 코드 변경 내용을 저장하지만 코드의 오류를 수정해야 하는 경우 코드 편집기 창을 열어 둡니다. 명령줄 창을 명확하게 볼 수 있도록 창의 크기를 조정합니다.
1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 Python 코드를 실행합니다.

    ```
   python read-card.py biz-card-1.png
    ```

1. 다음 명함의 필드에 대한 값을 표시해야 하는 프로그램의 출력을 검토합니다.

    ![Adventure Works Cycles 직원인 Roberto Tamburello의 명함입니다.](./media/biz-card-1.png)

1. 다음 명령을 사용하여 다른 명함으로 프로그램을 실행합니다.

    ```
   python read-card.py biz-card-2.png
    ```

1. 이 명함의 값을 반영해야 하는 결과를 검토합니다.

    ![Contoso 직원인 Mary Duartes의 명함입니다.](./media/biz-card-2.png)

1. Cloud Shell 명령줄 창에서 다음 명령을 사용하여 반환된 전체 JSON 응답을 봅니다.

    ```
   cat results.json
    ```

    스크롤하여 JSON을 봅니다.

## 정리

콘텐츠 이해 서비스에서 작업을 완료한 경우 불필요한 Azure 비용이 발생하지 않도록 이 연습에서 만든 리소스를 삭제해야 합니다.

1. Azure Portal에서 이 연습용으로 만든 리소스를 삭제합니다.
