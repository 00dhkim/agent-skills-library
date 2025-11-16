# Google ADK (Agent Development Kit) 마스터 가이드: 코드 중심 아키텍처, 오케스트레이션 및 배포

## I. Google ADK (Agent Development Kit) 개요: 코드 중심의 에이전트 개발 프레임워크

Google의 Agent Development Kit(ADK)는 AI 에이전트의 개발, 배포, 오케스트레이션을 위한 유연하고 모듈화된 오픈소스 프레임워크입니다. ADK의 핵심 철학은 AI 에이전트 개발 프로세스를 전통적인 소프트웨어 개발 수명주기와 유사하게 만드는 것입니다.

이 "코드 중심(Code-First)" 접근 방식은 에이전트의 로직, 도구 사용, 오케스트레이션 워크플로우를 Python과 같은 프로그래밍 언어로 직접 정의합니다. 이를 통해 개발자는 LLM(대규모 언어 모델) 기반 애플리케이션의 본질적인 비결정론적 특성을 관리하고, 강력한 디버깅, 안정적인 버전 관리, 향상된 테스트 용이성 등 검증된 소프트웨어 엔지니어링의 이점을 에이전트 개발에 적용할 수 있습니다.

### 주요 특징: 모델 및 배포 불가지론

ADK는 Google의 Gemini 모델 및 Vertex AI 생태계에 최적화되어 있지만, 근본적으로 특정 기술 스택에 종속되지 않습니다.

*   **모델 불가지론(Model-Agnostic):** ADK는 특정 LLM에 얽매이지 않습니다. LiteLLM과의 통합을 통해 Anthropic (Claude), Meta (Llama), Mistral AI 등 업계 전반의 다양한 모델을 유연하게 선택하고 교체할 수 있습니다.
    
*   **배포 불가지론(Deployment-Agnostic):** 개발된 에이전트는 로컬 머신에서 테스트하는 것부터 Docker 컨테이너를 통해 Google Cloud Run이나 Google Kubernetes Engine(GKE)에 배포하는 것, 나아가 Vertex AI Agent Engine과 같은 완전 관리형 서비스에 이르기까지 다양한 환경에 배포 가능합니다.
    

### ADK 생태계의 구성 요소

ADK는 에이전트 개발의 전체 수명주기를 지원하는 통합 도구 세트로 구성됩니다.

1.  **ADK 프레임워크 (Python, Go, Java):** 에이전트의 핵심 로직, 도구, 워크플로우를 코드로 정의하는 핵심 라이브러리입니다.
    
2.  **CLI (Command-Line Interface):**`adk create` (프로젝트 생성), `adk run` (터미널 실행), `adk web` (UI 실행), `adk deploy` (클라우드 배포) 등 프로젝트 관리를 위한 필수 명령어 도구입니다.
    
3.  **ADK Web UI:**`adk web` 명령어로 실행되는 내장형 개발 UI입니다. 개발자는 이 웹 인터페이스를 통해 에이전트와 실시간으로 채팅하며 시각적으로 테스트, 디버깅 및 시연할 수 있습니다.
    
4.  **Visual Agent Builder:** 드래그앤드롭 인터페이스와 자연어 대화를 통해 복잡한 다중 에이전트 시스템을 시각적으로 설계하고 구성할 수 있는 브라우저 기반 IDE입니다. 이 도구는 백그라운드에서 유효한 ADK YAML 구성을 생성하므로, 로우코드(Low-Code) 접근 방식과 코드 중심(Code-First) 개발 간의 다리 역할을 합니다.
    

ADK의 아키텍처는 근본적으로 시스템의 신뢰성을 높이는 데 중점을 둡니다. 이는 에이전트 유형을 두 가지로 명확히 구분하는 것에서 드러납니다: (1) LLM의 추론에 의존하는 비결정론적 `LlmAgent`와 (2) 예측 가능한 로직(예: 순차, 병렬)에 따라 작동하는 결정론적 `WorkflowAgent`. 이러한 분리를 통해 개발자는 예측 불가능한 LLM 호출로 인한 실패 지점을 강력하고 테스트 가능한 코드 기반 구조로 대체하여, 엔터프라이즈급 애플리케이션에 필요한 안정성을 확보할 수 있습니다.

## II. 시작하기: 설치 및 기본 에이전트 구축 (실행 코드 포함)

이 섹션에서는 ADK 설치부터 도구를 갖춘 간단한 단일 에이전트를 생성하고 실행하는 전 과정을 안내합니다.

### 환경 설정: Python 가상 환경 및 의존성

시작하기 전에 시스템에 Python 3.9 이상 버전과 패키지 관리자인 `pip`가 설치되어 있어야 합니다.

먼저, `pip`를 사용하여 Google ADK를 설치합니다.

Bash

```
pip install google-adk
```

프로젝트 간의 의존성 충돌을 방지하기 위해 Python 가상 환경을 생성하고 활성화하는 것이 강력히 권장됩니다.

Bash

```
# 1. 가상 환경 생성
python -m venv.venv

# 2. 가상 환경 활성화 (MacOS / Linux)
source.venv/bin/activate

# 2. 가상 환경 활성화 (Windows CMD)
#.venv\Scripts\activate.bat
```

### ADK 프로젝트 생성 및 구조

ADK CLI를 사용하여 새 에이전트 프로젝트를 생성합니다.

Bash

```
adk create my_agent
```

이 명령어는 `my_agent`라는 디렉토리를 생성하며, 그 구조는 다음과 같습니다 :

*   `my_agent/`
    
    *   `agent.py`: 에이전트의 메인 제어 로직이 포함된 파일입니다.
        
    *   `.env`: API 키와 같은 환경 변수를 저장하는 파일입니다.
        
    *   `__init__.py`: 이 디렉토리를 Python 패키지로 인식시킵니다.
        

### 핵심 구성: API 키 설정

ADK는 기본적으로 Google Gemini API를 사용하며, 이를 위해 API 키가 필요합니다. Google AI Studio에서 API 키를 발급받은 후, `.env` 파일에 환경 변수로 저장해야 합니다.

터미널에서 다음 명령을 실행하여 `.env` 파일을 업데이트합니다. ( `"YOUR_API_KEY"` 부분을 실제 키로 대체하십시오.)

Bash

```
echo 'GOOGLE_API_KEY="YOUR_API_KEY"' > my_agent/.env
```

### \[실행 가능 코드 1\] 단일 도구를 사용한 기본 `Agent` 생성

ADK 에이전트의 유일한 필수 요소는 `agent.py` 파일 내에 `root_agent` 객체를 정의하는 것입니다. 다음은 `get_current_time`이라는 간단한 모의(mock) 도구를 포함하는 완전한 `my_agent/agent.py` 파일입니다.

Python

