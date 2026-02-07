# Редактор конфигураций Remnawave
Проект ориентирован на простое и легкое редактирование конфигов, под mihomo, внутри remnawave sub-page 

Основные функции: 
- Создание Авто-серверов(балансировка)
- Скрытие серверов из подписки
- Отказоустойчивость подписки
- Замена select'ов в proxy-groups
- Удаление select'ов в конфигурации

Интеграция уровня Mihomo (подстраничный форк)

Резюме
- Добавлен конфигурационный слой Mihomo с возможностью быстрой перезагрузки в серверную часть, который переписывает
  subscription YAML на основе configMi.yaml.
- Работает во время отклика и не требует перезапуска службы при
изменении configMi.yaml.

Добавлены/изменены файлы
- backend/src/modules/root/mihomo-layer.service.ts (новый)
- backend/src/modules/root/root.service.ts
- backend/src/modules/root/root.module.ts
- backend/src/common/config/app-config/config.schema.ts

Что она делает
- Когда ответ на подписку выглядит как Mihomo YAML (содержит прокси-серверы
  и/или прокси-групп), он анализируется и переписывается заново:
  - применяются параметры баланс/переделка/скрытие/замена/удаление.
  - замена применяется только к группам с типом: select (включая ПРОКСИ).
  - удаление удаляет элементы прокси и ссылки на группы повсюду.
  - новые группы вставляются перед ПРОКСИ и добавляются в ПРОКСИ, если они
не являются целью замены, скрыты или удалены.

Куда поместить configMi.yaml
- Поместите configMi.yaml на хост и смонтируйте его в контейнер.
- Установите для env MIHOMO_LAYER_CONFIG_PATH значение пути внутри контейнера.
- Если MIHOMO_LAYER_CONFIG_PATH не задан, серверная часть выполнит поиск
  configMi.yaml в рабочем каталоге серверной части (process.cwd()).

Пример настройки (compose)
- Добавляем монтирование тома в контейнер серверной службы.
- Добавляем env MIHOMO_LAYER_CONFIG_PATH.

Пример фрагмента (адаптируйте к вашему файлу compose):
услуги:
  remnawave-страница подписки:
    тома:
      - /opt/remnawave/подписка/configMi.yaml:/app/configMi.yaml:ro
    окружающая среда:
      - MIHOMO_LAYER_CONFIG_PATH=/app/configMi.yaml

Локальный запуск (dev)
1) Установка deps:
   подстраница cd-fork/серверная часть
   установка npm
2) Установка env:
   MIHOMO_LAYER_CONFIG_PATH=C:\путь\к\configMi.yaml
3) Запустите:
   запуск npm:dev

Локальный запуск (prod)
1) Сборка:
   подстраница cd-форк/серверная часть
   запуск сборки npm
2) Установка env:
   MIHOMO_LAYER_CONFIG_PATH=/путь/к/configMi.yaml
3) Запустить:
   запуск npm:prod

Записи
- Слой горячей загружается с использованием файла время изменения; Изменения вступают в силу на следующий
  запрос на подписку.
- Если configMi.yaml отсутствует или недействителен, служба возвращается к
последней действительной кэшированной версии (или без изменений, если таковых нет).

# Configuration Editor Remnawave
The project is focused on simple and easy editing of configs, under mihomo, inside the remnawave sub-page 

Main features: 
- Creating Auto-servers (balancing)
- Hiding servers from the subscription
- Subscription failover
- Replacing select's in proxy-groups
- Removing select's in the configuration

Mihomo layer integration (sub-page-fork)

Summary
- Added a hot-reloadable Mihomo config layer to the backend that rewrites
  subscription YAML based on configMi.yaml.
- Works at response time and does not require restarting the service when
  configMi.yaml changes.

Files added/modified
- backend/src/modules/root/mihomo-layer.service.ts (new)
- backend/src/modules/root/root.service.ts
- backend/src/modules/root/root.module.ts
- backend/src/common/config/app-config/config.schema.ts

What it does
- When a subscription response looks like Mihomo YAML (contains proxies
  and/or proxy-groups), it is parsed and rewritten:
  - balance/remake/hidden/replace/delete are applied.
  - replace is applied only to groups with type: select (including PROXY).
  - delete removes proxy items and group references everywhere.
  - new groups are inserted before PROXY and added to PROXY unless they
    are a replace target, hidden, or deleted.

Where to put configMi.yaml
- Put configMi.yaml on the host and mount it into the container.
- Set env MIHOMO_LAYER_CONFIG_PATH to the in-container path.
- If MIHOMO_LAYER_CONFIG_PATH is not set, the backend looks for
  configMi.yaml in the backend working directory (process.cwd()).

Docker example (compose)
- Add a volume mount to the backend service container.
- Add env MIHOMO_LAYER_CONFIG_PATH.

Example snippet (adjust to your compose file):
services:
  remnawave-subscription-page:
    volumes:
      - /opt/remnawave/subscription/configMi.yaml:/app/configMi.yaml:ro
    environment:
      - MIHOMO_LAYER_CONFIG_PATH=/app/configMi.yaml

Local run (dev)
1) Install deps:
   cd sub-page-fork/backend
   npm install
2) Set env:
   MIHOMO_LAYER_CONFIG_PATH=C:\path\to\configMi.yaml
3) Run:
   npm run start:dev

Local run (prod)
1) Build:
   cd sub-page-fork/backend
   npm run build
2) Set env:
   MIHOMO_LAYER_CONFIG_PATH=/path/to/configMi.yaml
3) Run:
   npm run start:prod

Notes
- The layer is hot-loaded using file mtime; changes apply on the next
  subscription request.
- If configMi.yaml is missing or invalid, the service falls back to the
  last valid cached version (or no changes if none).
