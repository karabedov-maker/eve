# WORKLOG — журнал изменений общей работы агентов

> Новые записи добавляются СВЕРХУ. Формат: `## дата время — [агент] заголовок`.

## 2026-11-03 11:30 — [zcode] Раздельные SSH-дорожки закреплены железно

- По указанию Андрея: каждый агент — только своим ключом. Ключ ZCode удалён из общего блока github.com в ~/.ssh/config; для ZCode остался персональный алиас `github-zcode` (в AGENTS.md — инструкция и образец для новых агентов).
- Теперь ходить чужим ключом невозможно конфигурационно, а не по договорённости.

## 2026-11-03 11:00 — [zcode] Фикс атрибуции SSH-ключа (ходил через ключ Claude Code)

- Обнаружено (Андрей по «Last used» на GitHub): все git-операции ZCode аутентифицировались ключом `id_ed25519_github` (Claude Code) — ssh берёт первый сработавший ключ из config.
- Исправлено: в `~/.ssh/config` добавлен алиас `Host github-zcode` (только ключ ZCode); репо eve — `core.sshCommand` на ключ `id_ed25519_github_zcode`. Проверено: fetch/ssh теперь идут ключом ZCode.
- Вывод для агентов на этой машине: если нужен свой ключ — явный `core.sshCommand` или свой Host-алиас, порядок IdentityFile в общем блоке решает ssh за вас.

## 2026-11-03 10:00 — [zcode] GitHub-подключение завершено

- gh CLI авторизован (karabedov-maker, keyring), git_protocol → ssh. Видны все 18 репозиториев.
- Обнаружено: 17 приватных автобэкап-репо кода с VPS (`Код с VPS: /opt/…`), созданы 03.11 утром — вся ферма и сайты теперь зеркалируются на GitHub.
- Координация агентов: этот репозиторий (eve) — хаб, правила в AGENTS.md.

## 2026-11-03 09:30 — [zcode] Подключение ZCode к GitHub + этот хаб

- Подключён отдельный SSH-ключ ZCode (`id_ed25519_github_zcode`, аккаунт karabedov-maker), git-идентичность Andrey Karabedov (noreply).
- Создан координационный хаб: AGENTS.md (правила) + этот WORKLOG. Решение Андрея: GitHub — общая видимость изменений для ZCode / Codex / Claude Code / Евы.
- Установлен gh CLI v2.103.3 (авторизация — pending, инструкция у Андрея).
- Кратко о сделанном ранее на серверах (полные детали — /root/VPN-PIPELINE.md на Москве): Варшава EDIS встроена в failover (транспорт vless+ws+TLS:443 без SNI — DPI-лотерея EDIS), Telegram фермы ходит напрямую по IPv6 с авто-страховкой на VPN, 4 HAPP-профиля (CZ/PL × WiFi/LTE) у Андрея.

## 2026-11-18 19:10 — [zcode] ТСПУ-блок Варшавы: диагноз + v6-нога на vm754408 + доступы Zcode на все 4 VPS

- **ТСПУ-блок** (с ~18:05 МСК): v4 РФ→EDIS 192.36.39.17 дропается по TCP (ICMP проходит). Доказано tcpdump на vm746226: SYN Варшавы приходит, SYN-ACK Москвы уходит — не доходит. Фильтр адресный, на транзите RETN (AS57304/9002), неравномерный: masterhost.ru и яндекс с Варшавы открываются. Прага↔Варшава и IPv6 не затронуты.
- **Доступы Zcode**: отдельные ed25519-ключи на все 4 VPS (`msk-masterhost-zcode`, `prague-mchost`, `edis-warsaw-zcode` + обходы `edis-warsaw-relay`/`edis-warsaw-v6`, `mh2-zcode`), aliases в ~/.ssh/config. Пароли не нужны.
- **Опознан 4-й VPS** vm754408 («Москва VPN», вход HAPP): ogorod-kalendar.ru + REALITY/ws/xhttp на одном :443, выходы Прага и Варшава-через-Прагу.
- **Новая v6-нога** на vm754408: outbound `vless-pl-v6` (прямая IPv6 на Варшаву, тест 14.5 МБ/с против 12.6 через Прагу), тест-socks 127.0.0.1:1088, авто-failover `pl-leg-failover.py` (таймер 90 с, via-cz ⇄ v6, антифлап, валидация конфига). Варшава ip6tables: 443/v6 разрешён и для vm754408.
- Бэкапы: `config.json.bak.20261118-pre-v6leg` (vm754408), `rules.v6.bak.20261118-pre-v6leg` (Варшава), `VPN-PIPELINE.md.bak.20261118-pre-v6leg`. Полные детали — /root/VPN-PIPELINE.md, записи 18.11.