```
# my_agent/agent.py
# ADK의 핵심 Agent 클래스를 가져옵니다.
from google.adk.agents.llm_agent import Agent

# 1. 도구 정의 (사용자 정의 함수)
def get_current_time(city: str) -> dict:
    """
    지정된 도시의 현재 시간을 반환합니다.
    (이것은 실제 API 호출을 시뮬레이션하는 모의 함수입니다.)
    """
    print(f" get_current_time(city={city})")
    # 실제 구현에서는 'requests' 라이브러리를 사용하여
    # worldtimeapi.org 같은 외부 API를 호출할 수 있습니다.
    
    # 이 예제에서는 항상 고정된 시간을 반환합니다.
    if city.lower() == "london":
        return {"status": "success", "city": "London", "time": "09:30 AM"}
    elif city.lower() == "paris":
        return {"status": "success", "city": "Paris", "time": "10:30 AM"}
    else:
        return {"status": "error", "message": f"{city}의 시간을 찾을 수 없습니다."}

# 2. 루트 에이전트 정의
# 이것이 ADK 에이전트의 핵심입니다.
root_agent = Agent(
    # 사용할 LLM 모델을 지정합니다 (Gemini).
    model='gemini-2.5-flash', 
    
    # 에이전트의 이름과 설명 (Web UI 및 디버깅에 사용됨)
    name='root_agent',
    description="지정된 도시의 현재 시간을 알려줍니다.",
    
    # 시스템 프롬프트: 에이전트의 역할과 도구 사용법을 지시합니다.
    instruction="당신은 도시에 있는 현재 시간을 알려주는 유용한 비서입니다. "
                  "이 목적을 위해 'get_current_time' 도구를 사용해야 합니다. "
                  "도시 이름을 찾을 수 없으면 사용자에게 다시 물어보십시오.",
    
    # 에이전트가 사용할 수 있는 도구 목록을 전달합니다.
    # ADK는 함수의 docstring을 자동으로 파싱하여 LLM이 이해할 수 있도록 합니다.
    tools=[get_current_time],
)
```

### 에이전트 실행 및 테스트

ADK는 에이전트와 상호작용하고 테스트하기 위한 두 가지 기본 인터페이스를 제공합니다.

1.  **명령줄 인터페이스 (CLI)로 실행:** 터미널에서 직접 에이전트와 대화할 수 있습니다.

Bash \`\`\` adk run my\_agent

```
2.   **웹 인터페이스 (Web UI)로 실행:** ADK는 로컬 웹 서버를 실행하여 사용하기 쉬운 채팅 인터페이스를 제공합니다.

Bash  ```
adk web --port 8000
```

**중요:**`adk web` 명령어는 `my_agent/` 폴더를 포함하는 **상위 디렉토리**에서 실행해야 합니다. 예를 들어, 프로젝트가 `~/projects/my_agent/`에 있다면, `~/projects/` 디렉토리에서 명령을 실행해야 합니다.

실행 후 브라우저에서 `http://localhost:8000`에 접속하면 채팅 UI를 통해 에이전트를 테스트할 수 있습니다.

## III. ADK 도구 생태계: 에이전트 역량 확장 (실행 코드 포함)

ADK에서 '도구(Tool)'는 에이전트가 LLM의 핵심 추론 능력을 넘어, 외부 세계와 상호작용하고 구체적인 작업을 수행할 수 있도록 부여된 기능(capability)을 의미합니다. 도구는 데이터베이스 쿼리, API 요청, 웹 검색, RAG(검색 증강 생성), 코드 실행 등 모든 프로그래밍 함수가 될 수 있습니다.

LLM이 사용자 요청을 분석하여 도구 사용을 결정(Selection)하면, ADK 프레임워크가 해당 함수를 실행(Invocation)하고, 그 결과를 다시 LLM에게 관찰(Observation)시켜 최종 응답을 생성하도록 하는 'Selection-Invocation-Observation' 루프를 통해 작동합니다.

### \[실행 가능 코드 2\] 사용자 정의 함수 도구: `FunctionTool`

Python 함수를 ADK 도구로 변환하는 가장 직접적인 방법은 `FunctionTool` 클래스로 래핑(wrapping)하는 것입니다.`FunctionTool`은 함수의 시그니처(인수, 타입 힌트)와 docstring을 자동으로 파싱하여 LLM이 도구의 기능과 사용법을 이해할 수 있는 스키마로 변환합니다.

다음은 `get_weather_report`라는 함수를 도구로 사용하는 완전한 `weather_agent/agent.py` 파일입니다.

Python

```
# weather_agent/agent.py
import asyncio
from google.adk.agents import Agent
from google.adk.tools import FunctionTool 

# 1. 도구로 사용할 Python 함수 정의
def get_weather_report(city: str) -> dict: 
    """
    지정된 도시에 대한 현재 날씨 보고서를 검색합니다.
    외부 날씨 API 호출을 시뮬레이션합니다.
    
    Args:
        city (str): 날씨를 조회할 도시 이름.

    Returns:
        dict: 'status' 키와 'report' (성공 시) 또는 'error_message' (실패 시)를 포함한 사전.
    """ 
    print(f" get_weather_report(city={city})")
    
    city_lower = city.lower()
    weather_data = {
        "london": {"status": "success", "report": "런던은 현재 흐리고 18도이며 비 올 확률이 있습니다."},
        "paris": {"status": "success", "report": "파리는 맑고 25도입니다."},
        "tokyo": {"status": "success", "report": "도쿄는 습하고 29도입니다."}
    }
    
    return weather_data.get(city_lower, {"status": "error", "error_message": f"'{city}'의 날씨 정보는 없습니다."})

# 2. 함수를 FunctionTool로 래핑
# FunctionTool이 docstring을 자동으로 파싱합니다.
weather_tool = FunctionTool(func=get_weather_report) 

# 3. 루트 에이전트 정의
root_agent = Agent( 
    model='gemini-2.5-flash', 
    name='weather_assistant',
    description="날씨 정보를 제공하는 비서",
    instruction="""당신은 날씨 정보를 제공하는 유용한 비서입니다.
    1. 사용자가 특정 도시의 날씨를 물으면 'get_weather_report' 도구를 사용하십시오.
    2. 도구가 'success'를 반환하면 날씨 보고서를 친절하게 전달하십시오.
    3. 도구가 'error'를 반환하면 해당 도시의 정보를 찾을 수 없다고 정중히 알리십시오.
    """, 
    # 래핑된 도구를 에이전트에 전달
    tools=[weather_tool] 
)
```

### \[실행 가능 코드 3\] 외부 API 연동: `OpenAPIToolset`

수십, 수백 개의 엔드포인트가 있는 복잡한 REST API를 통합해야 할 때, 각 엔드포인트마다 `FunctionTool`을 수동으로 만드는 것은 비효율적입니다. ADK는 `OpenAPIToolset`이라는 강력한 솔루션을 제공합니다.

`OpenAPIToolset`은 OpenAPI v3 사양(JSON 또는 YAML) 파일을 읽어들여, 해당 API의 _모든 엔드포인트_ 를 호출 가능한 ADK 도구 세트로 _자동 생성_ 합니다. 또한 API 키, OAuth 등 필요한 인증 메커니즘을 처리하는 기능도 내장하고 있습니다.

다음은 GitHub REST API를 `OpenAPIToolset`으로 연동하는 `github_agent/agent.py` 예제입니다.

**사전 준비:**

1.  GitHub 개인 액세스 토큰(PAT)을 발급받아 `github_agent/.env` 파일에 `GITHUB_TOKEN="your_github_pat"` 형식으로 저장합니다.
    
2.  GitHub OpenAPI 사양 파일을 다운로드하여 `github_agent/api.github.com.json`으로 저장합니다. (예: `wget https://raw.githubusercontent.com/github/rest-api-description/main/descriptions/api.github.com.json -O github_agent/api.github.com.json`)
    

Python

