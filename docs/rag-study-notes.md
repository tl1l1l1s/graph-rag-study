# 학습노트

## RAG (Retrieval-Augmented Generation)

### RAG 정의

RAG = 검색 증강 생성

- 외부 DB에서 정보 검색 (Retrieval)
- 답변 생성 (Generation)
- LLM 특징 : 이미 학습된 데이터만 활용
- 한계 : 특정 도메인을 학습하는 데 한계 존재
- LLM 재학습의 어려움 : 시간, 비용이 많이 든다 → RAG를 사용하면 비용, 시간 ↓

### RAG 구성 요소

#### 1. Retrieval (검색)

- 사용자 질문 (Query) → 연관성 검색 → 정답 후보 제공
- Vector Search 활용

#### 2. Augmentation (증강)

- 검색된 정보를 입력(prompt)에 추가

#### 3. Generation (생성)

- LLM이 최종 답변 생성

<br />

## 데이터베이스

### 데이터 소스

- 문서 → Vector 변환, DB에 저장
- 데이터 형태 : JSON/CSV, PDF/DOCX, SQL/NoSQL
- 위치에 따른 그룹화 데이터

### Vector 검색?

≠ 키워드 검색

- 수치의 집합 (리스트, 배열)
- 데이터를 수치화 → 기계도 이해할 수 있음
- 문장의 의미를 벡터로 변환하여 수치화로 비교

✨ 임베딩 모델을 활용하여 벡터로 변환
→ 의미 구조 기반, 유사한 근처에 배치

⇒ 의미를 수치적 비교 & 검색이 가능!!

<br />

## 벡터 검색

- Query 벡터화 & DB 벡터와 Query 비교
- 유클리드 거리, 코사인 유사도를 사용할 수도 있지만 복잡한 수식과정이 필요 ⇒ 랭체인 라이브러리를 사용

### 벡터 검색 과정

FAISS: Meta에서 개발한 대규모 벡터 검색 라이브러리

문서를 라이브러리 Document 객체로 변환<br />
↓<br />
문서 분할<br />
↓ <br />
FAISS 벡터 저장, 생성 & 벡터검색을 위한 retriever 생성<br />
↓<br />
질문 기반 문서검색 (retriever.invoke(q))<br />
↓<br />
검색된 결과 내용을 LLM에게 입력하여 답 출력<br />

### 벡터 저장소

#### FAISS

- 대규모 검색 가능, 인덱싱 O, GPU → 빠른 연산 속도
- 정확도를 위해 튜닝 필요, 메모리 사용량 높음
- 분산 환경 지원 제한적(단일 머신 최적화)
- 실시간 업데이트가 어려움(배치 처리 적합)
- 클라우드 네이티브 기능 부족

#### Chroma

- 오픈소스
- 빠르고 가벼움 → 로컬 개발, 프로토타이핑에 최적화
- 메타데이터 필터링 기능이 우수
- FAISS, Milvus 대비 대용량 처리 ↓
- 고급 분산, 확장성 면에서 제한 O

#### Milvus

- 분산형 벡터 DB로 여러 노드에 데이터 분산 저장&처리 → scale out을 통해 확장 가능
- 실시간, 대규모에 최적화
- SQL과 유사 인터페이스
- 운영, 설정 복잡
- 리소스 요구사항이 높으며 설치와 운영이 복잡

#### Weaviate

- 오픈소스
- 다양한 임베딩 모델과 연동 가능
- API 기반 개발 → 다양한 분야에서 유리
- GraphQL API를 지원하여 복잡한 쿼리를 사용 가능
- 다른 벡터 저장소에 비해 성능 ↓ (순수 벡터 검색 성능, 메모리 사용량 등)

#### Pinecone

- 클라우드 기반
- 완전 관리형 서비스
- 벤더 종속성 우려

<br />

## Vector RAG

- RAG + 벡터검색
- Embedding + 벡터저장소 검색
- 벡터 유사도 기반 검색, 단순 질문에 효과적
- 정형화 된 정보가 아닌 문맥 이해가 필요할 경우 저장

### 구현

- FAISS, Annoy, Weaviate
- Langchain, Llamaindex

1. pandas로 데이터 전처리
2. embedding으로 벡터로 변환
3. faiss로 저장소에 저장
4. retriever 생성, 질의 응답 시스템을 생성해 답변 진행

#### 웹사이트 데이터 이용 시

- chromadb : 벡터 DB
- beautifulsoup4 : HTML 텍스트 추출 도구
- requests : HTTP 요청을 위한 라이브러리
- WebBaseLoader : 웹 URL 텍스트 추출
- RecursiveCharacterTextSplitter : 작은 청크로 분할
- pypdf : pdf 처리
- PromptTemplate : Prompt 처리

