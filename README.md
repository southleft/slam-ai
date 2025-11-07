# SLAM AI Mermaid Diagrams

## 1. High-Level System Architecture

```mermaid
graph TB
    User[👤 User] -->|Natural Language Query| Frontend[🖥️ Frontend<br/>React + Vite]
    Frontend -->|POST /chat/completions| Backend[⚡ Backend API<br/>FastAPI]
    Backend -->|visualization_hint| Router{Route by<br/>Metadata}
    
    Router -->|"auto"| SQL[Generate SQL Only]
    Router -->|"table"| SQLTable[Generate SQL + Table]
    Router -->|"bar_chart/line/scatter"| SQLChart[Generate SQL + Chart]
    Router -->|"map"| SQLMap[Generate SQL + Map]
    
    SQL --> SDK[🧠 SLAM AI SDK<br/>LangChain]
    SQLTable --> SDK
    SQLChart --> SDK
    SQLMap --> SDK
    
    SDK -->|Prompt Engineering| LLM[🤖 LLM Provider<br/>OpenAI/vLLM/Local]
    LLM -->|Response| Parser[📝 Output Parsers]
    Parser -->|Validated| ToolCalls[🔧 Tool Calls]
    
    ToolCalls -->|JSON Response| Frontend
    Frontend -->|Render| Viz[📊 Visualizations<br/>SQL/Table/Chart/Map]
    
    style User fill:#e1f5ff
    style Frontend fill:#fff4e1
    style Backend fill:#ffe1f5
    style SDK fill:#e1ffe1
    style LLM fill:#f5e1ff
    style Viz fill:#fff4e1
```

---

## 2. Intent to Action Mapping

```mermaid
flowchart TD
    Start([User Input]) --> Intent{Extract Intent<br/>from metadata}
    
    Intent -->|visualization_hint:<br/>'auto'| A1[Intent: SQL Only]
    Intent -->|visualization_hint:<br/>'table'| A2[Intent: Tabular Data]
    Intent -->|visualization_hint:<br/>'bar_chart'| A3[Intent: Bar Chart]
    Intent -->|visualization_hint:<br/>'line_chart'| A4[Intent: Line Chart]
    Intent -->|visualization_hint:<br/>'scatter_plot'| A5[Intent: Scatter Plot]
    Intent -->|visualization_hint:<br/>'map'| A6[Intent: Geographic Map]
    
    A1 --> SQL1[System Prompt:<br/>'You are a SQL expert...']
    A2 --> SQL2[System Prompt:<br/>'You are a SQL expert...']
    A3 --> SQL3[System Prompt:<br/>'You are a SQL expert...']
    A4 --> SQL4[System Prompt:<br/>'You are a SQL expert...']
    A5 --> SQL5[System Prompt:<br/>'You are a SQL expert...']
    A6 --> SQL6[System Prompt:<br/>'You are a SQL expert...']
    
    SQL1 --> Chain1[Text-to-SQL Chain]
    SQL2 --> Chain2[Text-to-SQL Chain]
    SQL3 --> Chain3[Text-to-SQL Chain]
    SQL4 --> Chain4[Text-to-SQL Chain]
    SQL5 --> Chain5[Text-to-SQL Chain]
    SQL6 --> Chain6[Text-to-SQL Chain]
    
    Chain1 --> Output1[Tool Call:<br/>display_code]
    
    Chain2 --> Table[Execute SQL]
    Table --> Output2A[Tool Call:<br/>display_code]
    Table --> Output2B[Tool Call:<br/>display_table]
    
    Chain3 --> VegaPrompt3[System Prompt:<br/>'Produce Vega-Lite<br/>specification...']
    Chain4 --> VegaPrompt4[System Prompt:<br/>'Produce Vega-Lite<br/>specification...']
    Chain5 --> VegaPrompt5[System Prompt:<br/>'Produce Vega-Lite<br/>specification...']
    
    VegaPrompt3 --> VegaChain3[Text-to-Vegalite Chain]
    VegaPrompt4 --> VegaChain4[Text-to-Vegalite Chain]
    VegaPrompt5 --> VegaChain5[Text-to-Vegalite Chain]
    
    VegaChain3 --> Output3A[Tool Call:<br/>display_code]
    VegaChain3 --> Output3B[Tool Call:<br/>render_vegalite]
    
    VegaChain4 --> Output4A[Tool Call:<br/>display_code]
    VegaChain4 --> Output4B[Tool Call:<br/>render_vegalite]
    
    VegaChain5 --> Output5A[Tool Call:<br/>display_code]
    VegaChain5 --> Output5B[Tool Call:<br/>render_vegalite]
    
    Chain6 --> GeoData[Generate GeoJSON]
    GeoData --> Output6A[Tool Call:<br/>display_code]
    GeoData --> Output6B[Tool Call:<br/>render_geospatial]
    
    Output1 --> Response([Return to Frontend])
    Output2A --> Response
    Output2B --> Response
    Output3A --> Response
    Output3B --> Response
    Output4A --> Response
    Output4B --> Response
    Output5A --> Response
    Output5B --> Response
    Output6A --> Response
    Output6B --> Response
    
    style Start fill:#e1f5ff
    style Intent fill:#ffe1e1
    style Response fill:#e1ffe1
```

