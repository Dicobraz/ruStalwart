# Патчи ruStalwart

База: `v0.16.27`. Один патч = один коммит в `ru/main`. Статусы: `PR` — открыт PR, `в ru/main` — слит,
`снят` — выброшен при обновлении (upstream исправил).

| № | Патч (PR) | Что исправляет | Сообщение в upstream | Статус | Снять, когда upstream исправит |
|---|---|---|---|---|---|
| 01 | [#1](https://github.com/Dicobraz/ruStalwart/pull/1) `ru/01-caldav-freebusy-acl` | CalDAV free/busy через outbox (RFC 6638) отдавал пустой VFREEBUSY, если календарь расшарен только с правом «free/busy» (`read-free-busy` / `mayReadFreeBusy`); требовалось ещё чтение событий. RFC 4791 §6.1.1. REPORT `free-busy-query` по адресу чужого календаря с таким правом по-прежнему 403 — не исправлено (не локально) | — (заполнить: тема на support.stalw.art) | PR | да |
| 03 | [#3](https://github.com/Dicobraz/ruStalwart/pull/3) `ru/03-email-query-text` | `Email/query` `text` из нескольких слов находил письмо, только если все слова в одном поле; «Re:»/«Fwd:» в `subject`/`text` не находили ничего (тема индексируется без префикса). RFC 8621 §4.4.1 | — | PR | да |
| 04 | [#2](https://github.com/Dicobraz/ruStalwart/pull/2) `ru/04-list-id-astext` | `header:List-Id:asText` всегда `null` в `Email/get`/`Email/parse`. RFC 8621 §4.1.2.2 | — | PR | да |
| 05 | [#4](https://github.com/Dicobraz/ruStalwart/pull/4) `ru/05-keywords-lowercase` | Ключевые слова JMAP хранились с исходным регистром: «Foo» и «foo» — два слова, `hasKeyword` зависел от регистра, `$MailFlagBit0..2` отдавались не в нижнем регистре. RFC 8621 §4.1.1. Новые слова сохраняются в нижнем регистре. Уже записанные не мигрируются: JMAP отдаёт их в нижнем регистре, `hasKeyword`, добавление и снятие работают без учёта регистра; IMAP FETCH показывает старый регистр, пока ключевые слова письма не заменят целиком. IMAP-клиенты теперь видят свои флаги в нижнем регистре | — | PR | да |

Номера — номера черновиков сообщений об ошибках; 02 и 06–14 патчей пока не имеют.
