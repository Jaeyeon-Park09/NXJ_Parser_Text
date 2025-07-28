# PDF 파싱 툴 (NXJ_Parser_Text)

**의약품 인허가 가이드라인 RAG 기반 QA 챗봇 개발을 위한 한국어 PDF 파싱 시스템**

---

## 주요 특징

### PDF 처리 기능
- **일반 텍스트 PDF**: 텍스트와 표 구조 파싱
- **이미지 포함 PDF**: 텍스트만 추출, 이미지 제외
- **스캔본 PDF**: 자동 감지하여 파싱 제외

### 파싱 세부 기능
- **폰트 기반 제목 계층** 자동 인식 및 마크다운 변환
- **복잡한 표 구조** 감지 및 Markdown 변환
- **문장별 청킹** 기능 (LlamaIndex SentenceWindowNodeParser 사용)
- **문장 위치 추적** 기능: 문장의 pg 추적 가능 (metadata)

---

## 시스템 아키텍처

### 모듈화된 설계

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   LogManager    │    │ PDFTypeDetector │    │ TableDetector   │
│                 │    │                 │    │                 │
│ • 진행률 바      │    │ • 스캔본 감지    │    │ • 표 구조 분석   │
│ • 로그 파일 관리 │    │ • 이미지 감지    │    │ • 경계박스 계산  │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │   PDFParser     │
                    │                 │
                    │ • 전체 조정     │
                    │ • 상태 관리     │
                    └─────────────────┘
                                 │
         ┌───────────────────────┼───────────────────────┐
         │                       │                       │
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│TextChunker      │    │MarkdownConverter│    │  DataClasses    │
│                 │    │                 │    │                 │
│ • 텍스트 청킹    │    │ • Markdown 변환  │    │ • TextBlock     │
│ • 페이지 매핑    │    │ • 표 변환       │    │ • 메타데이터    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### 클래스별 기능

| 클래스 | 책임 | 주요 기능 |
|--------|------|-----------|
| **LogManager** | 로깅 관리 | 진행률 바, 로그 파일 넘버링 |
| **PDFTypeDetector** | PDF 타입 감지 | 스캔본/이미지/텍스트 분류 |
| **TableDetector** | 표 구조 감지 | 위치 기반 격자 분석 |
| **TextChunker** | 텍스트 청킹 | LlamaIndex 기반 청킹 |
| **MarkdownConverter** | Markdown 변환 | 구조화된 텍스트 생성 |
| **PDFParser** | 메인 조정자 | 전체 파싱 흐름 제어 |

---

## 프로젝트 구조

```
NXJ_Parser_Text/
├── pdf_files/                    # 파싱 대상 PDF 파일 (사용자가 직접 생성 후 파일 첨부해야 함ㅍㅍㅎㅍㅎ )
├── output/                       # 파싱 결과 출력
│   └── *.json                   # 구조화된 메타데이터
├── logs/                         # 실행 로그 디렉토리
│   ├── parsing_run_001.log      # 자동 넘버링 로그
│   └── parsing_run_002.log
├── utils/                        # 모듈화된 유틸리티
│   ├── __init__.py
│   ├── data_classes.py          # 데이터 클래스
│   ├── logging_manager.py       # 로깅 관리
│   ├── pdf_detector.py          # PDF 타입 감지
│   ├── table_detector.py        # 표 감지
│   ├── text_chunker.py          # 텍스트 청킹
│   └── markdown_converter.py    # Markdown 변환
├── main_parser.py               # 메인 파싱 시스템
├── requirements.txt             # Python 의존성
└── README.md                    # 이 파일
```

---

## 설치 방법

### 1. 의존성 설치
```bash
pip install -r requirements.txt
```

## 2. 파싱 대상 PDF 첨부
pdf_files/ 폴더 생성 후, 파싱할 PDF 첨부

### 3. PDF 파싱 실행
```bash
python main_parser.py
```
혹은 main_parser.py 실행

### 3. 실시간 진행상황 확인 가능
```
(예시)
🚀 PDF 파싱 시작 - 7개 파일
==================================================

📄 [1/7] 파일명.pdf
----------------------------------------
[1/7] [███░░░░░░░░░░░░░░░░░] 14.3% - PDF 열기: 파일명.pdf
[2/7] [██████░░░░░░░░░░░░░░] 28.6% - PDF 타입 감지 중...
[3/7] [██████████░░░░░░░░░░] 42.9% - 텍스트 블록 추출 중...
[4/7] [█████████████░░░░░░░] 57.1% - 문서 구조 분석 중...
[5/7] [████████████████░░░░] 71.4% - Markdown 변환 중...
[6/7] [███████████████████░░] 85.7% - 텍스트 청킹 중...
[7/7] [████████████████████] 100.0% - 결과 생성 완료
  ✅ JSON: 파일명.json
  📊 블록: 1,202개
  📊 표: 13개
  🧩 청크: 160개
```

---

## 출력 형식

### JSON 구조 (메타데이터)
```json
{
  "file_name": "파일명.pdf",
  "total_pages": 29,
  "pdf_type": "mixed",
  "font_analysis": {
    "font_sizes_frequency": {...},
    "heading_sizes": [27.9, 20.0, 17.9]
  },
  "table_analysis": {
    "total_tables": 13,
    "tables_info": [...]
  },
  "text_blocks": [...],
  "markdown_content": "...",
  "chunks": [...],
  "chunking_analysis": {
    "total_chunks": 160,
    "avg_chunk_length": 245.6,
    "window_size": 3
  },
  "statistics": {
    "total_text_blocks": 1202,
    "heading_count": 213,
    "paragraph_count": 937,
    "list_item_count": 52,
    "table_cell_count": 807,
    "tables_detected": 13,
    "chunks_generated": 160
  }
}
```

### 청크 메타데이터 (페이지 정보 포함)
```json
{
  "chunk_id": "chunk_001",
  "text": "청크 텍스트 내용...",
  "metadata": {
    "source": "파일명.pdf",
    "chunk_index": 0,
    "window_size": 3,
    "char_count": 245,
    "word_count": 45,
    "page_number": 1,
    "window_text": "윈도우 텍스트...",
    "original_sentence": "원본 문장..."
  }
}
```

---

## 로그 시스템

### 상세 로깅
- **실행별 넘버링**: `parsing_run_001.log`
- **실시간 진행률**: 7단계 처리 과정
- **디버그 정보**: 페이지별, 블록별 상세 추적
- **에러 추적**: 실패 원인 상세 기록

### 진행상황 모니터링
```
📁 로그 파일: logs/parsing_run_001.log
⏱️  총 소요시간: 1.35초
✅ 성공: 6개
❌ 실패: 0개
```