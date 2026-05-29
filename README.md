# vibecodings
바이브코딩 리포지토리

** https://github.com/hugoMGSung/hugo-vibecodings **

# Ollama 

- ChatGPT같은 대규모 언어 모델(LLM)을 개인 PC나 서버 등 로컬 환경에서 무료로 쉽게 설치하고 
- Local 머신의 GPU 성능의 영향을 받음

## 사용법

### 설치

- powershell 설치 명령어
    `irm https://ollama.com/install.ps1 | iex`

![alt text](image-20.png)

- https://ollama.com/ 웹사이트 진입
- 다운로드 페이지에서 OS플랫폼에 맞춰서 다운로드

![alt text](image.png)

![alt text](image-1.png)

### 설치완료

![alt text](image-2.png)

#### 터미널 실행

- 설치 및 버전확인

    ```powershell
    > ollama --version
    ollama version is 0.21.1
    ```

- Codex 설치

    ```powershell
    > ollama launch codex
    Error: codex is not installed, install from https://developers.openai.com/codex/cli/
    ```

- Codex CLI setup - 이전 node 24에 설치했으나, node 20으로 downgrade하면서 다시 설치

    ```powershell
    > npm i -g @openai/codex
    > codex
    ╭──────────────────────────────────────────────╮
    │ >_ OpenAI Codex (v0.124.0)                   │
    │                                              │
    │ model:     gpt-5.4 medium   /model to change │
    │ directory: ~                                 │
    ╰──────────────────────────────────────────────╯

    Tip: New Use /fast to enable our fastest inference with increased plan usage.


    › Explain this codebase

    gpt-5.4 medium · ~
    ```

- 재설치

    ```powershell
    > ollama launch codex
    Select model for Codex: Type to filter...

    Recommended
    ▸ kimi-k2.6:cloud
        State-of-the-art coding, long-horizon execution, and multimodal agent swarm capability
        qwen3.5:cloud
        Reasoning, coding, and agentic tool use with vision
        glm-5.1:cloud
        Reasoning and code generation
        minimax-m2.7:cloud
        Fast, efficient coding and real-world productivity
        gemma4
        Reasoning and code generation locally, ~16GB, (not downloaded)
        qwen3.5
        Reasoning, coding, and visual understanding locally, ~11GB, (not downloaded)

    ↑/↓ navigate • enter select • ← back
    ```

#### 모델 선택

- GPU 성능에 영향을 받음
- LLM: 숫자b - 숫자가 크면 클수록 모델 용량 큼. 그래픽카드 성능도 좋아야 함

- 선택지 
    - kimi-k2.6:cloud - 코딩 우수, 설치안함, 바로 사용가능
    - qwen3.5:cloud - 코딩, 추론 균형형, 설치안함
    - gemma4 - 16G 로컬설치. 코딩 무난, 조금 무거움, 요구사항 큼
    - **qwen3.5** - 11G 로컬설치. 코딩 좋음, 안정적, gemma 보다 빠르고 부담적음. 추천

    ![alt text](image-3.png)

- 설치 진행

    ![alt text](image-4.png)

- 설치 완료

    ![alt text](image-5.png)

- 사용화면

    ![alt text](image-6.png)


### CLI 사용 

#### Ollama + Codex CLI 튜토리얼

1. 준비 확인

    ```powershell
    > ollama --version
    ```

2. 프로젝트 폴더 생성 - 네이밍 참...

    ```powershell
    > mkdir ollama-todo
    > cd ollama-todo
    ```

3. Codex 실행

    ```powershell
    > ollama launch codex
    ```

4. 명령 전달

    ```powershell
    FastAPI로 TODO API 만들어줘.
        - 파일명: main.py
        - 기능: 목록조회, 추가, 완료처리, 삭제
        - 실행방법도 같이 알려줘
    ```

    ![alt text](image-7.png)

    ![alt text](image-8.png)

    ![alt text](image-9.png)