---

## 3. Text-to-SQL Chain (Detailed)

```mermaid
sequenceDiagram
    participant U as User
    participant F as Frontend
    participant B as Backend
    participant CS as Column Selection<br/>(RAG)
    participant GQ as Golden Queries<br/>(Few-Shot)
    participant P as Prompt Builder
    participant LLM as LLM Provider
    participant Parse as Output Parsers
    participant DB as PostgreSQL
    
    U->>F: "Show top 5 products by revenue"
    F->>B: POST /chat/completions<br/>{visualization_hint: "bar_chart"}
    
    Note over B: Extract user query
    
    B->>CS: Find relevant columns
    activate CS
    CS->>CS: Embed query vector
    CS->>CS: Similarity search in schema
    CS-->>B: dataset_description<br/>(relevant columns)
    deactivate CS
    
    B->>GQ: Find similar examples
    activate GQ
    GQ->>GQ: Embed query vector
    GQ->>GQ: Search past Q&A
    GQ-->>B: golden_queries<br/>(few-shot examples)
    deactivate GQ
    
    B->>P: Build SQL prompt
    activate P
    Note over P: System: "You are a SQL expert..."<br/>+ dataset_description<br/>+ schema_description<br/>+ golden_queries<br/>+ user_query
    P-->>B: Complete prompt
    deactivate P
    
    B->>LLM: Send prompt
    activate LLM
    LLM->>LLM: Generate SQL
    LLM-->>B: "```sql<br/>SELECT...```"
    deactivate LLM
    
    B->>Parse: Extract SQL
    activate Parse
    Parse->>Parse: Remove markers
    Parse->>Parse: Validate syntax
    Parse->>DB: Test query (rollback)
    DB-->>Parse: ✓ Valid
    Parse-->>B: Validated SQL
    deactivate Parse
    
    B->>B: Create tool calls
    B->>F: {tool_calls: [{display_code}, {render_vegalite}]}
    F->>U: Render SQL + Chart
```

---

## 4. Prompt Engineering Flow

```mermaid
graph LR
    subgraph Input
        Q["User Query:<br/>Show top products<br/>by revenue"]
    end
    
    subgraph "Column Selection (RAG)"
        Q --> E1[Embedding Model]
        E1 --> V1["Query Vector:<br/>0.23, 0.45, ..."]
        V1 --> S1["Similarity Search<br/>in DB Schema"]
        S1 --> C1["Top Relevant Columns:<br/>sales.product_id<br/>sales.quantity<br/>sales.price<br/>products.name"]
    end
    
    subgraph "Golden Queries (Few-Shot)"
        Q --> E2[Embedding Model]
        E2 --> V2["Query Vector:<br/>0.23, 0.45, ..."]
        V2 --> S2["Similarity Search<br/>in Past Q&A"]
        S2 --> C2["Similar Examples:<br/>Q: top customers<br/>A: SELECT LIMIT"]
    end
    
    subgraph "Prompt Assembly"
        C1 --> PA[Prompt Builder]
        C2 --> PA
        Schema["Schema<br/>Description<br/>JSON"] --> PA
        Instructions["System<br/>Instructions"] --> PA
        Q --> PA
        
        PA --> FP[Final Prompt]
    end
    
    subgraph "LLM Processing"
        FP --> LLM[LLM]
        LLM --> R[SQL Response]
    end
    
    subgraph "Output Validation"
        R --> P1[Extract SQL]
        P1 --> P2[Validate Syntax]
        P2 --> P3[Test on DB]
        P3 --> Out[Validated SQL]
    end
    
    style Q fill:#e1f5ff
    style FP fill:#ffe1e1
    style Out fill:#e1ffe1
```