```
# github_agent/agent.py
import os
import json
from google.adk.agents import Agent
from google.adk.tools.openapi_tool.openapi_spec_parser.openapi_toolset import OpenAPIToolset
from google.adk.tools.openapi_tool.auth.auth_helpers import token_to_scheme_credential

# 1. 환경 변수 및 API 사양 경로 설정
API_SPEC_PATH = "api.github.com.json" # 같은 디렉토리에 저장된 사양 파일
TOKEN_ENV_VAR = "GITHUB_TOKEN"

token = os.getenv(TOKEN_ENV_VAR)
if not token:
    raise ValueError(f"{TOKEN_ENV_VAR} 환경 변수가.env 파일에 설정되지 않았습니다.")

try:
    with open(API_SPEC_PATH, 'r') as f:
        spec = json.load(f)
except FileNotFoundError:
    raise FileNotFoundError(f"'{API_SPEC_PATH}' 파일을 찾을 수 없습니다. 다운로드하십시오.")

# 2. ADK용 인증 자격 증명 포맷팅
# GitHub는 'Bearer <TOKEN>' 형식을 사용합니다.
auth_scheme, auth_credential = token_to_scheme_credential(
    "bearer", # 인증 유형 (bearer, apikey 등)
    "header", # 토큰 위치
    "Authorization", # 헤더 이름
    token # 실제 토큰 값 (ADK가 'Bearer ' 접두사를 자동으로 추가함)
)

# 3. OpenAPIToolset 생성 (핵심)
# spec_str에 JSON 사양을 문자열로 전달합니다.
toolset = OpenAPIToolset(
    spec_str=json.dumps(spec),
    spec_str_type="json",
    auth_scheme=auth_scheme,
    auth_credential=auth_credential
)

# 4. Toolset에서 모든 도구 추출
# 이 리스트에는 'repos_list_issues', 'users_get_by_username' 등
# GitHub API의 모든 엔드포인트가 도구로 포함됩니다.
all_github_tools = toolset.get_tools()

# 5. 에이전트 정의
root_agent = Agent(
    name="github_agent",
    description="GitHub API와 상호작용하는 에이전트",
    # OpenAPI 스키마는 매우 복잡하므로, 강력한 모델(예: Pro)을 권장합니다.
    model="gemini-2.5-pro-preview-03-25", 
    instruction="""당신은 GitHub 비서입니다. 
    OpenAPI 도구를 사용하여 사용자의 요청에 응답하십시오.
    예: 'google/adk-python 리포지토리의 이슈 목록을 보여줘'
    'repos_list_issues' 같은 적절한 도구를 호출해야 합니다.
    """,
    # 자동 생성된 수백 개의 도구를 에이전트에 전달
    tools=all_github_tools 
)
```

### 사전 구축된 도구 및 생태계

ADK는 Google 생태계와 긴밀하게 통합된 다양한 사전 구축 도구를 제공하여, 개발자가 일반적인 작업을 즉시 에이전트에 통합할 수 있도록 지원합니다.

**표 1: ADK 도구 생태계 개요**

카테고리 도구 예시 설명 **Gemini 도구**`google_search`Google 검색을 통한 실시간 정보 접근 및 접지(Grounding). `code_interpreter`LLM이 생성한 Python 코드 스니펫을 안전하게 실행. **Google Cloud 도구**`Vertex AI RAG Engine`조직의 비공개 지식 기반(문서)에서 정보를 검색 (RAG). `Vertex AI Search`비공개 데이터 저장소에서 구성된 데이터를 검색. `BigQuery Tools`BigQuery 데이터베이스에 대한 자연어 쿼리 및 분석 수행. `Apigee API Hub`Apigee에 문서화된 모든 엔터프라이즈 API를 즉시 도구로 전환. `GKE Code Executor`GKE의 보안 환경에서 AI 생성 코드를 실행. **서드파티 도구**AgentQL, Exa, Firecrawl 웹사이트에서 구조화된 데이터를 추출, 검색 및 크롤링. GitHub, Notion 코드 리포지토리 관리, 작업 공간 검색 등. **사용자 정의 도구**`FunctionTool`모든 Python 함수를 에이전트 도구로 래핑. `OpenAPIToolset`OpenAPI v3 사양에서 도구 세트를 자동 생성.

ADK의 설계에서 주목할 점은 RAG(Retrieval-Augmented Generation)를 에이전트의 _유형_ 이 아닌 _도구_ 로 취급한다는 것입니다.`Vertex AI RAG Engine` 또는 `rag_tool`을 도구 목록에 추가함으로써, 에이전트의 LLM은 '문서 검색'이라는 작업을 다른 도구(날씨 검색, API 호출 등)와 동일하게 취급합니다. 이는 RAG 파이프라인(문서 수집, 임베딩, 벡터 검색)의 복잡성을 에이전트의 핵심 로직(`agent.py`)에서 완벽하게 추상화합니다. 이러한 설계는 에이전트의 시스템 프롬프트를 단순하게 유지하고, 전체 아키텍처의 모듈성을 극대화하는 세련된 엔지니어링 결정입니다.

## IV. 오케스트레이션: 결정론적 워크플로우 에이전트 (실행 코드 포함)

복잡한 작업을 수행하려면 여러 단계나 여러 에이전트의 협력이 필요합니다. ADK는 이러한 실행 흐름을 제어하기 위해 `WorkflowAgent`라는 강력한 메커니즘을 제공합니다.

`WorkflowAgent`는 `LlmAgent`와 근본적으로 다릅니다. 이 에이전트들은 LLM을 사용하여 다음에 무엇을 할지 _추론_ 하지 않고, 개발자가 사전에 정의한 _결정론적 로직_(예: 순차, 병렬, 루프)에 따라 하위 에이전트들을 실행합니다. 이는 워크플로우의 예측 가능성, 테스트 용이성, 신뢰성을 보장합니다.

### \[실행 가능 코드 4\] `SequentialAgent` (순차 실행) (Generator-Critic 패턴)

`SequentialAgent`는 하위 에이전트 목록을 제공된 순서대로 하나씩 실행합니다.

에서 제공된 코드 파이프라인(Code Pipeline) 예제는 이 패턴의 완벽한 활용 사례이며, 동시에 'Generator-Critic' 패턴(생성자-비평가) 의 구체적인 구현입니다.

*   **핵심 메커니즘 (상태 공유):** 하위 에이전트 간의 데이터 전달은 공유 세션 상태(`session.state`)를 통해 이루어집니다. 첫 번째 에이전트가 `output_key="generated_code"`를 사용해 `state['generated_code']`에 결과를 쓰면, 다음 에이전트가 프롬프트에서 `{generated_code}` 키워드를 사용하여 해당 값을 읽어들입니다.

Python

