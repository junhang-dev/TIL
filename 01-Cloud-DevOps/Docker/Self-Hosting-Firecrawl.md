# [TIL] EC2 t3.small에서 Docker Compose로 오픈소스(Firecrawl) 셀프 호스팅하기

**Date:** 2025-11-07

## 1. 목표

* 유료 SaaS(Firecrawl)의 토큰/호출 제한과 구독 비용을 피한다.
* AWS EC2에 오픈소스를 직접 배포하여, 내 AI 에이전트의 `web_search_tool`을 위한 무제한 커스텀 API 엔드포인트를 구축한다.

## 2. 인프라 구축 (최초 1회)

1.  **EC2 인스턴스 생성:**
    * `AMI`: Ubuntu 22.04 LTS
    * `유형`: `t3.small` (2GB RAM).
    * **학습:** `t3.micro` (1GB RAM)로 시도했으나, `firecrawl-playwright-service` 등 여러 컨테이너를 동시에 빌드/실행하기에 메모리가 부족하여 `docker compose up`이 실패했다. `t3.small`로 업그레이드 후 해결.
2.  **키 페어 생성:**
    * `Firecrawl_server_key.pem` 생성 및 로컬 PC의 `C:\Users\내이름\.ssh\` 경로에 저장.
    * VS Code의 `Remote-SSH` 플러그인 설정에 이 키 경로를 지정.
3.  **보안 그룹 설정:**
    * `TCP 22` (SSH) - `내 IP`만 허용.
    * `TCP 3002` (Firecrawl API) - `0.0.0.0/0` (Anywhere) 허용.

## 3. 서버 환경 설정 (최초 1회)

1.  **SSH 접속:** VS Code를 통해 `ubuntu@[EC2_퍼블릭_IP]`로 접속.
2.  **Docker 설치:** `sudo apt update`, `Docker` 및 `Docker Compose` 설치.
3.  **소스 코드 복제:** `git clone https://github.com/mendableai/firecrawl.git`

## 4. 나의 개발 사이클 (시작/중지 루틴)

### A. 개발 재개 시

1.  **[AWS 콘솔]** `Firecrawl_server` 인스턴스 **[시작]**.
2.  **[AWS 콘솔]** 새로 할당된 **[퍼블릭 IPv4 주소]** 확인 및 복사 (예: `13.238.253.214`)
3.  **[로컬 VS Code]** Python `tools.py` 수정:
    ```python
    app = FirecrawlApp(api_url="[http://13.238.253.214:3002](http://13.238.253.214:3002)")
    ```
4.  **[로컬 VS Code]** SSH 접속 정보 (`.ssh/config`)의 `HostName`을 새 IP로 수정 후 EC2 서버 접속.
5.  **[EC2 터미널]** Firecrawl 디렉터리로 이동:
    ```bash
    cd firecrawl
    ```
6.  **[EC2 터미널]** `docker-compose.yml` 파일이 있는 위치에서 컨테이너 실행:
    ```bash
    docker compose up -d
    ```
    * `no configuration file provided` 오류: `docker-compose.yml` 파일이 없는 홈 디렉터리(`~`) 등에서 실행했기 때문. 반드시 `cd firecrawl`로 이동해야 함.

### B. 개발 종료 시 (비용 차단)

1.  **[EC2 터미널]** 컨테이너 중지 및 **삭제**:
    ```bash
    docker compose down
    ```
    * `up -d`로 켠 것은 `down`으로 끄는 것이 가장 깔끔하다. (컨테이너/네트워크 정리)
    * (이미지는 보존되므로 다음 `up` 때는 빌드 없이 빠름)
2.  **[EC2 터미널]** 접속 종료:
    ```bash
    exit
    ```
3.  **[AWS 콘솔]** `Firecrawl_server` 인스턴스 **[중지]**. (절대 '종료' 누르지 말 것)