#### 다양한 파일을 처리해야 하는 경우

1. 폴더 내 순회 `os.walk(path)`
2. 형식별 처리 메소드 구현한 것을 기반으로 파일 확장자에 맞게 처리
3. 이후 플로우 동일 (임베딩, retriever...)

<br />

## Graph RAG

- 관계 그래프 기반 추론, 복잡한 연관성 질문에 강력
- 네트워크 기반
- 다중 연결 정보 (다중 단계(multi-hop))
- 데이터 간의 관계를 고려해야 할 때, 의미론적 관게가 중요할 때
- 방향 그래프 : 데이터의 관계, 직관적 표현 O
- 무방향 그래프
  <br /><br />
- 연결 데이터 탐색 용이 → 추천 시스템
- 복잡 패턴, 네트워크 분석에 용이
- 구조화 정리 → 지식 그래프 (소셜 네트워크, 검색 엔진, 교통 등)
  - 웹 페이지 간 연결 구조를 분석하여 각 페이지의 중요도를 평가하는 방식으로 검색 엔진에도 사용됨

### Graph RAG 구현

데이터 간 연관성 고려가 필요하다면 **GraphRAG 필요**

- Neo4j, ArangoDB
- json-repair : 손상된 json 데이터 자동복구
- networkx : 그래프 데이터 다루는 툴 (노드, 에지를 분석)

#### NetworkX를 사용하는 경우

1. LLM 모델 호출
2. 랭체인을 사용해 문서를 그래프로 변환 (`LLMGraphTransformer`)
3. 필요한 관계만 볼 수 있도록 필터링 적용
4. NetworkX 그래프 사용하여 실제 데이터 구조로 변환
5. QA 체인 생성, 쿼리를 받아와 답변 진행

#### Neo4j를 사용하는 경우

- langchain-neo4j

1. db 연결
2. 새로 고침
3. 자연어 질문 → Cypher 쿼리 변환 (`allow-dangerous-requests=True`)

- pdf 불러오기 (로드, 텍스트 추출, 저장 후 분할)
- 사용자의 질문을 입력 받은 뒤 LLM을 사용해 prompt 기반 cypher 쿼리로 변환

<br />

## Neo4j

Graph 기반 DB

### 특징

- Cypher 쿼리언어
- ACID transaction
- 확장성

### 구조

- 개체(node)
- 속성 : 정보
- 관계(edge) : reference, similar, subsumes, related-to
- Merge: 중복일 경우, 이미 존재 시 동작 X `Create or update`

### SQL vs. Cypher

- SQL: `INSERT INTO` = Cypher: `CREATE`
- SQL: `SELECT` = Cypher: `MATCH`
- Cypher: `JOIN` X (그래프에서는 관계로 자동 연결)

<br />

## Copilot

- 시멘틱 인덱스
  - 데이터를 벡터 형태로 인덱싱
  - 의미, 맥락에 기반하여 유사성 파악
- 그래프 검색(Microsoft Graph API)
  - 데이터를 그래프 형태로 연결해 관계를 이해
  - e.g., 사용자, 문서, 이메일, 일정, 파일

## 평가 지표(Metrics)

- Retrieval Metrics

  - Precision@K
  - Recall@K
  - MRR

- Generation Metrics

  - BLEU
  - ROUGE
  - BERTScore

- End-to-end Metrics
  - 답변의 정확성
  - 관련성
  - 충실도

<br/>

## 미래

- 하이브리드 검색
  1. 초기 검색: Vector DB에서 의미적으로 유사한 문서 검색
  2. 관계 확장: Graph DB에서 관련 엔티티와 관계 탐색
  3. 컨텍스트 병합: 두 결과를 결합하여 풍부한 컨텍스트 생성
  4. 최종 생성: LLM에 전달하여 답변 생성
- CoT-RAG
- 멀티모달 RAG
- AI agent + RAG
- 강화 학습 기반 RAG

  - Retrieval Optimization
  - Response Generation

<br/>

## 기타

#### 청크 분할을 하는 이유?

- LLM의 입력 토큰 제한
- 컨텍스트 윈도우 초과 방지
- 검색 성능 향상

일반적으로 200-500 토큰이 적절

- Fixed-size chunking : 고정 크기로 분할 (간단하지만 문맥 손실 가능)
- Semantic chunking : 의미 단위로 분할 (더 정확하지만 복잡)
- Overlapping chunks : 청크 간 중복을 두어 문맥 보존

### Embedding 모델

- OpenAI Embeddings : 높은 품질, 유료
- Sentence-BERT : 오픈소스, 다국어 지원
- BGE (BAAI General Embedding) : 오픈소스, 우수한 성능
- Multilingual E5 : 다국어 지원 우수
