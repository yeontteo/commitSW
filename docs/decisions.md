# 결정 현황

> 마지막 갱신: 2026-11-04 (기획 단계 완료, 중간 보고서 제출)
> 결정이 바뀌면 이 문서의 해당 항목을 고치고, 맨 아래 **변경 이력**에 한 줄을 추가한다.

---

## 1. 서비스

| 항목 | 내용 |
|---|---|
| 서비스명 | commitSW |
| 컨셉 | 개발자가 코드 변경을 커밋하듯, 학생들이 휴강·과제·시험·공지를 캘린더에 "커밋"해 함께 기록하고 공유한다 |
| 한 줄 소개 | 혼자 들어도, 결석해도 공지는 놓치지 않는 소프트웨어학부 전용 공유 캘린더 |
| 사용자 | 소프트웨어학부 본전공 재학생, 전과생, 복수전공생 (부전공 제외) |
| 형태 | 반응형 웹 + PWA |

## 2. 기능 요구사항 (MVP)

| ID | 기능 | 요구사항 |
|---|---|---|
| F1 | 회원가입·로그인 | 학번 + 비밀번호 + 복구용 이메일 가입. 본전공은 학번 규칙 + 학부 명단 대조 + 중복 가입 차단. 명단에 없는 전과·복수전공생은 증빙 첨부 후 관리자 승인. 복구용 이메일 인증 및 비밀번호 재설정 |
| F2 | 수강 과목 선택 | 이번 학기 과목 중 수강 과목 선택, 해당 과목 일정만 표시 |
| F3 | 메인 캘린더 | 월간/주간 전환, 유형별 색상, 우측 상단 일정 등록 버튼, 종일 일정(휴강·공지)과 시각 일정(과제·시험) 구분 |
| F4 | 일정 등록 | 과목, 유형, 제목, 설명, 출처(LMS/구두/이메일/기타 + 링크·캡처), 일시(과제·시험은 시각까지). 저장 직전 같은 과목·같은 날짜 기존 일정 표시. 제목·설명은 첫 등록자만 작성·수정 |
| F5 | 일정 상세 | 구글 캘린더 상세 화면 참고. 과목, 유형, 제목, 설명, 출처, 작성자, 좋아요 수 표시. 댓글로 추가 정보·정정, 댓글 좋아요, 최다 좋아요 댓글 상단 노출 |
| F6 | 신고·문의 | 틀린 일정·부적절한 댓글 신고. 명의 도용·비밀번호 문제는 비로그인 문의 가능 |
| F7 | 관리자 | 전과·복수전공 승인/거절(사유), 신고 처리(숨김, 사용자 정지), 문의 처리 |
| - | MVP 이후 | 구글 캘린더 내보내기 |

## 3. 인증

| 단계 | 처리 |
|---|---|
| 가입 (본전공) | 학번 규칙 `^20\d{2}041\d{3}$` 검사 → `student_roster` 존재 확인 → 이미 가입된 학번인지 확인 → 복구용 이메일로 인증 메일 → `member_type=primary`, `status=active` |
| 가입 (전과·복수전공) | 명단에 없음 안내 → 유형 선택 + 증빙 이미지 첨부 → 이메일 인증 → `status=pending` → 관리자 승인 시 `active`, 거절 시 사유 안내 |
| 로그인 | 학번 + 비밀번호 입력 → 서버에서 학번으로 이메일 조회 → Supabase Auth 로그인 |
| 비밀번호 찾기 | Supabase 기본 재설정 메일 (복구용 이메일로 발송) |
| 명의 도용 | 이미 가입된 학번이면 비로그인 상태로 관리자 문의 |

- 복구용 이메일은 Supabase `auth.users`의 이메일로 사용한다 (학교 이메일 아님).
- 학번 규칙은 041 코드만 사용한다.
- 명단 대조와 가입 처리는 서버에서만 실행한다.

## 4. 일정 규칙

| 유형 | 값 | 일정 형태 |
|---|---|---|
| 휴강 | `cancel` | 종일 |
| 과제 | `assignment` | 시각 (마감 시각) |
| 시험 | `test` | 시각 |
| 공지 | `notice` | 종일 |

- 종일·시각 구분은 유형으로 자동 결정한다. 등록 폼에 "하루 종일" 선택지를 두지 않는다.
- 제목·설명은 첫 등록자만 작성·수정한다. 다른 사용자는 댓글로 보완한다.
- 일정 저장 직전에 같은 과목·같은 날짜의 기존 일정을 보여주고 중복 등록 여부를 확인한다.
- 유형별 색상은 미정.

## 5. 기술 스택

| 영역 | 선택 | 이유 |
|---|---|---|
| 프레임워크 | Next.js (React) | 화면과 서버 로직(명단 대조, 가입 처리)을 한 프로젝트에서 처리 |
| 스타일 | Tailwind CSS | 반응형 화면을 빠르게 구현 |
| 캘린더 UI | FullCalendar | 월간/주간 전환, 유형별 색상, 종일 일정 기본 제공 |
| 백엔드·DB·인증·파일 | Supabase | 이메일 인증, 비밀번호 재설정, Storage, RLS 기본 제공. 관계형 DB |
| 배포 | Vercel + GitHub | 푸시하면 자동 배포 |
| 형태 | 반응형 웹 + PWA | 앱을 따로 만들지 않고 모바일에서 앱처럼 사용 |

