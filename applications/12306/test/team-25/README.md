# Team 25 — 12306 Test Submissions

Primary assigned application: **12306** (`applications/12306`).

## Team members

| Name | GitHub | Role |
| --- | --- | --- |
| _TBD_ | TheToror | Team lead |
| _TBD_ | _TBD_ | _TBD_ |

## Submitted test cases

| # | Document | Type | Requirement | Summary |
| --- | --- | --- | --- | --- |
| 1 | [black-box-01.md](black-box-01.md) | Black-box | REQ-2.2.2, REQ-2.1.1 | A registered account cannot sign in with its own username and password when another account's email address is identical to that username |

## Layout

```text
applications/12306/test/team-25/
├── README.md
├── black-box-01.md
└── assets/
    ├── register_a.png
    ├── register_b.png
    ├── login_a_pa.png
    ├── login_a_pb.png
    ├── login_b_pa.png
    └── login_b_pb.png
```

## Environment used

- Application started from `applications/12306` on a freshly initialised database
  (`.data/` removed before the run, so every seed account and the business clock
  are in their initial state at 2026-07-21 00:00 Asia/Shanghai).
- Plain browser interaction; no automation script is required to reproduce the
  documented failure.