```
# code_pipeline/agent.py
from google.adk.agents.llm_agent import Agent
from google.adk.agents.workflow_agents.sequential_agent import SequentialAgent

# 1. 하위 에이전트 1: CodeWriter (Generator)
# 사용자의 요청을 받아 초기 코드를 생성합니다.
code_writer_agent = Agent(
    name="CodeWriterAgent",
    model="gemini-2.5-flash",
    instruction="당신은 Python 코드 생성기입니다. 사용자 요청에 따라 Python 코드를 작성하고, "
                  "오직 ```python... ``` 코드 블록만 출력하십시오.",
    description="초기 Python 코드를 작성합니다.",
    # 실행 결과를 'generated_code' 키로 세션 상태(state)에 저장합니다.
    output_key="generated_code" 
)

# 2. 하위 에이전트 2: CodeReviewer (Critic)
# 이전 단계의 코드를 검토합니다.
code_reviewer_agent = Agent(
    name="CodeReviewerAgent",
    model="gemini-2.5-flash",
    # 이전 단계의 'generated_code'를 프롬프트 입력으로 참조합니다.
    instruction="당신은 전문 Python 코드 리뷰어입니다. 다음 코드를 검토하십시오:\n{generated_code}\n\n"
                  "가독성, 효율성, 잠재적 버그를 검토하고 개선을 위한 피드백을 불릿 포인트로 제공하십시오. "
                  "문제가 없으면 'No major issues found.'라고만 응답하십시오.",
    description="코드를 검토하고 피드백을 제공합니다.",
    # 검토 결과를 'review_comments' 키로 세션 상태에 저장합니다.
    output_key="review_comments"
)

# 3. 하위 에이전트 3: CodeRefactorer (Updater)
# 검토 의견을 바탕으로 코드를 수정합니다.
code_refactorer_agent = Agent(
    name="CodeRefactorerAgent",
    model="gemini-2.5-flash",
    # 'generated_code'와 'review_comments'를 모두 입력으로 참조합니다.
    instruction="당신은 Python 코드 리팩토링 AI입니다. "
                  "**Original Code**:\n{generated_code}\n\n"
                  "**Review Comments**:\n{review_comments}\n\n"
                  "위의 리뷰 코멘트를 반영하여 원본 코드를 개선하십시오. "
                  "리뷰가 'No major issues found.'이면 원본 코드를 그대로 반환하십시오. "
                  "오직 수정된 ```python... ``` 코드 블록만 출력하십시오.",
    description="검토 의견을 바탕으로 코드를 리팩토링합니다.",
    # 최종 코드를 'refactored_code' 키로 저장합니다.
    output_key="refactored_code"
)

# 4. 루트 에이전트: SequentialAgent
# 이 에이전트가 전체 워크플로우를 결정론적으로 조율합니다.
root_agent = SequentialAgent(
    name="CodePipelineAgent",
    description="코드 작성, 검토, 리팩토링 파이프라인을 순차적으로 실행합니다.",
    # 하위 에이전트가 리스트 순서대로 실행됩니다:
    # 1. CodeWriterAgent
    # 2. CodeReviewerAgent
    # 3. CodeRefactorerAgent
    sub_agents=[
        code_writer_agent, 
        code_reviewer_agent, 
        code_refactorer_agent
    ]
)
```

### \[실행 가능 코드 5\] `LoopAgent` (반복 실행)

`LoopAgent`는 하위 에이전트 시퀀스를 지정된 횟수(`max_iterations`) 또는 특정 종료 조건이 충족될 때까지 반복적으로 실행합니다.

종료 조건은 일반적으로 루프 내의 하위 에이전트가 특정 조건을 평가한 후, `tool_context.actions.escalate = True`를 설정하여 상위 `LoopAgent`에게 중단 신호를 보내는 방식으로 구현됩니다.

Python

```
# iterative_refinement/agent.py
from google.adk.agents import Agent, LoopAgent
from google.adk.tools import FunctionTool, ToolContext

# 1. 종료 조건 검사 도구
def check_if_done(critique: str, tool_context: ToolContext):
    """
    비평(critique) 텍스트를 분석하여 작업이 완료되었는지 
    (예: "No further changes", "Perfect") 판단합니다.
    완료되었다고 판단되면 루프를 중단시킵니다.
    """
    print(f" check_if_done(critique={critique[:20]}...)")
    critique_lower = critique.lower()
    
    # 종료 조건 확인
    if "no further changes" in critique_lower or "perfect" in critique_lower:
        print(" 'escalate=True' 설정. 루프를 종료합니다.")
        # 이 플래그가 LoopAgent에게 중지 신호를 보냅니다.
        tool_context.actions.escalate = True 
        return {"status": "finished"}
    
    return {"status": "needs_revision"}

# 도구로 래핑
check_done_tool = FunctionTool(func=check_if_done)

# --- 루프 내에서 실행될 하위 에이전트들 ---

# 2. 하위 에이전트 1: Writer
# 매 루프마다 현재 {draft}를 입력받아 개선합니다.
writer_agent = Agent(
    name="WriterAgent",
    model="gemini-2.5-flash",
    instruction="당신은 작가입니다. 다음 초안을 비평({critique})을 바탕으로 개선하십시오:\n{draft}\n"
                  "이것이 첫 번째 루프라면({critique}가 비어있다면), {draft}를 기반으로 새 버전을 작성하십시오."
                  "개선된 버전을 'improved_draft' 키로 출력하십시오.",
    description="초안을 수정합니다.",
    output_key="improved_draft"
)

# 3. 하위 에이전트 2: Critic
# 개선된 {improved_draft}를 입력받아 비평합니다.
critic_agent = Agent(
    name="CriticAgent",
    model="gemini-2.5-flash",
    instruction="당신은 엄격한 비평가입니다. 다음 초안을 검토하십시오:\n{improved_draft}\n"
                  "간결한 비평을 'critique' 키로 제공하십시오. "
                  "초안이 완벽하다면 'Perfect, no further changes needed.'라고만 말하십시오.",
    description="개선된 초안을 비평합니다.",
    output_key="critique"
)

# 4. 하위 에이전트 3: State Updater
# 다음 루프를 위해 상태를 전이시킵니다.
# (다음 루프의 'draft'는 이번 루프의 'improved_draft'가 됩니다.)
state_updater_agent = Agent(
    name="StateUpdater",
    model="gemini-2.5-flash", # 실제 LLM 호출은 필요 없음
    instruction="상태를 업데이트합니다.", # 더미 프롬프트
    description="다음 루프를 위해 상태를 매핑합니다.",
    # 'improved_draft'의 값을 'draft' 키에 복사합니다.
    # 'critique' 값은 그대로 유지합니다.
    output_key_map={
        "draft": "{improved_draft}",
        "critique": "{critique}"
    }
)

# 5. 하위 에이전트 4: Termination Checker
# 비평({critique})을 바탕으로 루프 종료 여부를 결정합니다.
checker_agent = Agent(
    name="CheckerAgent",
    model="gemini-2.5-flash",
    instruction="현재 비평({critique})을 바탕으로 'check_if_done' 도구를 즉시 호출하여 "
                  "루프 종료 여부를 결정하십시오.",
    tools=[check_done_tool]
)

# 6. 루트 에이전트: LoopAgent
root_agent = LoopAgent(
    name="IterativeRefinementAgent",
    description="초안을 최대 3회 반복하며 개선하거나, 비평가가 만족하면 중단합니다.",
    
    # 최대 반복 횟수 (무한 루프 방지)
    max_iterations=3, 
    
    # 루프마다 이 시퀀스를 순차적으로 실행합니다.
    sub_agents=[
        writer_agent,       # 1. 쓰기
        critic_agent,       # 2. 비평하기
        state_updater_agent,  # 3. 상태 전이 (draft <- improved_draft)
        checker_agent       # 4. 종료 확인
    ]
)

# 참고: 이 에이전트를 실행하려면 
# 사용자가 첫 번째 입력으로 state['draft'] = "사용자의 초기 아이디어"를 제공해야 합니다.
```

### \[실행 가능 코드 6\] `ParallelAgent` (병렬 실행)

