# [TIL] 셀프 호스팅 Firecrawl API를 AI Agent 커스텀 툴로 연동하기

**Date:** 2025-11-07

## 1. 목표

AI Agnet 에이전트가 유료 SaaS가 아닌, 내가 EC2에 직접 구축한 Firecrawl API를 `web_search_tool`로 사용하도록 설정한다.

## 2. 핵심 코드 (`tools.py`)

```python
import re
import logging
from firecrawl import FirecrawlApp

def web_search_tool(query: str) -> str:
    """
    Search the web for information using a self-hosted Firecrawl server.
    Args:
        query (str): The query to search the web for.
    Returns:
        A formatted string of search results with cleaned website content.
    """
    try:
        # API 키 대신, 직접 구축한 EC2 서버의 주소를 사용합니다.
        # 이 IP 주소는 EC2 재부팅 시마다 변경되므로,
        # AWS 콘솔에서 확인 후 수동으로 업데이트해야 합니다.
        app = FirecrawlApp(
            api_url="[http://13.238.253.214:3002](http://13.238.253.214:3002)"
        )
        
        # CrewAI 에이전트가 이해할 수 있도록 문자열로 가공하여 반환합니다.
        search_results = app.search(query=query, limit=3)
        
        if not search_results or not search_results.data:
            return "No results found."

        # (이하 결과 포매팅 로직)
        # ...
        
    except Exception as e:
        logging.error(f"Error in web_search_tool: {e}")
        return f"Error searching: {e}"
