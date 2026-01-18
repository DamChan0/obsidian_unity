# search-core 구조 요약

```mermaid
flowchart TB
    subgraph Binary
        main[main.rs
(tui::run_tui)]
    end

    subgraph Library
        lib[lib.rs
re-exports]
        tui[tui.rs
TUI 상태/이벤트]
        repls[repls.rs
REPL 입력 루프]
        command[command.rs
Command::parse]
        search_dir[search_dir.rs
search_stream]
        searcher[searcher.rs
searcher]
        matcher[matcher.rs
find_matches
extract_line_context]
        types[types.rs
SearchOptions
MatchInfo]
    end

    main --> tui
    repls --> command
    repls --> search_dir
    repls --> types

    tui --> search_dir
    tui --> types

    search_dir --> searcher
    search_dir --> types

    searcher --> matcher
    searcher --> types

    lib --> repls
    lib --> search_dir
    lib --> searcher
    lib --> types
```

## 흐름 요약
- TUI 모드: `main.rs`에서 `tui::run_tui()` 시작
- REPL 모드: `repls::run_repl()`이 입력을 받아 `Command::parse`로 해석
- 검색 파이프라인: `search_stream`(디렉토리 순회) -> `searcher`(파일 검색) -> `matcher`(바이트 매칭)
- 결과 타입: `MatchInfo`, `SearchOptions`는 `types.rs`