---

## 5. System Prompt Templates

```mermaid
graph TB
    subgraph "SQL Generation Prompt"
        SP1[System Prompt:<br/>'You are a SQL expert...']
        SP1 --> V1["{dataset_description}"]
        SP1 --> V2["{schema_description}"]
        SP1 --> V3["{golden_queries}"]
        SP1 --> V4["{additional_prompt}"]
        SP1 --> V5["{natural_language_query}"]
        
        V1 --> I1[Instructions:<br/>• Use full schema.table.column<br/>• Only SELECT relevant columns<br/>• Enclose in ```sql ... ```<br/>• No explanations]
        V2 --> I1
        V3 --> I1
        V4 --> I1
        V5 --> I1
        
        I1 --> OUT1[Output: SQL Query]
    end
    
    subgraph "Vega-Lite Generation Prompt"
        SP2[System Prompt:<br/>'Produce Vega-Lite v5.17.0<br/>specification...']
        SP2 --> V6["{natural_language_query}"]
        SP2 --> V7["{plot_type}"]
        SP2 --> V8["{column_names}"]
        SP2 --> V9["{vegalite_template}"]
        
        V6 --> I2[Instructions:<br/>• Data already filtered<br/>• Date → 'nominal'<br/>• Numeric → 'quantitative'<br/>• Valid JSON only<br/>• No placeholders]
        V7 --> I2
        V8 --> I2
        V9 --> I2
        
        I2 --> OUT2[Output: Vega-Lite Spec]
    end
    
    style SP1 fill:#ffe1e1
    style SP2 fill:#ffe1e1
    style OUT1 fill:#e1ffe1
    style OUT2 fill:#e1ffe1
```

---

## 6. Error Recovery & Auto-Repair

```mermaid
flowchart TD
    Start[LLM Response] --> Extract[Extract SQL]
    Extract --> Validate{"Validate<br/>Syntax"}
    
    Validate -->|✓ Valid| TestDB[Test on Database]
    Validate -->|✗ Invalid| Error1[Syntax Error]
    
    TestDB -->|✓ Success| Return[Return SQL]
    TestDB -->|✗ Fail| Error2[Execution Error]
    
    Error1 --> Repair1{"Retry<br/>Count < Max?"}
    Error2 --> Repair2{"Retry<br/>Count < Max?"}
    
    Repair1 -->|Yes| Fix1["Build Repair Prompt:<br/>Above completion has<br/>invalid syntax<br/>Please fix"]
    Repair2 -->|Yes| Fix2["Build Repair Prompt:<br/>Query failed<br/>Please fix"]
    
    Repair1 -->|No| Fail[Raise Exception]
    Repair2 -->|No| Fail
    
    Fix1 --> LLM1[Send to LLM]
    Fix2 --> LLM2[Send to LLM]
    
    LLM1 --> Increment1[Increment Retry Count]
    LLM2 --> Increment2[Increment Retry Count]
    
    Increment1 --> Extract
    Increment2 --> Extract
    
    style Start fill:#e1f5ff
    style Return fill:#e1ffe1
    style Fail fill:#ffe1e1
    style Fix1 fill:#fff4e1
    style Fix2 fill:#fff4e1
```

---

## 7. Tool Call Generation