`ParallelAgent`는 하위 에이전트 목록을 _동시에_ 병렬로 실행합니다. 이는 서로 독립적인 정보를(예: 날씨와 뉴스) 한 번에 가져와 응답 시간을 단축하는 'Fan-Out/Gather' 패턴에 유용합니다.

모든 병렬 실행 에이전트들은 동일한 `session.state`에 접근하여, 고유한 `output_key`를 통해 결과를 비동기적으로 씁니다.

Python

```
# parallel_data_fetcher/agent.py
from google.adk.agents import Agent
from google.adk.agents.workflow_agents.parallel_agent import ParallelAgent
from google.adk.agents.workflow_agents.sequential_agent import SequentialAgent

# --- 병렬로 실행될 작업들 ---

# 1. 병렬 작업 1: 날씨 가져오기
fetch_weather_agent = Agent(
    name="WeatherFetcher",
    model="gemini-2.5-flash",
    instruction="사용자 요청({query})에서 도시 이름을 찾아 현재 날씨를 알려주고, "
                  "'weather_report' 키로 출력하십시오.",
    description="날씨 정보를 가져옵니다.",
    output_key="weather_report"
)

# 2. 병렬 작업 2: 뉴스 가져오기
fetch_news_agent = Agent(
    name="NewsFetcher",
    model="gemini-2.5-flash",
    instruction="사용자 요청({query})의 주제에 대한 최신 뉴스 헤드라인을 요약하고, "
                  "'news_summary' 키로 출력하십시오.",
    description="뉴스 정보를 가져옵니다.",
    output_key="news_summary"
)

# 3. 병렬 작업 3: 주식 가격 가져오기 (모의)
fetch_stock_agent = Agent(
    name="StockFetcher",
    model="gemini-2.5-flash",
    instruction="사용자 요청({query})에서 회사 이름을 찾아 모의 주식 가격을 알려주고, "
                  "'stock_price' 키로 출력하십시오. (예: 'Google' -> $180)",
    description="주식 가격 정보를 가져옵니다.",
    output_key="stock_price"
)

# --- 워크플로우 제어 ---

# 4. 병렬 실행 에이전트 (Fan-Out)
# 이 에이전트는 위 3개의 에이전트를 동시에 실행합니다.
parallel_gatherer = ParallelAgent(
    name="InfoGatherer",
    description="날씨, 뉴스, 주식 정보를 동시에 가져옵니다.",
    sub_agents=[
        fetch_weather_agent,
        fetch_news_agent,
        fetch_stock_agent
    ]
)

# 5. 종합 에이전트 (Gather)
# 병렬 에이전트가 state에 쓴 모든 데이터를 읽어 하나의 응답으로 종합합니다.
summarizer_agent = Agent(
    name="SummarizerAgent",
    model="gemini-2.5-flash",
    instruction="다음 정보를 종합하여 사용자에게 최종 보고서를 작성하십시오:\n\n"
                  "**날씨 정보:**\n{weather_report}\n\n"
                  "**최신 뉴스:**\n{news_summary}\n\n"
                  "**주식 가격:**\n{stock_price}\n\n"
                  "만약 정보가 없다면 '정보 없음'으로 표시하십시오.",
    description="모든 정보를 종합합니다."
)

# 6. 루트 에이전트: SequentialAgent
# (병렬 실행 -> 종합)의 순차 파이프라인을 정의합니다.
root_agent = SequentialAgent(
    name="ParallelPipelineAgent",
    description="병렬로 정보를 수집한 후 순차적으로 요약합니다.",
    sub_agents=[
        parallel_gatherer,  # 1단계: 3개 에이전트 병렬 실행
        summarizer_agent    # 2단계: 결과 종합 (순차 실행)
    ]
)
```

## V. 다중 에이전트 아키텍처(MAS) 설계: 4가지 상호작용 패턴

작업이 더욱 복잡해지면, 단일 에이전트나 경직된 워크플로우만으로는 한계에 부딪힙니다. 이때 여러 전문 에이전트가 협력하는 다중 에이전트 시스템(MAS)이 필요합니다. ADK는 에이전트 간의 상호작용(조정 및 협력)을 위해 4가지 뚜렷하고 강력한 메커니즘을 제공합니다. 개발자는 작업의 복잡성과 에이전트 간 결합도(coupling) 요구사항에 따라 적절한 패턴을 선택해야 합니다.

### 패턴 1: 암묵적 상태 공유 (Implicit State Sharing)

*   **설명:**`WorkflowAgent` (Sequential, Loop, Parallel) 내의 하위 에이전트들이 공유 `InvocationContext`의 `session.state`를 통해 데이터를 주고받습니다.
    
*   **장점:** 가장 간단하고 기본적인 통신 방식입니다.
    
*   **단점:** 하위 에이전트 간의 결합도가 높아집니다 (모든 에이전트가 `state['generated_code']`와 같은 공유 상태 키 이름을 알아야 함).
    
*   **예시:** 섹션 IV의 `SequentialAgent` 코드 에서 `code_writer_agent`가 `state['generated_code']`에 쓰고 `code_reviewer_agent`가 이를 읽는 방식입니다.
    

### 패턴 2: LLM 기반 동적 위임 (LLM-Driven Delegation)

*   **설명:**`WorkflowAgent`의 결정론적 로직과 달리, 상위 `LlmAgent`가 LLM의 추론을 사용하여 여러 `sub_agents` 중 _어떤_ 에이전트를 호출할지 동적으로 결정(라우팅)합니다. 이는 '지능형 위임(intelligent delegation)' 또는 '자동 라우팅(auto flow)'이라고도 합니다.
    
*   **장점:** 유연하고 적응력이 뛰어난 워크플로우를 만들 수 있습니다.
    
*   **단점:** LLM의 결정에 의존하므로 100% 결정론적이지 않으며, 라우팅을 위한 프롬프트 엔지니어링이 필요합니다.
    
*   \*\*예시 (개념 코드):\*\*의 'Weather Bot Team' 예제. 상위 에이전트(코디네이터)가 사용자의 의도를 파악하여 적절한 전문가(하위 에이전트)에게 작업을 위임합니다.
    

Python

```
# weather_team/agent.py (개념 코드)
from google.adk.agents import Agent
# (get_current_time 도구가 정의되어 있다고 가정)
# from my_tools import get_current_time 

# 하위 에이전트 1: 인사 전문가
greeter_agent = Agent(
    name="GreeterAgent",
    model="gemini-2.5-flash",
    instruction="사용자에게 친절하게 인사하십시오. 대화의 시작과 끝을 담당합니다.",
    description="인사말(예: 안녕, 하이) 및 작별 인사(예: 잘 가, 바이)를 처리합니다."
)

# 하위 에이전트 2: 날씨 전문가 (도구 사용)
weather_lookup_agent = Agent(
    name="WeatherLookupAgent",
    model="gemini-2.5-flash",
    instruction="사용자의 요청에 따라 'get_current_time' 도구를 사용해 날씨를 찾습니다.",
    description="특정 도시의 날씨 정보(예: '런던 날씨 어때?')를 조회합니다.",
    tools=[get_current_time] 
)

# 루트 에이전트: 동적 라우터 (Coordinator/Dispatcher)
root_agent = Agent(
    name="CoordinatorAgent",
    # 라우팅은 복잡한 작업이므로 Pro 모델 사용 권장
    model="gemini-2.5-pro-preview-03-25", 
    instruction="""당신은 요청을 가장 적절한 전문가에게 위임하는 팀의 코디네이터입니다.
    당신에게는 두 명의 전문가(하위 에이전트)가 있습니다.
    
    1. 'GreeterAgent': 인사 및 작별 인사를 처리합니다.
    2. 'WeatherLookupAgent': 도시의 날씨를 조회합니다.

    사용자의 요청을 분석하여 다음 규칙에 따라 작업을 위임하십시오:
    - 사용자가 '안녕', '하이', '반가워' 등 인사를 하면 'GreeterAgent'에게 작업을 위임하십시오.
    - 사용자가 '날씨', '기온', '런던', '파리' 등 날씨 관련 질문을 하면 'WeatherLookupAgent'에게 위임하십시오.
    - 사용자가 '잘 가', '바이' 등 작별 인사를 하면 'GreeterAgent'에게 위임하십시오.
    - 그 외 일반적인 대화(예: '넌 누구니?')는 당신이 직접 응답하십시오.
    """,
    # 이 하위 에이전트들이 LLM의 동적 위임 대상이 됩니다.
    sub_agents=[
        greeter_agent,
        weather_lookup_agent
    ]
)
```

