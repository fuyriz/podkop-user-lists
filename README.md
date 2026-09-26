# podkop-user-lists

Личные списки доменов/подсетей для роутинга через Podkop (sing-box).

Репозиторий собирает **два независимых направления**:

- **Proxy** (`domains.srs` + `subnets.srs`) — всё, что должно идти через VPN
- **Russia-Direct** (`russia-domains.srs` + `russia-subnets.srs`) — российские ресурсы, которые должны идти напрямую, минуя VPN

## Свои списки (`data/`)

| Файл | Направление | Что туда писать |
|---|---|---|
| `data/own_domains.lst` | Proxy | свои домены, по одному на строку |
| `data/own_subnets.lst` | Proxy | свои подсети (CIDR), по одной на строку |
| `data/own_ru_domains.lst` | Russia-Direct | российские домены, которых не хватает в базовых списках |
| `data/own_ru_subnets.lst` | Russia-Direct | российские подсети, которых не хватает в базовых списках |

Строки, начинающиеся с `#`, и всё после `#` в строке — комментарии, игнорируются. Некорректные домены/CIDR не роняют сборку — пропускаются с предупреждением в логе Action.

## Источники

**Proxy** — [itdoginfo/allow-domains](https://github.com/itdoginfo/allow-domains) (категории `cloudflare`, `cloudfront`, `digitalocean`, `meta`, `discord`, `google_ai`, `hetzner`, `hodca`, `ovh`, `roblox`, `russia_inside`, `telegram`, `google_play`), `spotify.srs` от [MetaCubeX/meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat), `geosite-ru-blocked.srs` и `all-in-one.srs` (подсети) от [runetfreedom](https://github.com/runetfreedom) / [mudachyo/IP-Ranger](https://github.com/mudachyo/IP-Ranger).

**Russia-Direct** — `category-ru.srs` от [hydraponique/roscomvpn-geosite](https://github.com/hydraponique/roscomvpn-geosite), `ru.srs` от [Loyalsoldier/geoip](https://github.com/Loyalsoldier/geoip), текстовый список `ru.txt` от [runetfreedom/russia-blocked-geoip](https://github.com/runetfreedom/russia-blocked-geoip) — с вычитанием `ru-whitelist.txt` оттуда же (исключения, которые не должны блокироваться/уходить в отдельный маршрут).

## Сборка

`.github/workflows/build.yml` запускается ежедневно по расписанию, а также при пуше в `data/`, `scripts/` или сам workflow. Скачивает все источники выше, декомпилирует через `sing-box rule-set decompile`, сливает со своими списками (`scripts/merge.py`), чистит дубли и покрытые родительским доменом поддомены, схлопывает смежные подсети, компилирует в `.srs` и коммитит результат в `dist/`.

Дополнительно публикуется [Release `latest`](../../releases/latest) с теми же файлами — это резервная копия с историей версий, роутеры используют **не её** (см. ниже почему).

## Использование в Podkop

Роутеры на OpenWrt должны брать файлы напрямую из `master`, а не из Release — `uclient-fetch` не проходит цепочку редиректов, через которую GitHub отдаёт файлы релизов:
