# 세션 작업 요약

## 1. 프런트엔드 (Moonlight UI + ChatGPT 스타일)
- 좌측 사이드바: 챗봇 기록/파일 업로드 분리, 토글 버튼으로 접기/펼치기.
- 중앙: LLM이 생성한 보고서/체크리스트 표시, 검색 및 스크롤 가능, B_clean2 로고 클릭 시 초기화.
- 우측: 단일 ESG 챗봇 패널 (`ChatBotPanel`)에서 `/api/chat/stream` 스트리밍 사용.
  - react-markdown + remark-gfm으로 H1/H2/굵게/목록/코드 등을 ChatGPT 스타일로 렌더링.
  - 메시지 버블 최대 폭 80%, 스크롤 처리, Enter 전송/Shift+Enter 줄바꿈, 업로드 버튼 아이콘 등 적용.

## 2. 백엔드 (FastAPI + LangChain/LangGraph)
- `/api/agent/{type}` 하나에서 policy/regulation/risk/report/custom 모두 처리. regulation 전용 엔드포인트는 제거.
- `run_custom_agent`는 LangGraph 파이프라인(`src/workflows/custom_graph.py`)을 호출해 4개 모듈을 한 번에 실행하고 컨텍스트에 저장.
- `/api/chat`은 질문마다 custom 파이프라인을 자동 실행하고, “요약→근거→권고” 템플릿을 프롬프트에 포함. 멀티턴 저장을 위해 `chat_history`를 기록/프롬프트에 포함.
- `/api/chat/stream`은 SSE로 토큰을 실시간 전달하며, `/api/chat`과 동일한 템플릿/컨텍스트를 사용.

## 3. VectorDB & RAG
- `vector_db/esg_all.py` 실행으로 K-ESG 등 신규 문서 임베딩 완료 (`🚀 VectorDB 업데이트 완료`).
- 각 툴은 같은 Chroma 컬렉션을 사용하더라도 retriever 설정/프롬프트가 서로 다름 (정책/규제/리스크 용도별 최적화).
- 관련 설명은 `docs/retrieval_architecture.md`에 한국어로 정리.

## 4. LangGraph + 정리
- `src/workflows/custom_graph.py`: policy → regulation → risk → report 노드를 StateGraph로 구성.
- `docs/app_architecture_overview.md`: run_custom_agent가 LangGraph를 호출해 종합 결과를 만드는 구조를 설명.
- 불필요한 `(NEW)graph.py`, `(보관용)graph.py`, `schema.py` 삭제.

## 5. 문서화
- `docs/app_architecture_overview.md`: 백엔드/프런트 구조, 데이터 흐름, LangGraph 등을 설명.
- `docs/session_summary.md`: 세션 중 진행 사항 기록 용도 (본 문서).

## 6. TODO/주의사항
- `/api/chat/stream`에서 SSE 파싱 실패 시 “Error: 응답을 가져오지 못했습니다.”를 표시하므로, 프런트 로그/콘솔에서 오류 메시지를 확인 필요.
- LangGraph는 현재 run_custom_agent 내부에서만 호출되며, 추후 직접 사용해야 한다면 `run_langgraph_pipeline()`을 import해 `_pipeline.invoke()`로 실행할 수 있음.
- Vite dev 서버는 `http://localhost:5173`, FastAPI는 `uvicorn backend.main:app --host 0.0.0.0 --port 8000 --reload`로 실행.
- RAG 고도화: 여러 문서를 동시에 검색/조합하는 MultiRetriever 구조를 고려 (추후 작업).
- 멀티턴: `chat_history`를 프롬프트에 넣었지만 길이가 늘어나면 요약/압축이 필요.