### 패턴 3: `AgentTool`을 통한 명시적 로컬 호출 (Explicit Local Invocation)

*   **설명:**`AgentTool` 클래스를 사용하여 _다른 에이전트 전체_ 를 상위 에이전트의 _단일 도구_ 로 래핑할 수 있습니다.
    
*   **장점:** 강력한 캡슐화(Encapsulation). 상위 에이전트는 하위 에이전트가 내부에 어떤 프롬프트나 도구(예: `google_search`)를 사용하는지 알 필요가 없습니다. 그저 'search\_tool'이라는 도구를 호출할 뿐입니다.
    
*   **\[실행 가능 코드 7\] `AgentTool` 사용 예제:**
    

Python

```
# agent_as_tool/agent.py
from google.adk.agents import Agent
from google.adk.tools import google_search
# AgentTool 클래스를 임포트합니다.
from google.adk.tools.agent_tool import AgentTool 

# 1. 하위 에이전트 정의 (Google 검색 전문가)
# 이 에이전트는 내부에 'google_search' 도구를 가지고 있습니다.
search_agent = Agent(
    model="gemini-2.5-flash",
    name="search_agent",
    instruction="당신은 Google 검색 전문가입니다. 받은 쿼리를 그대로 사용하여 Google 검색을 수행합니다.",
    description="Google 검색을 사용하여 최신 실시간 정보를 얻습니다.",
    tools=[google_search],
)

# 2. AgentTool로 하위 에이전트 래핑 (핵심)
# 'search_agent'가 이제 'search_tool'이라는 이름의 *도구*가 됩니다.
search_tool = AgentTool(search_agent)
# (참고: AgentTool은 기본적으로 래핑된 에이전트의 이름과 설명을 상속받습니다. 
#  즉, 이 도구의 이름은 'search_agent'가 됩니다.)

# 3. 루트 에이전트 정의
root_agent = Agent(
    model="gemini-2.5-flash",
    name="research_assistant",
    instruction="당신은 리서치 비서입니다. 최신 정보나 시사 이슈가 필요할 때, "
                  "'search_agent' 도구를 사용하여 웹에서 정보를 검색하십시오.",
    description="리서치 작업을 수행하고 필요시 웹 검색을 합니다.",
    
    # 래핑된 'search_tool'을 일반 도구처럼 전달합니다.
    tools=[search_tool]
)
```

### 패턴 4: `A2A` 프로토콜을 통한 명시적 원격 호출 (Explicit Remote Invocation)

*   **설명:**`A2A (Agent-to-Agent)` 프로토콜은 에이전트가 네트워크 경계(심지어 다른 데이터센터나 다른 프로그래밍 언어로 작성된 에이전트)를 넘어 서로 통신할 수 있게 해주는 표준입니다. 이는 MAS를 위한 마이크로서비스 아키텍처와 유사합니다.
    
*   **워크플로우:**
    
    1.  **노출(Exposing):** 원격 에이전트(예: `PrimeAgent`)는 `to_a2a()` 유틸리티를 사용하여 A2A 서버로 노출됩니다. 이 서버는 에이전트의 기능 명세서인 '에이전트 카드(Agent Card)'를 `.well-known/agent-card.json` 엔드포인트에 자동으로 게시합니다.
        
    2.  **소비(Consuming):** 메인 에이전트(예: `RootAgent`)는 `RemoteA2aAgent` 클래스를 사용하여 원격 서버의 '에이전트 카드 URL'을 읽어들여, 해당 원격 에이전트를 로컬 도구처럼 소비합니다.
        

***

*   **\[실행 가능 코드 8-A\] A2A 에이전트 노출 (Server - `prime_agent/agent.py`)**

_이 파일은 원격 서버 역할을 하며, 포트 8001에서 실행됩니다._

Python

```
# prime_agent/agent.py (원격 서버 측 코드)
from google.adk.agents import Agent
from google.adk.tools import FunctionTool
from google.adk.a2a.utils.agent_to_a2a import to_a2a
import uvicorn # 서버 실행을 위해

# 1. 원격에서 제공할 전문 도구 정의
def check_prime(num: int) -> dict:
    """
    주어진 숫자가 소수(prime number)인지 확인합니다.
    """
    print(f" check_prime(num={num}) 실행됨")
    if num < 2:
        return {"number": num, "is_prime": False, "message": "2보다 작습니다."}
    for i in range(2, int(num**0.5) + 1):
        if num % i == 0:
            return {"number": num, "is_prime": False, "factor": i}
    return {"number": num, "is_prime": True}

prime_tool = FunctionTool(func=check_prime)

# 2. 노출시킬 루트 에이전트 정의
# 이 에이전트는 'check_prime' 도구만 전문적으로 처리합니다.
root_agent = Agent(
    name="prime_agent", # A2A 경로에 사용될 이름
    model="gemini-2.5-flash",
    instruction="당신은 소수 판별 전문가입니다. 'check_prime' 도구를 사용하여 요청을 처리하십시오.",
    description="숫자가 소수인지 확인하는 전문 에이전트.",
    tools=[prime_tool]
)

# 3. to_a2a()를 사용하여 에이전트를 A2A 앱으로 변환 (핵심)
# 이 앱은 http://localhost:8001/a2a/prime_agent/.well-known/agent-card.json 에
# 에이전트 카드를 자동으로 생성하고 노출합니다.
a2a_app = to_a2a(root_agent, port=8001)

# 4. 서버 실행
# 이 파일을 직접 실행할 수 없으며, 터미널에서 uvicorn으로 실행해야 합니다.
# 터미널 명령어:
# uvicorn prime_agent.agent:a2a_app --host localhost --port 8001
```

***

*   \*\* A2A 에이전트 사용 (Client - `root_project/agent.py`)\*\*

_이 파일은 로컬 소비자 역할을 하며, `adk web` (포트 8000)으로 실행됩니다._

Python