```mermaid
graph TB
    subgraph "Backend Processing"
        Intent[User Intent] --> Route{"Visualization<br/>Hint"}
        
        Route -->|auto| Gen1[Generate SQL Only]
        Route -->|table| Gen2[Generate SQL + Execute]
        Route -->|chart| Gen3[Generate SQL + Chart Spec]
        Route -->|map| Gen4[Generate SQL + GeoJSON]
    end
    
    subgraph "Tool Call Assembly"
        Gen1 --> T1["Tool Call 1:<br/>display_code<br/>language: sql"]
        
        Gen2 --> T2A["Tool Call 1:<br/>display_code"]
        Gen2 --> T2B["Tool Call 2:<br/>display_table<br/>columns and data"]
        
        Gen3 --> T3A["Tool Call 1:<br/>display_code"]
        Gen3 --> T3B["Tool Call 2:<br/>render_vegalite<br/>schema and mark"]
        
        Gen4 --> T4A["Tool Call 1:<br/>display_code"]
        Gen4 --> T4B["Tool Call 2:<br/>render_geospatial<br/>FeatureCollection"]
    end
    
    subgraph "Response Format"
        T1 --> R1["ChatCompletion:<br/>tool_calls array"]
        T2A --> R2["ChatCompletion:<br/>tool_calls array"]
        T2B --> R2
        T3A --> R3["ChatCompletion:<br/>tool_calls array"]
        T3B --> R3
        T4A --> R4["ChatCompletion:<br/>tool_calls array"]
        T4B --> R4
    end
    
    subgraph "Frontend Rendering"
        R1 --> F1[Render: SQL Code Block]
        R2 --> F2A[Render: SQL Code Block]
        R2 --> F2B[Render: Data Table]
        R3 --> F3A[Render: SQL Code Block]
        R3 --> F3B[Render: Vega-Lite Chart]
        R4 --> F4A[Render: SQL Code Block]
        R4 --> F4B[Render: Leaflet Map]
    end
    
    style Intent fill:#e1f5ff
    style Route fill:#ffe1e1
    style F1 fill:#e1ffe1
    style F2A fill:#e1ffe1
    style F2B fill:#e1ffe1
    style F3A fill:#e1ffe1
    style F3B fill:#e1ffe1
    style F4A fill:#e1ffe1
    style F4B fill:#e1ffe1
```

---

## 8. Complete Data Flow

```mermaid
flowchart LR
    subgraph User Layer
        U[👤 User Input]
    end
    
    subgraph Frontend Layer
        UI[Chat Interface]
        Metadata[Add Metadata:<br/>visualization_hint]
    end
    
    subgraph Backend Layer
        API[FastAPI Endpoint]
        Router[Intent Router]
    end
    
    subgraph SDK Layer
        ColumnRAG[Column Selection<br/>RAG]
        GoldenRAG[Golden Queries<br/>RAG]
        PromptBuilder[Prompt Builder]
        LLMClient[LLM Client]
        Parsers[Output Parsers]
    end
    
    subgraph External Services
        LLM[(LLM Provider)]
        DB[(PostgreSQL)]
    end
    
    subgraph Response Layer
        Tools[Tool Calls]
        Render[Visualizations]
    end
    
    U --> UI
    UI --> Metadata
    Metadata --> API
    API --> Router
    
    Router --> ColumnRAG
    Router --> GoldenRAG
    ColumnRAG --> PromptBuilder
    GoldenRAG --> PromptBuilder
    
    PromptBuilder --> LLMClient
    LLMClient --> LLM
    LLM --> LLMClient
    LLMClient --> Parsers
    
    Parsers --> DB
    DB --> Parsers
    Parsers --> Tools
    
    Tools --> Render
    Render --> U
    
    style U fill:#e1f5ff
    style LLM fill:#f5e1ff
    style DB fill:#ffe1f5
    style Render fill:#e1ffe1
```

---

## Usage Notes

These diagrams can be rendered in:
- GitHub/GitLab (native support)
- VS Code (with Mermaid extension)
- Online: https://mermaid.live/
- Documentation sites (GitBook, Docusaurus, etc.)

Each diagram shows a different aspect of how system prompts map user intent to actions and responses in the SLAM AI system.