## 6. AI 워크플로우

| 단계 | 도구 | 방식 |
|---|---|---|
| 기획·요구사항 | Claude (claude.ai) | 문제 정의, 기능 명세, DB 설계, 시나리오 |
| 구현 | Claude Code (VS Code) | CLAUDE.md 작업 규칙에 따라 설계안 승인 후 구현 |
| 디버깅 | Claude Code | 재현 순서, 에러 로그, 시도한 내용을 정리해 원인 분석 |
| 리뷰·테스트 | Claude + Gemini Pro | 같은 코드·같은 프롬프트로 각각 리뷰받아 교차 검증 |

- 교차 검증: 공통 지적은 우선 수정, 단독 지적은 직접 확인 후 판단, 오탐은 기록해 모델별 특성 비교

## 7. DB 설계 (11개 테이블)

### student_roster: 본전공 재학생 명단
| 컬럼 | 타입 | 설명 |
|---|---|---|
| `student_id` | text, PK | 학번 |

- RLS: 일반 사용자 접근 전면 차단. 서버(service role)에서만 조회
- 입학년도는 학번 앞 4자리로 계산 (별도 컬럼 없음)

### profiles: 회원 정보
| 컬럼 | 타입 | 설명 |
|---|---|---|
| `id` | uuid, PK | `auth.users.id` 참조 |
| `student_id` | text, UNIQUE | 학번 (중복 가입 차단) |
| `nickname` | text | 화면 표시 이름 |
| `role` | `student` / `admin` | 권한 |
| `member_type` | `primary` / `transfer` / `double_major` | 본전공 / 전과 / 복수전공 |
| `status` | `pending` / `active` / `suspended` | 승인 대기 / 활성 / 정지 |
| `created_at` | timestamptz | 가입 시각 |

### verification_requests: 전과·복수전공 승인 요청
| 컬럼 | 타입 | 설명 |
|---|---|---|
| `id` | uuid, PK | |
| `user_id` | uuid | profiles 참조 |
| `member_type` | `transfer` / `double_major` | 신청 유형 |
| `proof_image` | text | 증빙 이미지 경로 (Storage, 관리자만 열람) |
| `status` | `pending` / `approved` / `rejected` | |
| `reject_reason` | text, null | 거절 사유 |
| `handled_by` | uuid, null | 처리한 관리자 |
| `created_at` / `handled_at` | timestamptz | 신청·처리 시각 |

- 증빙 이미지는 승인·거절 처리 후 삭제

### courses: 과목
| 컬럼 | 타입 | 설명 |
|---|---|---|
| `id` | uuid, PK | |
| `semester` | text | 예: `2026-2` |
| `name` | text | 과목명 |
| `professor` | text | 교수명 |
| `section` | text | 분반 |

- UNIQUE (`semester`, `name`, `section`)

### enrollments: 수강 과목
| 컬럼 | 타입 | 설명 |
|---|---|---|
| `user_id` | uuid | profiles 참조 |
| `course_id` | uuid | courses 참조 |

- PK (`user_id`, `course_id`)

### events: 일정
| 컬럼 | 타입 | 설명 |
|---|---|---|
| `id` | uuid, PK | |
| `course_id` | uuid | courses 참조 |
| `type` | `cancel` / `assignment` / `test` / `notice` | 유형 |
| `title` | text | 제목 (작성자만 수정) |
| `description` | text | 상세 설명 (작성자만 수정) |
| `source_type` | `lms` / `verbal` / `email` / `other` | 출처 유형 |
| `source_url` | text, null | 출처 링크 |
| `source_image` | text, null | 출처 캡처 경로 (Storage) |
| `start_at` / `end_at` | timestamptz | 일시 (과제는 마감 시각) |
| `all_day` | boolean | 유형에서 자동 결정 (cancel·notice = true) |
| `created_by` | uuid | 첫 등록자 |
| `is_hidden` | boolean | 신고 처리로 숨김 |
| `created_at` / `updated_at` | timestamptz | |

- 인덱스 (`course_id`, `start_at`): 중복 일정 확인용

### comments: 댓글
| 컬럼 | 타입 | 설명 |
|---|---|---|
| `id` | uuid, PK | |
| `event_id` | uuid | events 참조 |
| `user_id` | uuid | 작성자 |
| `content` | text | 내용 |
| `is_hidden` | boolean | 신고 처리용 |
| `created_at` / `updated_at` | timestamptz | |

### event_likes / comment_likes: 좋아요
| 컬럼 | 설명 |
|---|---|
| `user_id` + `event_id` (또는 `comment_id`) | PK로 묶어 1인 1회 |
| `created_at` | |