5. 실행결과

    ```powershell
     FastAPI TODO API 생성 계획

    - SQLite 데이터베이스 사용으로 파일 없이 즉시 실행 가능
    - Pydantic 모델로 데이터 유효성 검증
    - CRUD 4 가지 엔드포인트 구현
    - 자동화된 API 문서 생성 (Swagger) 포함

    main.py 파일을 생성하여 아래 기능을 구현합니다:

    from fastapi import FastAPI, HTTPException
    from pydantic import BaseModel
    from typing import List, Optional
    from datetime import datetime
    import sqlite3
    import json
    import os

    app = FastAPI(title="TODO API", version="1.0.0")

    DATABASE_FILE = "todos.db"

    # TODO 모델 정의
    class Todo(BaseModel):
        title: str
        description: Optional[str] = None
        completed: bool = False

    class TodoResponse(Todo):
        id: int
        created_at: datetime
        completed_at: Optional[datetime] = None

    # 데이터베이스 설정
    def get_db_connection():
        conn = sqlite3.connect(DATABASE_FILE)
        return conn

    def init_db():
        conn = get_db_connection()
        cursor = conn.cursor()
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS todos (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                title TEXT NOT NULL,
                description TEXT,
                completed INTEGER DEFAULT 0,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                completed_at TIMESTAMP
            )
        ''')
        conn.commit()
        conn.close()

    # 초기화
    init_db()

    # 엔드포인트 구현
    @app.get("/todos")
    async def list_todos():
        conn = get_db_connection()
        cursor = conn.cursor()
        cursor.execute('SELECT * FROM todos ORDER BY created_at DESC')
        todos = cursor.fetchall()
        conn.close()

        return [
            {
                "id": todo[0],
                "title": todo[1],
                "description": todo[2],
                "completed": bool(todo[3]),
                "created_at": todo[4],
                "completed_at": todo[5]
            } for todo in todos
        ]

    @app.post("/todos")
    async def create_todo(todo: Todo):
        conn = get_db_connection()
        cursor = conn.cursor()
        cursor.execute('''
            INSERT INTO todos (title, description, completed)
            VALUES (?, ?, ?)
        ''', (todo.title, todo.description, todo.completed))
        todo_id = cursor.lastrowid
        created_at = datetime.now().isoformat()

        cursor.execute('''
            SELECT * FROM todos WHERE id = ?
        ''', (todo_id,))
        row = cursor.fetchone()
        conn.close()

        return {
            "id": row[0],
            "title": row[1],
            "description": row[2],
            "completed": bool(row[3]),
            "created_at": row[4],
            "completed_at": row[5]
        }

    @app.put("/todos/{todo_id}")
    async def update_todo(todo_id: int, todo: Todo):
        conn = get_db_connection()
        cursor = conn.cursor()

        cursor.execute('''
            UPDATE todos
            SET title = ?, description = ?, completed = ?
            WHERE id = ?
        ''', (todo.title, todo.description, todo.completed, todo_id))

        if cursor.rowcount == 0:
            conn.close()
            raise HTTPException(status_code=404, detail="TODO 를 찾을 수 없습니다")

        cursor.execute('''
            SELECT * FROM todos WHERE id = ?
        ''', (todo_id,))
        row = cursor.fetchone()
        conn.close()

        return {
            "id": row[0],
            "title": row[1],
            "description": row[2],
            "completed": bool(row[3]),
            "created_at": row[4],
            "completed_at": row[5]
        }

    @app.delete("/todos/{todo_id}")
    async def delete_todo(todo_id: int):
        conn = get_db_connection()
        cursor = conn.cursor()

        cursor.execute('DELETE FROM todos WHERE id = ?', (todo_id,))
        if cursor.rowcount == 0:
            conn.close()
            raise HTTPException(status_code=404, detail="TODO 를 찾을 수 없습니다")

        conn.close()
        return {"message": "TODO 가 삭제되었습니다"}

    @app.get("/todos/{todo_id}/toggle")
    async def toggle_todo(todo_id: int):
        conn = get_db_connection()
        cursor = conn.cursor()

        cursor.execute('''
            UPDATE todos
            SET completed = NOT completed,
                completed_at = CASE completed THEN NULL ELSE CURRENT_TIMESTAMP END
            WHERE id = ?
        ''', (todo_id,))

        if cursor.rowcount == 0:
            conn.close()
            raise HTTPException(status_code=404, detail="TODO 를 찾을 수 없습니다")

        cursor.execute('''
            SELECT * FROM todos WHERE id = ?
        ''', (todo_id,))
        row = cursor.fetchone()
        conn.close()

        return {
            "id": row[0],
            "title": row[1],
            "description": row[2],
            "completed": bool(row[3]),
            "created_at": row[4],
            "completed_at": row[5]
        }

    실행 방법:

    1. 가상환경 생성 (권장):

    python -m venv venv
    source venv/bin/activate  # Windows: venv\Scripts\activate

    2. 의존성 설치:

    pip install fastapi uvicorn

    uvicorn main:app --reload --host 127.0.0.1 --port 8000

    4. API 테스트:

    - 브라우저: http://127.0.0.1:8000/docs (Swagger UI)
    - 목록 조회: http://127.0.0.1:8000/todos

    추가 테스트:

    # TODO 추가 (curl 명령어)
    curl -X POST http://127.0.0.1:8000/todos \
        -H "Content-Type: application/json" \
        -d '{"title": "작업 완료", "description": "중요한 작업"}'

    # 완료 처리
    curl -X GET http://127.0.0.1:8000/todos/1/toggle

    모든 기능 구현 완료! 🎉
    ```

