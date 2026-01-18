# search-core 구조 요약

```mermaid
flowchart TB
    subgraph Entry
        main["1. main.rs | tui::run_tui"]
        repls["1b. repls.rs | REPL 입력 루프"]
    end

    subgraph Parse
        command["2. command.rs | Command::parse"]
    end

    subgraph Search
        search_dir["3. search_dir.rs | search_stream"]
        searcher["4. searcher.rs | searcher"]
        matcher["5. matcher.rs | find_matches, extract_line_context"]
    end

    subgraph Data
        types["Data. types.rs | SearchOptions, MatchInfo"]
    end

    main --> search_dir
    repls --> command --> search_dir
    search_dir --> searcher --> matcher

    search_dir --> types
    searcher --> types
    matcher --> types
    repls --> types
```

## 흐름 요약
- TUI 모드: `main.rs`에서 `tui::run_tui()` 시작
- REPL 모드: `repls::run_repl()`이 입력을 받아 `Command::parse`로 해석
- 검색 파이프라인: `search_stream`(디렉토리 순회) -> `searcher`(파일 검색) -> `matcher`(바이트 매칭)
- 결과 타입: `MatchInfo`, `SearchOptions`는 `types.rs`