- 좋아요 수는 저장하지 않고 집계 뷰로 계산. 최다 좋아요 댓글 상단 노출에도 사용

### reports: 신고
| 컬럼 | 타입 | 설명 |
|---|---|---|
| `id` | uuid, PK | |
| `reporter_id` | uuid | 신고자 |
| `event_id` / `comment_id` | uuid, null | 둘 중 정확히 하나만 값이 있음 (CHECK) |
| `reason` | text | 사유 |
| `status` | `pending` / `resolved` / `rejected` | |
| `handled_by` | uuid, null | 처리한 관리자 |
| `created_at` | timestamptz | |

### inquiries: 관리자 문의
| 컬럼 | 타입 | 설명 |
|---|---|---|
| `id` | uuid, PK | |
| `student_id` | text | 문의자 학번 |
| `contact_email` | text | 답변받을 이메일 |
| `category` | `impersonation` / `password` / `other` | |
| `message` | text | 내용 |
| `status` | `pending` / `resolved` | |
| `created_at` | timestamptz | 스팸 제한에 사용 |

- 비로그인 작성 허용, 같은 이메일의 짧은 시간 내 반복 작성 제한

## 8. 권한 (RLS)

| 대상 | 조회 | 작성 | 수정·삭제 |
|---|---|---|---|
| `student_roster` | 서버만 | 서버만 | 서버만 |
| `events` | active 사용자 | active 사용자 | 작성자, 관리자 |
| `comments` | active 사용자 | active 사용자 | 작성자, 관리자 |
| 좋아요 | active 사용자 | active 사용자 (본인) | 본인 |
| `verification_requests` | 본인(자기 신청), 관리자 | 본인 | 관리자 |
| `reports` | 관리자 | 로그인 사용자 | 관리자 |
| `inquiries` | 관리자 | 누구나 (비로그인 포함) | 관리자 |

- `is_hidden = true`인 항목은 관리자에게만 보인다.
- `pending`, `suspended` 사용자는 캘린더와 일정을 볼 수 없다.

## 9. 알려진 한계
- 타인의 학번으로 먼저 가입하는 명의 도용은 원천 차단이 어려워 관리자 문의로 대응한다.
- 정보의 정확성은 사용자 참여(출처, 좋아요, 댓글, 신고)에 의존한다.
- 전과·복수전공 승인은 학번의 형식이 달라 관리자 수작업 승인이 필요하다.

## 10. 미정 사항
- [ ] 과사무실 학번 명단 제공 여부 (미제공 시 허가받은 큐빅 동아리원 명단으로 시연)
- [ ] 일정 유형별 색상
- [ ] 구글 캘린더 내보내기 방식 (MVP 이후)
- [ ] 최종 발표일

---

## 변경 이력

| 날짜 | 주제 | 변경 전 → 변경 후 | 이유 |
|---|---|---|---|
| 2026-11-04 | 인증 | 학교 이메일 + 학번 규칙 → 학번 + 비밀번호 + 명단 대조 | 학교 이메일로는 학부 확인 불가 |
| 2026-11-04 | 인증 | 이메일 없음 → 복구용 개인 이메일 추가 | 비밀번호 분실 시 복구 수단 필요 |
| 2026-11-04 | 프론트엔드 | React → Next.js | 명단 대조를 서버에서 처리하기 위함 (브라우저 노출 방지) |
| 2026-11-04 | 공지 설명 | 사용자 누적 설명 → 첫 등록자 작성 + 댓글 보완 | 구조 단순화 |
| 2026-11-04 | AI 파이프라인 | 전 단계 Claude → 리뷰에 Gemini Pro 추가 | 교차 검증 |
| 2026-11-04 | DB | `admission_year` 삭제 | 학번 앞 4자리로 계산 가능 |
| 2026-11-04 | 일정 유형 | 휴강/과제/공지 → 시험 추가 | 시험 일정 필요 |
| 2026-11-04 | 사용자 | 본전공만 → 전과·복수전공 포함 (관리자 승인) | 전과해도 학번이 바뀌지 않아 명단·학번으로 구분 불가 |
| 2026-11-04 | DB | 10개 → 11개 테이블 (`verification_requests` 추가) | 승인 흐름 |
| 2026-11-04 | 구현 방식 | 계획 먼저 → 설계안 6단계 + CLAUDE.md 고정 | 선택 과정과 이유를 직접 확인하며 학습 |
| 2026-11-04 | 신뢰도 | 댓글 정정 + 신고 → 최다 좋아요 댓글 상단 노출 추가 | 첫 등록자만 수정 가능한 구조 보완 |
| 2026-11-04 | 일정 형태 | 사용자가 종일 여부 선택 → 유형별 자동 결정 | 입력 단순화 |
| 2026-11-04 | 학번 규칙 | 041·078 → 041만 | 사용자 판단 |
| 2026-11-04 | 시연 명단 | 더미 명단 → 허가받은 큐빅 동아리원 명단 | 실제 학번은 개인정보 |