## 2026-11-18 20:00 — [zcode] Прага: «огородный» стек на 443 + прямые профили CZ Direct

- По идее Андрея (russianpraga.ru как маска в стиле ogorod) построен полный стек: xray REALITY :443 (serverNames russianpraga.ru, dest→Caddy lo:8444) + ws/xhttp за Caddy-путями (lo:10010/10011).
- 3 HAPP-профиля «🇨🇿 CZ Direct | WiFi / LTE / LTE XHTTP» — телефоны ходят в Прагу напрямую, минуя masterhost (разгрузка концентрации на одном хостере).
- Проверено E2E: REALITY+vision → exit 178.208.75.190; ws через Caddy → тот же; сайт-маска через fallback 200/24мс; tproxy жив; нога ботов 8443 (alza.cz) не трогалась и работает.
- Нюансы: юнит xray@relay.service (шаблонный); Caddy admin off → только restart; ACME теперь HTTP-01 only; `xray x25519` в 26.10.31 меняет формат вывода.
- Бэкапы: relay.json.bak.20261118-pre-direct, Caddyfile.bak.20261118-pre-direct. Профили: prague_direct_profiles.txt. Детали: /root/VPN-PIPELINE.md (запись 18.11 вечер).

## 2026-11-18 23:30 — [zcode] Выход Варшавы Prefer-v6 + AAAA всем доменам + HAPP v2 (9 профилей)

- Warsaw freedom: domainStrategy=UseIPv6 (PreferIPv6/ForceIPv6 не подходят в 26.10.31: Force не откатывается на v4). Сайты с AAAA теперь уходят с 2a03:f80:48:8e28::1 (PL-гео), v4-only — откат.
- Всем 8 DNS-зонам на Moscow named добавлены AAAA @+www → 2a00:15f8:a003:f:1::1d (db.*.bak.20261118-pre-aaaa). Все свои сайты + ChatGPT проверены через PL-выход: 200/~0.6с.
- Грабли: отрицательный кеш AAAA в systemd-resolved Варшавы → resolvctl flush-caches. www.karabedov.ru — серт без SAN www (не VPN).
- Полный комплект HAPP v2: 9 профилей = 6 через «Москва VPN» (CZ/PL × WiFi/LTE/XHTTP) + 3 «CZ Direct» через Прагу (russianpraga.ru). Файл: happ_all_profiles_v2.txt.

## 2026-11-19 — [zcode] karabedov.ru: серт расширен SAN www

- Единственный серт без www (остальные 13 доменов — с www). Исправлено: nginx 443-блоку добавлен www в server_name (бэкап karabedov.ru.bak.20261119-pre-www), certbot --nginx certonly --expand → SAN теперь karabedov.ru + www.karabedov.ru.
- Проверено: локально 200 (0.13с), через PL-выход v6 200 (0.63с). Апекс не тронут.

## 2026-11-20 — [zcode] ogorod-kalendar.ru: AAAA в панели MCHost (через браузер)

- Андрей залогинился в my.mchost.ru во встроенном браузере; я добавил в зону /dnsZone/index/36167 две записи: `@ AAAA 2a00:15f8:a003:8:1::2c` и `www AAAA 2a00:15f8:a003:8:1::2c`.
- Проверено: dns1.mchost.ru и 1.1.1.1 отдают AAAA; сайт открывается по v6 (200/0.25с, remote ip = v6 «Москва VPN»). Маска огорода теперь dual-stack.
- Грабли UI: кнопка «Добавить запись» — это input.addRecord (клик через evaluate), форма — bootbox-модал (name/type/prio/content).

## 2026-11-20 16:05 — [zcode] russianwarsaw.ru: домен-маска Варшавы + v6-нога основная

- Зарегистрирован russianwarsaw.ru через MCHost (панель, 490₽ с баланса; зона 36963: A+AAAA @/www → Варшава; автопродление выключено).
- Ноги mh2→Варшава: serverName=russianwarsaw.ru (пин сохранён) — SNI↔IP↔DNS согласованы, для DPI это HTTPS к сайту.
- pl-leg-failover: приоритет сменён, активная PL-нога теперь прямая v6 (exit 2a03:f80:48:8e28::1), через Прагу — запасная.
- Проверки: обе ноги E2E OK, PL-rule=vless-pl-v6, failover-журнал чист. Бэкап config.json.bak.20261120-pre-sni.
