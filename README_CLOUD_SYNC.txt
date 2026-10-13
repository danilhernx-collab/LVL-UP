DC: LEVEL UP — НАСТРОЙКА ОБЛАЧНОЙ СИНХРОНИЗАЦИИ ДЛЯ IPHONE + ПК

ЧТО ПОЛУЧИТСЯ
Один и тот же прогресс на компьютере и iPhone. Приложение устанавливается на главный экран iPhone через Safari. Для синхронизации нужен интернет; после изменений локальная копия сохраняется сразу, а облако обновляется при подключении.

ШАГ 1. Создай облачную базу
1. Открой https://supabase.com/ и создай аккаунт/проект (Free достаточно для личного трекера).
2. В панели проекта найди Project URL и публичный ключ publishable/anon (обычно Settings → API Keys / Connect). Нужны только Project URL и публичный ключ. НЕ используй service_role или secret key.
3. Открой SQL Editor → New query, вставь весь SQL ниже и нажми Run:

create table if not exists public.dc_level_up (
  user_id uuid primary key references auth.users(id) on delete cascade,
  data jsonb not null,
  updated_at timestamptz not null default now()
);
alter table public.dc_level_up enable row level security;
drop policy if exists "Users can read own DC data" on public.dc_level_up;
drop policy if exists "Users can insert own DC data" on public.dc_level_up;
drop policy if exists "Users can update own DC data" on public.dc_level_up;
create policy "Users can read own DC data" on public.dc_level_up for select to authenticated using ((select auth.uid()) = user_id);
create policy "Users can insert own DC data" on public.dc_level_up for insert to authenticated with check ((select auth.uid()) = user_id);
create policy "Users can update own DC data" on public.dc_level_up for update to authenticated using ((select auth.uid()) = user_id) with check ((select auth.uid()) = user_id);
grant select, insert, update on public.dc_level_up to authenticated;

4. В Authentication → Providers → Email проверь, что Email provider включён. Для самого простого старта можно оставить подтверждение email включённым: после регистрации подтверди письмо и затем войди. Если отключишь подтверждение, защити аккаунт надёжным паролем.

ШАГ 2. Размести приложение в интернете (GitHub Pages)
1. Создай аккаунт на https://github.com/ если его нет.
2. Создай новый репозиторий, например dc-level-up, и выбери Public.
3. Распакуй ZIP. Загрузи в корень репозитория файлы index.html, manifest.webmanifest и sw.js. Можно также загрузить README.txt и README_CLOUD_SYNC.txt.
4. В репозитории открой Settings → Pages. В разделе Build and deployment выбери Deploy from a branch, Branch: main, folder: /(root), нажми Save.
5. Подожди завершения публикации. GitHub покажет адрес вида https://ТВОЙ-ЛОГИН.github.io/dc-level-up/ . Открой этот адрес на компьютере.

ШАГ 3. Подключи облако в трекере на компьютере
1. В трекере открой Настройки → Синхронизация между устройствами.
2. Вставь Supabase Project URL и публичный anon/publishable key.
3. Нажми «Сохранить настройки подключения».
4. Введи email и пароль, нажми «Войти / создать аккаунт». Если создан новый аккаунт и включено подтверждение email, подтверди письмо и войди снова.
5. При первом входе, если облако пустое, текущие данные отправятся в облако. Если в облаке уже есть данные, приложение спросит, загрузить их на устройство или сохранить локальные данные в облако. Внимательно выбери правильный вариант.
6. Нажми «Синхронизировать сейчас» и убедись, что статус показывает «Синхронизировано».

ШАГ 4. Установи на iPhone
1. Открой опубликованную GitHub Pages ссылку именно в Safari на iPhone.
2. Нажми кнопку «Поделиться» (квадрат со стрелкой вверх).
3. Прокрути вниз и выбери «На экран Домой» / Add to Home Screen. Если есть опция открытия как веб-приложение, включи её.
4. Нажми «Добавить». Запускай DC: LEVEL UP с новой иконки на экране «Домой».
5. Открой Настройки трекера, введи тот же Project URL, тот же публичный ключ, тот же email и пароль, сохрани настройки и войди в тот же аккаунт. Данные из облака загрузятся на iPhone.

КАК ЭТО РАБОТАЕТ
- Любое изменение сразу сохраняется локально и через короткую паузу отправляется в облако, если ты вошёл в аккаунт и есть интернет.
- Для первого подключения второго устройства приложение спрашивает, загрузить данные из облака или отправить локальные данные. Не отправляй пустую копию поверх ценных данных.
- При работе без интернета изменения остаются локально, но автоматическое объединение конфликтов между двумя устройствами не реализовано. После офлайн-периода подключись к интернету и нажми «Синхронизировать сейчас»; избегай редактирования на двух устройствах одновременно без интернета.
- Экспортируй JSON-резервную копию периодически. Не очищай данные Safari/браузера до проверки, что облачная копия актуальна.
- Если Safari на iPhone блокирует вход, открой ссылку в обычной вкладке Safari, а не внутри Telegram/Instagram.

ЕСЛИ ЧТО-ТО НЕ РАБОТАЕТ
- «Failed to fetch»: проверь Project URL, интернет и что сайт опубликован по HTTPS.
- 401 / Invalid API key: используй именно публичный anon/publishable key из своего проекта.
- 403 / permission denied: проверь, что SQL из инструкции выполнен целиком и RLS включён.
- Email not confirmed: подтверди адрес из письма, затем войди снова.
- Не видишь свежие изменения: проверь статус синхронизации и нажми «Синхронизировать сейчас».

БЕЗОПАСНОСТЬ
Публичный anon/publishable key предназначен для клиентского приложения, но безопасность обеспечивают политики RLS. Не вставляй в приложение service_role/secret key. Не делись паролем и сохраняй резервные копии.