```
# root_project/agent.py (로컬 소비자 측 코드)
from google.adk.agents import Agent
from google.adk.agents.remote_a2a_agent import RemoteA2aAgent
from google.adk.agents.remote_a2a_agent import AGENT_CARD_WELL_KNOWN_PATH

# 1. 원격 A2A 에이전트를 로컬 도구로 정의 (핵심)
# 원격 서버(localhost:8001)의 에이전트 카드 경로를 지정합니다.
PRIME_AGENT_URL = "http://localhost:8001"
PRIME_AGENT_NAME = "prime_agent"

prime_agent_tool = RemoteA2aAgent(
    name=PRIME_AGENT_NAME, # 로컬에서 사용할 이름
    description="네트워크를 통해 원격 A2A 서버에 연결하여 숫자가 소수인지 확인합니다.",
    # 원격 에이전트의 '에이전트 카드' URL을 지정합니다.
    agent_card=(
        f"{PRIME_AGENT_URL}/a2a/{PRIME_AGENT_NAME}{AGENT_CARD_WELL_KNOWN_PATH}"
    )
)

# 2. 루트 에이전트 정의
# 이 에이전트는 'prime_agent_tool'을 일반 로컬 도구처럼 취급합니다.
root_agent = Agent(
    model="gemini-2.5-flash",
    name="root_agent",
    instruction="""당신은 수학 조수입니다. 
    사용자가 소수 확인을 요청하면(예: '17은 소수야?'), 'prime_agent' 도구를 사용하십시오.
    그 외의 계산은 당신이 직접 하십시오.
    """,
    # RemoteA2aAgent 객체를 도구 목록에 전달합니다.
    tools=[prime_agent_tool],
)

# --- 실행 방법 ---
# 1. (터미널 1) A2A 서버 (prime_agent) 실행:
#    uvicorn prime_agent.agent:a2a_app --host localhost --port 8001
#
# 2. (터미널 2) 로컬 소비자 (root_project) 실행 (상위 디렉토리에서):
#    adk web --port 8000
#
# 3. 브라우저(http://localhost:8000)에서 'root_project'를 선택하고
#    "17은 소수인가요?"라고 질문합니다.
```

***

## VI. 고급 적용 사례 및 패턴 (실행 코드 포함)

ADK는 기본 기능 외에도 RAG, 시각적 빌더 등 최신 AI 개발 패러다임을 지원합니다.

### \[실행 가능 코드 8\] RAG (Retrieval-Augmented Generation) 구현

앞서 언급했듯이, ADK는 RAG를 에이전트의 핵심 기능이 아닌, 확장 가능한 '도구'로 취급합니다.`VertexAiSearchTool`이나 일반 `rag_tool`을 사용하여 RAG 파이프라인을 에이전트에 통합할 수 있습니다.

다음은 `rag_tool`을 설정하는 `rag_agent/agent.py` 예제입니다. 실제 Vertex AI Search 데이터 스토어가 없는 사용자를 위해, 도구 설정에 실패할 경우 실행 가능한 모의(mock) 도구로 대체되는 예외 처리 로직을 포함했습니다.

Python

```
# rag_agent/agent.py
from google.adk.agents import Agent
from google.adk.tools import FunctionTool
import os

# 1. RAG 도구 설정 시도
try:
    # Vertex AI Search 도구를 직접 임포트 (권장 방식)
    from google.adk.tools import VertexAiSearchTool

    #.env 파일에 'VERTEX_AI_SEARCH_DATA_STORE_ID'가 설정되어 있어야 합니다.
    data_store_id = os.getenv("VERTEX_AI_SEARCH_DATA_STORE_ID")

    if not data_store_id:
        raise ValueError("VERTEX_AI_SEARCH_DATA_STORE_ID가 설정되지 않았습니다.")

    # Vertex AI Search 도구 인스턴스화
    my_rag_tool = VertexAiSearchTool(
        data_store_id=data_store_id,
        # LLM이 호출할 도구 이름과 설명을 정의합니다.
        name="company_document_search",
        description="회사의 내부 정책, 제품 가이드라인, HR 규정 등 비공개 문서에서 정보를 검색합니다."
    )
    print(f"VertexAiSearchTool이 성공적으로 로드되었습니다. (Data Store: {data_store_id})")

except (ImportError, ValueError, Exception) as e:
    # Vertex AI 설정이 없거나 실패할 경우, 모의 도구로 대체합니다.
    print(f"RAG 도구 설정 실패: {e}. 실행 가능한 모의(Mock) 도구를 사용합니다.")
    
    def mock_rag_tool(query: str) -> dict:
        """
        (모의 도구) 회사의 내부 정책, 제품 가이드라인, HR 규정 등 
        비공개 문서에서 정보를 검색합니다.
        """
        print(f" Query: {query}")
        return {"retrieved_chunk": f"'{query}'에 대한 모의 검색 결과: '회사의 휴가 정책은 연 15일입니다.'"}
    
    my_rag_tool = FunctionTool(
        func=mock_rag_tool,
        name="company_document_search", # VertexAiSearchTool과 동일한 이름 사용
        description="회사의 내부 정책, 제품 가이드라인, HR 규정 등 비공개 문서에서 정보를 검색합니다."
    )

# 2. RAG 에이전트 정의
root_agent = Agent(
    name="docs_assistant",
    model="gemini-2.5-flash",
    instruction="""당신은 회사 내부 문서 도우미입니다. 
    사용자가 회사 정책(예: '휴가 규정이 뭐야?')이나 가이드라인에 대해 물으면, 
    반드시 'company_document_search' 도구를 사용하여 관련 정보를 찾으십시오.
    검색된 내용을 바탕으로 사용자에게 정확하고 친절하게 답변을 생성하십시오.
    정보를 찾을 수 없으면 모른다고 솔직하게 답하십시오.
    """,
    # 설정된 RAG 도구(실제 또는 모의)를 에이전트에 전달
    tools=[my_rag_tool]
)
```

### 로우코드(Low-Code) 접근: ADK Visual Agent Builder

ADK v1.18.0부터 도입된 'Visual Agent Builder'는 코드를 직접 작성하지 않고도 복잡한 에이전트 아키텍처를 설계, 구성, 테스트할 수 있는 브라우저 기반 IDE입니다.

**핵심 기능:**

*   **시각적 워크플로우 디자이너:** 루트 에이전트, 하위 에이전트, 도구 간의 관계를 그래프(캔버스)로 시각화하여 복잡한 계층 구조를 한눈에 파악할 수 있습니다.
    
*   **구성 패널:** 폼 기반 인터페이스를 통해 에이전트 유형(예: `LlmAgent`, `SequentialAgent`), 모델, 시스템 프롬프트(Instructions), 도구 목록을 YAML 구문 오류 없이 편집할 수 있습니다.
    
*   **AI 어시스턴트:** "여러 도시의 날씨를 병렬로 조회한 후 요약하는 에이전트 팀을 만들어줘"와 같이 자연어로 요구사항을 설명하면, AI가 해당 아키텍처(예: `ParallelAgent`와 `SequentialAgent`의 조합)를 자동으로 생성해줍니다.
    
*   **실시간 테스트:** 빌드와 테스트가 동일한 인터페이스 내에서 이루어져 컨텍스트 전환 없이 빠른 반복 작업(iteration)이 가능합니다.
    

Visual Agent Builder의 가장 중요한 특징은 ADK의 '코드 중심' 철학을 포기하지 않았다는 점입니다. 사용자가 시각적으로 설계한 모든 워크플로우는 백그라운드에서 표준 ADK YAML 구성 파일로 즉시 변환됩니다. 이 YAML 파일은 언제든지 내보내기(export)하여 버전 관리 시스템(예: Git)에 체크인하고, 개발자가 직접 코드로 수정 및 확장(예: `OpenAPIToolset` 또는 `A2A` 통합 추가)할 수 있습니다.

