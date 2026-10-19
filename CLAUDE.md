# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Что это

Upptime (https://github.com/upptime/upptime) — мониторинг и статус-страница моих проектов целиком на GitHub Actions + Pages. Своего кода, сборки, линта и тестов нет. Репо `zagoruykov1/upptime`, публичный, статус-страница: https://status.zagoruykov.site. Это не пет-проект с VPS: Traefik/Ansible/compose из глобальных правил деплоя сюда не относятся.

## Что редактируется руками

Только `.upptimerc.yml` (список `sites`, блок `status-website`, уведомления). Всё остальное генерируют workflows и бот:

- `history/`, `api/`, `graphs/`, `summary.json`, таблица статусов в `README.md` между `<!--start: status pages-->` и `<!--end: status pages-->` — пишет бот, правки перетираются;
- `.github/workflows/*` — поставляются шаблоном, обновляются workflow `update-template`; руками не править;
- ветка `gh-pages` — собранная статус-страница, не трогать.

Ветка по умолчанию — `master` (так в шаблоне, workflows завязаны на неё). Бот постоянно коммитит в `master`, поэтому перед правками `git pull`.

## Как устроено

- Workflow `uptime` по cron (раз в 5 мин) опрашивает `sites` и пишет результат в `history/<slug>.yml`; при падении открывает issue, при восстановлении закрывает его. Уведомления: issues и Telegram-бот (блок `notifications` в `.upptimerc.yml`, о падении и восстановлении; секреты репо `NOTIFICATIONS_TELEGRAM_BOT_TOKEN` и `NOTIFICATIONS_TELEGRAM_CHAT_ID`). Доставка в Telegram ещё не проверялась на реальном падении.
- Slug в `history/`, `api/`, `graphs/` — kebab-case от `name` сайта (`FotoFlash` → `foto-flash`). Переименование сайта в конфиге создаёт новую историю, старая остаётся мёртвым грузом.
- `setup` (Setup CI) срабатывает на push изменения `.upptimerc.yml`: подчищает и перегенерирует README/сайт под новый конфиг, поэтому после правки конфига ждите его коммит.
- Статус-страница: `cname: status.zagoruykov.site` в конфиге + CNAME `status` → `zagoruykov1.github.io` в DNS + Pages из `gh-pages`.
- Токен `GH_PAT` (classic PAT, scopes `repo` и `workflow`) лежит в секретах репо; без него workflows не могут пушить. Срок действия токена ограничен — при истечении мониторинг молча остановится.

## Мониторимые сайты

Gurman (`ресторан-гурман.рф`, в конфиге punycode), FotoFlash (`fotoflash.zagoruykov.site`), FotoFlash API (`/healthz` того же хоста). Источник адресов — compose-файлы в `~/Projects/Personal/<проект>`. При добавлении нового проекта добавляйте его сюда же в `sites`.

## Команды

Локально запускать нечего. Полезное через `gh`:

```bash
gh run list -R zagoruykov1/upptime -L 10        # состояние workflows
gh workflow run uptime.yml -R zagoruykov1/upptime   # внеочередная проверка
gh issue list -R zagoruykov1/upptime            # активные инциденты
```

## Задачи
- Локальные: `TASKS.md` (формат: skill `tasks`).
- Внешнего трекера нет: GitHub Issues заняты инцидентами Upptime.