6. 실행방법 실행

    - 콘솔 창에 나온 가이드 대로 실행

7. 실행이 나쁘지는 않으나 get 결과 일치하지 않음


### VS Code 기본 Copilot Chat에 Ollama 붙이기

1. 버전확인

    ```powershell
    > ollama --version
    > code --version
    ```

    - Ollama 공식 문서 기준 Ollama 0.18.3+, VS Code 1.113+, GitHub Copilot Chat 확장 0.41.0+ 필요. 
    - 커스텀 모델 선택을 위해 GitHub 로그인은 필요, 유료 Copilot 계정은 필수 아님

2. qwen3.5 로컬 모델 준비
    - 이미 받았으면 패스

    ```powershell
    # 없으면 새로 받기
    > ollama pull qwen3.5
    # 테스트
    > ollama run qwen3.5
    >>> 하이
    ...
    >>> /bye
    ```

3. VS Code 연결 실행

    - 터미널에서 실행

    ```powershell
    > ollama launch vscode
    ```

    - 이 명령은 Ollama 모델을 VS Code의 Copilot Chat 모델 선택창에 붙이는 방식. 공식 문서에 안내하고 있음

4. 트러블슈팅

    ```powershell
    > ollama launch vscode
    Error: vscode is not installed, install from https://code.visualstudio.com
    ```

    ```powershell
    > ollama stop
    > ollama serve
    ```

    - 실행확인

    ```powershell
    > ollama serve
    Error: listen tcp 127.0.0.1:11434: bind: Only one usage of each socket address (protocol/network address/port) is normally permitted.
    ```

    ![alt text](image-10.png)

5. VS Code에서 Ollama 직접 추가
    - Ctrl + Shift + P > Chat : Manage Language Models 선택

    ![alt text](image-13.png)

    ![alt text](image-14.png)

    ![alt text](image-15.png)

    ![alt text](image-16.png)

6. GitHub Copilot Chat

    ![alt text](image-17.png)

    ![alt text](image-18.png)

    ```text
    ollama 폴더에 Express를 사용한 Node.js로 Rest API를 만들어줘
    ```

    ![alt text](image-19.png)

7. 결과
    - ChatGPT Codex 와 유사한 기능 동작 확인
    - 제대로 된 폴더나 파일은 생성 못함
    - 대신 출력된 내용대로 복사/붙여넣기 가능