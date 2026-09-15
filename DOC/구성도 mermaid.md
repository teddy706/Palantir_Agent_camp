flowchart TB

    subgraph SRC["데이터 소스"]
        direction TB
        TACS_W["TACS 무선<br/>접근이력·사용자·장비"]
        TACS_L["TACS 유선<br/>접근이력·사용자·장비"]
        ITAM["ITAM<br/>서버 자산"]
        NMS["NMS<br/>네트워크 자산(5G스위치)"]
        AV["서버백신알람"]
        SI["SecureInsight(SMP)<br/>미조치리스트·진단현황"]
        LOG["서버보안설정 로그<br/>audit·bash_history·messages·secure"]
        PLAN["작업계획서<br/>(샘플 확보)"]
    end

    subgraph LAYER["Foundry 데이터 레이어"]
        direction TB
        INFRA["인프라 상태 레이어<br/>TACS 장비목록 vs (ITAM ∪ NMS)<br/>→ 유령자산(Shadow IT) 식별"]
        SEC["보안 상태 레이어<br/>백신알람 + SecureInsight 결합 → 방어선/취약점 매핑"]
    end

    TACS_W --> CORE
    TACS_L --> CORE
    TACS_W -.장비목록.-> INFRA
    TACS_L -.장비목록.-> INFRA
    ITAM -->|서버 자산| INFRA
    NMS -->|네트워크 자산| INFRA
    AV --> SEC
    SI --> SEC
    LOG --> CORE
    PLAN -.아이디어 단계.-> INFRA

    CORE["TACS 명령어 중심 교차 감시 엔진<br/>(Cross-Correlation)<br/>누가·무슨 권한·무슨 명령어·상태 변화"]

    INFRA --> CORE
    SEC --> CORE

    CORE --> DETECT{"이상 징후 / 유령자산 탐지"}

    subgraph EXT["외부 진화 루프"]
        KISA["KISA 보안공지<br/>MITRE ATT&CK<br/>(파일 업로드)"]
        AIP["AIP<br/>공격 명령어 패턴 파싱"]
        SEEDLIST["금지명령어 블랙/화이트리스트<br/>이상유형·킬체인 시나리오<br/>(기존 로직 = 초기 시드)"]
        CANDIDATE["후보 룰<br/>(승인 대기)"]
        KISA --> AIP --> CANDIDATE
        SEEDLIST --> RULE
    end

    RULE["동적 룰 마스터"]
    RULE --> CORE

    DETECT -->|"이상 없음"| END1["모니터링 계속"]
    DETECT -->|"이상 발생"| JUDGE1

    subgraph LOOP["내부 진화 루프 (2단계 판정)"]
        direction TB
        JUDGE1["1차 판단: 장비 운영자<br/>현장 맥락 확인·소명"]
        JUDGE2["2차 판단: NW보안팀 담당자<br/>소명 타당성 검토·최종 확정<br/>(AIP 후보 룰 승인도 포함)"]
        JUDGE1 --> JUDGE2
        JUDGE2 -->|"정상"| WL["동적 화이트리스트 등록"]
        JUDGE2 -->|"실제위협"| ALERT["대응/에스컬레이션"]
        JUDGE2 -->|"예외"| EXCEPTION["예외 자산 관리"]
    end

    CANDIDATE -->|"승인 필요"| JUDGE2
    JUDGE2 -->|"승인"| RULE

    WL --> RULE
    WL --> RECHECK{"재검토 트리거?<br/>N회 반복 / 작업계획서 만료 / 자산·사용자 확산"}
    RECHECK -->|"해당"| JUDGE2
    RECHECK -->|"미해당"| MONITOR["지속 모니터링"]

    style CORE fill:#bee3f8,color:#1a365d
    style JUDGE2 fill:#fbd38d,color:#652b19
    style JUDGE1 fill:#feebc8,color:#7b341e
    style RULE fill:#c6f6d5,color:#22543d
    style DETECT fill:#fed7d7,color:#742a2a
    style INFRA fill:#e9d8fd,color:#44337a