이는 Visual Agent Builder가 단순한 '노코드' 도구가 아니라, 도메인 전문가, 프로덕트 매니저, 개발자 간의 협업을 위한 강력한 '브릿지' 역할을 함을 의미합니다. 비개발자가 시각적으로 프로토타입을 만들면, 개발자가 이를 이어받아 프로덕션 수준의 코드로 완성할 수 있습니다.

## VII. 프로덕션 배포 전략 (설정 예제 포함)

ADK는 로컬 테스트(`adk run`, `adk web`)에서 프로덕션 배포로 원활하게 전환할 수 있는 명확한 경로를 제공합니다. 핵심은 `adk api_server` 명령어를 사용하여 에이전트를 컨테이너화 가능한 API 서버로 실행하는 것입니다.

### 배포 준비: 컨테이너화 및 `Dockerfile`

ADK는 배포 불가지론을 따르므로, `Dockerfile`을 사용하여 에이전트를 컨테이너화하면 사실상 모든 클라우드 환경에 배포할 수 있습니다.

*   **\[설정 예제 1\] 일반적인 `Dockerfile`**

_이 `Dockerfile`은 `my\_agent`라는 에이전트 디렉토리를 컨테이너화합니다._

Dockerfile

```
# 1. 기본 이미지 (Python 3.11-slim 사용)
FROM python:3.11-slim

# 2. 작업 디렉토리 설정
WORKDIR /app

# 3. 시스템 의존성 설치 (필요시)
RUN apt-get update && apt-get install -y build-essential curl && \
    rm -rf /var/lib/apt/lists/*

# 4. Python 환경 변수
ENV PYTHONUNBUFFERED=1

# 5. ADK 및 종속성 설치
# (필요에 따라 gunicorn, uvicorn 등 추가)
RUN pip install --no-cache-dir \
    google-adk \
    python-dotenv \
    uvicorn

# 6. 에이전트 코드 복사
# (Dockerfile과 같은 위치에 'my_agent' 디렉토리가 있다고 가정)
COPY./my_agent /app/my_agent
COPY./.env /app/.env # API 키 포함

# 7. 프로덕션 포트 노출
EXPOSE 8000

# 8. ADK API 서버 실행 (핵심)
# adk web 대신 adk api_server를 사용합니다.
# '--host 0.0.0.0'은 컨테이너 외부에서 접근 가능하도록 합니다.
# 마지막 인수는 에이전트 디렉토리 경로입니다.
CMD ["adk", "api_server", "--host", "0.0.0.0", "--port", "8000", "./my_agent"]
```

### 배포 옵션 1: Google Cloud Run (서버리스)

Cloud Run은 컨테이너를 실행하는 완전 관리형 서버리스 플랫폼으로, 트래픽에 따라 자동으로 확장/축소됩니다. ADK CLI는 Cloud Run 배포를 위한 간소화된 명령어를 내장하고 있습니다.

*   **\[설정 예제 2\] `adk deploy cloud_run`**
    
    1.  Google Cloud CLI로 인증 및 프로젝트 설정:

Bash \`\`\` gcloud auth login gcloud config set project YOUR\_GCP\_PROJECT\_ID

```
    2.   배포를 위한 환경 변수 설정:

Bash  ```
export GOOGLE_CLOUD_PROJECT="YOUR_GCP_PROJECT_ID"
export GOOGLE_CLOUD_LOCATION="us-central1" # 예: 배포 리전
export AGENT_PATH="./my_agent" # 배포할 에이전트 디렉토리
export SERVICE_NAME="my-adk-agent-service"
```

```
3.   `adk` 명령어로 배포:
```

Bash \`\`\` adk deploy cloud\_run  
\--project=$GOOGLE\_CLOUD\_PROJECT  
\--region=$GOOGLE\_CLOUD\_LOCATION  
\--service\_name=$SERVICE\_NAME  
$AGENT\_PATH

```

이 명령어는 백그라운드에서 자동으로 `Dockerfile`을 빌드하고, 이미지를 Artifact Registry에 푸시하며, Cloud Run 서비스를 생성 및 배포하는 모든 과정을 처리합니다.

### 배포 옵션 2: Vertex AI Agent Engine (관리형)

Vertex AI Agent Engine은 ADK와 같은 프레임워크로 구축된 AI 에이전트를 배포, 관리, 모니터링 및 확장하기 위해 특별히 설계된 **완전 관리형 서비스**입니다.

이는 단순한 컨테이너 호스팅을 넘어, 에이전트 런타임, 품질 및 평가, 컨텍스트 관리(메모리), 보안 코드 실행(도구용)과 같은 핵심 서비스를 추상화하여 제공함으로써, 개발자가 인프라가 아닌 에이전트 로직에만 집중할 수 있도록 합니다.

*   **[설정 예제 3] `adk deploy agent_engine`**

    1.   GCP 인증 및 환경 변수 설정 (Cloud Run과 유사).

    2.   에이전트 빌드 아티팩트를 업로드할 GCS 버킷이 필요합니다.

    3.   `adk` 명령어로 Agent Engine에 배포:

Bash  ```
adk deploy agent_engine \
  --project="YOUR_GCP_PROJECT_ID" \
  --region="us-central1" \
  --staging_bucket="gs://YOUR_STAGING_BUCKET_NAME" \
  --display_name="My Production Agent" \
 ./my_agent
```

이는 ADK 에이전트를 위한 최상위급, 엔터프라이즈급 관리형 배포 옵션으로, 대규모 트래픽과 복잡한 운영 요구사항을 처리하는 데 최적화되어 있습니다.

## VIII. 결론: ADK를 통한 에이전트 개발의 미래

Google ADK는 단편적인 LLM 호출 라이브러리를 넘어, AI 에이전트 구축에 소프트웨어 엔지니어링의 엄격함, 제어권, 신뢰성을 도입하는 포괄적인 프레임워크입니다.

'코드 중심(Code-First)' 철학을 바탕으로 , ADK는 개발자가 `LlmAgent`의 비결정론적 창의성과 `WorkflowAgent`의 결정론적 신뢰성을 명확히 분리하고 조합할 수 있도록 설계되었습니다. 이를 통해 예측 가능하고 테스트 가능한 에이전트 시스템을 구축할 수 있습니다.

ADK의 진정한 강점은 에이전트 상호작용을 위해 명확하게 정의된 4가지 아키텍처 패턴을 제공하는 데 있습니다:

1.  **암묵적 상태 공유** (간단한 워크플로우용)
    
2.  **LLM 기반 동적 위임** (유연한 라우팅용)
    
3.  **`AgentTool`** (로컬 캡슐화용)
    
4.  **`A2A` 프로토콜** (원격 마이크로서비스용)
    

이러한 패턴은 개발자가 단순한 단일 에이전트부터 복잡한 마이크로서비스 기반의 다중 에이전트 시스템(MAS)에 이르기까지, 모든 규모의 아키텍처를 구현할 수 있는 완전한 제어권을 부여합니다.

`OpenAPIToolset`과 같은 강력한 도구 생태계 , Visual Agent Builder를 통한 협업 기능 , 그리고 Cloud Run 및 Vertex AI Agent Engine으로 이어지는 명확한 프로덕션 배포 경로 는 ADK가 단순한 프로토타이핑 도구를 넘어, 실제 엔터프라이즈 환경에서 신뢰할 수 있고 확장 가능한 차세대 AI 에이전트를 구축하기 위한 핵심 프레임워크임을 증명합니다.
