# UNKNOWN: игрова основа и първа глава — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Работеща първа глава с профил за България, четири възрастови режима, интерактивни сцени, доказателства, избори и надежден локален прогрес.

**Architecture:** Самостоятелни JavaScript ES modules без framework и без runtime build. Чист reducer управлява напредъка; съдържанието е отделено по възраст и артефакт. Backend функциите са следващ самостоятелен план, а локалният state никога не е доказателство за покупка, родителство или турнирен резултат.

**Tech Stack:** HTML, CSS, native JavaScript modules; Node.js built-in test runner за логиката; GitHub Pages за тестовия клиент.

**Spec:** ../specs/2026-10-22-unknown-chapter-one-design.md и ../specs/2026-10-23-unknown-social-design.md, включително добавката за 7–9 години.

## Обхват на този план
Това е първият изпълним подпроект от одобрения продукт. Той доставя реална локална история, а не симулиран multiplayer.
Следващ отделен план: Supabase, реални акаунти/родителски връзки, приятели, частен чат и „Двойният сигнал“ между два профила.
Следващи отделни планове: покупки и активации на артефакти; реклами и подсказки; по-късните наградни турнири. Те не се включват като работещи бутони в тази доставка.
Преди публична продажба трябва да е готово обещаното съдържание на целия артефакт; първата глава не се продава автоматично като пълния продукт.

## Global Constraints
- Играта е предназначена само за България и интерфейсът е на български.
- Режими: 7–9, 10–12, 13–17 и 18+.
- Липсващ детски текст не може да се замества с 18+ текст.
- Под 7 години не е целева група за първата детска версия.
- Няма задължително чакане или ограничени опити.
- Текущият прототип се запазва на /legacy/; новата глава го заменя в корена след проверки.
- Импортиран или локално редактиран прогрес не носи класационни точки и право на награда.
- Не се използват симулирани играчи за попълване на екраните.
- Източници от проучванията не се качват в repository.
- Няма secrets, реални плащания, реклами или активни наградни обещания в тази доставка.

## Review Focus
1. Прекъснат запис или повреден import не унищожава последния валиден прогрес — задача 2.
2. Еднакви имена на населени места и невалидна възраст не водят до грешен профил — задача 1.
3. Повторно действие, refresh и бързо двукратно натискане не дублират предмет/избор — задача 3.
4. Детски профил или липсващ ресурс не показва възрастна история — задачи 1 и 4.
5. Тесен екран, клавиатура, липса на аудио и хостинг под /unknown-game/ запазват достъпа до задачите — задачи 5 и 6.

## File structure
- index.html — семантичен shell и entry module.
- legacy/index.html — непроменено текущо демо.
- package.json — type=module, scripts: test=node --test.
- src/domain/profile.js — входни данни и възраст.
- src/domain/state.js — състояние и валидни преходи.
- src/domain/storage.js — запис, възстановяване и import/export.
- src/content/catalog.js — статус и описание на артефактите.
- src/content/adult.js, teen.js, junior.js, kids.js — независими графи на историите.
- src/content/validate.js — валидиране на съдържание.
- src/data/settlements.json — проверени български населени места.
- src/ui/onboarding.js, scene.js, chat.js, evidence.js, settings.js — интерфейсни отговорности.
- src/app.js — orchestration, navigation и event handlers.
- styles/base.css, layout.css, themes.css — основа, отзивчивост и възрастови теми.
- assets/scenes/{mode}/ — оригинални фонове; уликите са HTML слоеве.
- assets/audio/kids/ — проверени български инструкции за детския режим.
- tests/profile.test.js, storage.test.js, state.test.js, content.test.js — тестове на реалните рискове.
- docs/qa/chapter-one.md — резултати и непроверени ограничения.

## Общи типове
AgeMode = 'kids' | 'junior' | 'teen' | 'adult'.
Settlement = {id:string, name:string, kind:'град'|'село', municipality:string, region:string}.
Profile = {nickname:string, age:number, mode:AgeMode, settlementId:string}.
ArtifactProgress = {nodeId:string, inventory:string[], evidence:string[], choices:Record<string,string>, flags:string[], ending:string|null}.
State = {schemaVersion:1, profile:Profile, selectedArtifactId:string, progress:Record<string,ArtifactProgress>, settings:{sound:boolean,reducedMotion:boolean}}.
Story = {id:string, artifactId:string, mode:AgeMode, startNodeId:string, nodes:Record<string,StoryNode>}.
StoryNode = {id:string, kind:'chat'|'scene'|'ending', title:string, text:string[], requiredFlags:string[], actions:ActionDefinition[]}.
ActionDefinition = {id:string, label:string, requires:{flags:string[],items:string[]}, answer:string|null, grants:{flags:string[],items:string[],evidence:string[]}, choice:{key:string,value:string}|null, nextNodeId:string|null, ending:string|null}.
Result<T> = {ok:true,value:T} | {ok:false,error:string}.
ActionInput = {actionId:string, answer?:string}.
UI не променя State директно. Ново действие минава през applyAction; ако отговорът е грешен, получава грешка и state остава непроменен.

### Task 1: Профил, населено място и възрастова изолация
**Files:** package.json, src/domain/profile.js, src/data/settlements.json, tests/profile.test.js.
**Interfaces:** ageMode(age:number):Result<AgeMode>; validateProfile(input:unknown, settlements:Settlement[]):Result<Profile>; searchSettlements(query:string, data:Settlement[]):Settlement[].

- [ ] Добави тестове: ageMode(7/9) → kids; 10/12 → junior; 13/17 → teen; 18 → adult; 6, NaN, 8.5 → error. Непознат settlementId и празен nickname се отхвърлят.
- [ ] Добави тест за две села с еднакво име и различни id: търсенето връща и двете, изборът запазва точния id.
- [ ] Изпълни node --test tests/profile.test.js и потвърди failure преди implementation.
- [ ] Подготви пълните записи за градове и села от проверим български регистър; нормализирай до Settlement, провери уникални id и непразни област/община. Не заменяй с измислен кратък списък; при недостъпни данни отбележи задачата блокирана.
- [ ] Реализирай функциите: възраст е цяло число >=7; nickname е trim-нат текст от 2 до 24 знака; търсенето е case-insensitive на български. Списъкът показва вид, населено място, община и област.
- [ ] Изпълни тестовете; commit само task файловете с message feat: add Bulgaria profile and age routing.

### Task 2: Надежден прогрес по артефакт
**Files:** src/domain/storage.js, tests/storage.test.js.
**Interfaces:** validateState(value:unknown, stories:Story[], settlements:Settlement[]):Result<State>; loadState(storage:StorageLike, stories, settlements):Result<State|null>; saveState(storage,state):Result<null>; exportState(state):string; importState(text,stories,settlements):Result<State>.
StorageLike има getItem/setItem/removeItem. Ключ unknown:story:v1 и резервен unknown:story:v1:backup.

- [ ] Тествай валиден roundtrip, невалиден JSON, неизвестен schemaVersion, неизвестни node/item ids и mode, несъвместим с възрастта.
- [ ] Тествай storage, който хвърля при setItem: saveState връща error, текущият memory state остава наличен. Повреден главен запис възстановява валидния backup.
- [ ] Изпълни node --test tests/storage.test.js; потвърди failure.
- [ ] Реализирай strict validation по структурата и текущия Story. Преди замяна на валиден главен запис го пази като backup. При import ограничи текста до 256 KB, отхвърли невалидното преди запис и не приемай ключове за точки/покупки/родителски права.
- [ ] Изпълни тестовете; commit feat: persist validated artifact progress.

### Task 3: Игрови преходи и предмети
**Files:** src/domain/state.js, tests/state.test.js.
**Interfaces:** createState(profile:Profile,story:Story):State; applyAction(state:State,input:ActionInput,story:Story):Result<State>; switchArtifact(state:State,story:Story):Result<State>.
Конкретният Story се избира по профила и artifact id, не по произволен URL параметър.

- [ ] С тестов story graph докажи, че заключен action не минава без флаг/предмет, грешен answer не променя state, събиране на улика е еднократно и трите ending стойности са достижими.
- [ ] Тествай повторно подаване на приключен action: не дублира item/evidence и не пренаписва choice. Несъвместим story.mode връща error.
- [ ] Изпълни node --test tests/state.test.js; потвърди failure.
- [ ] Реализирай immutable reducer с проверки преди промени. Списъците се третират като множества; story/action от друг артефакт се отхвърля. Завършен финал е устойчив при reload.
- [ ] Изпълни тестовете; commit feat: add story progression engine.

### Task 4: Четири реални версии на съдържанието
**Files:** src/content/{adult,teen,junior,kids,catalog,validate}.js, tests/content.test.js, assets/scenes/.
**Interfaces:** getStory(mode:AgeMode,artifactId:string):Result<Story>; validateStory(story:Story):string[].
Каталогът връща само готовия playable artifact за избраната възраст; 002 и 003 са обозначени „В разработка“, без fake purchase.

- [ ] Напиши тестове за уникални ids, съществуващи nextNodeId, достижими финали и липса на пътища между различни възрастови графи. getStory('kids','adult-only') връща error.
- [ ] Изпълни node --test tests/content.test.js; потвърди failure.
- [ ] Adult/teen: реализирай трите сцени от spec и избора transmit/hide/destroy; споделяне на улика и подкрепено обвинение променят конкретни последващи реплики.
- [ ] Junior: самостоятелно умерено разследване с няколко стъпки; никакъв fallback към adult.
- [ ] Kids: робот и обсерватория, три сцени — събиране на форми, подреждане на звездна карта и възстановяване на сигнала; доброжелателен завършек, кратки изречения и безплатни подсказки. Всички варианти решават локалната история.
- [ ] Създай оригинални изображения чрез наличния image-generation workflow. Детските сцени са самостоятелни топли изображения. Четимите улики не се запичат във фон.
- [ ] Изпълни content tests и преминаване на всеки граф с reducer. Ръчно прегледай текста за логика, тон и последователност. Commit feat: add age-specific chapter content and scenes.

### Task 5: Интерфейс и достъпност
**Files:** index.html, legacy/index.html, src/app.js, src/ui/*.js, styles/*.css, assets/audio/kids/.
**Interfaces:** mountOnboarding(root:HTMLElement,{settlements,onSubmit}); renderScene(root,{state,story,onAction}); renderChat(root,{state,story,onAction}); renderEvidence(root,{state}); renderSettings(root,{state,onChange,onExport,onImport,onRestart}).
onAction приема ActionInput; onSubmit приема валиден Profile; всички текстове се вмъкват с textContent или безопасно създадени елементи.

- [ ] Запази текущото index.html без изменения в legacy/index.html. Изгради responsive shell с „Съобщения“, „Разследване“, „Доказателства“, „Артефакти“ и настройки.
- [ ] Свържи профила, reducer и storage. Показвай истинска грешка при неуспешен запис; local progress се обозначава като пазен на това устройство.
- [ ] Добави сцени с достъпни button hotspots и алтернативен списък на предметите; панели с видим фокус и връщане на фокуса след затваряне.
- [ ] Добави три степени на подсказка без таймер; няма монетизация в тази версия. При import/restart поискайте потвърждение вътре в продукта преди загуба на текущ прогрес.
- [ ] Реализирай детски audio control с български записи за основните инструкции, текстов еквивалент и без autoplay. Ако няма подходящо средство за записи, остави текста работещ, отбележи audio като незавършен и не обявявай детския режим за готов за пускане.
- [ ] Проверка в браузър: 390px и 1280px, клавиатура, намалено движение, изключен звук, грешен отговор, popup focus, презареждане в сцена. Ако browser QA не е достъпен, документирай ограничението вместо да твърдиш, че е минал.
- [ ] Commit feat: connect playable responsive chapter interface.

### Task 6: Интеграция, преглед и тестово публикуване
**Files:** docs/qa/chapter-one.md; корекции само по намерени дефекти.
**Interfaces:** крайният сайт използва относителни imports/assets; няма runtime server dependency за самостоятелния сюжет.

- [ ] Изпълни npm test; очакван резултат: всички profile/storage/state/content тестове минават.
- [ ] Провери под /unknown-game/, не само на /: module imports, images и data fetch нямат 404.
- [ ] Изиграй четирите режима; adult през трите финални избора; kids без възрастни текстове. Провери import/export и прекъсване на storage.
- [ ] Направи независим code review според избрания метод на изпълнение; коригирай установените дефекти преди публикация.
- [ ] Запиши точните проверки и незавършените части в QA файла. Ако липсват визуални/аудио ресурси, не наричай главата завършена.
- [ ] Качи на работен branch, интегрирай само проверената версия по разрешения GitHub workflow. Запази legacy достъпен.
- [ ] Провери GitHub Pages deployment result; предостави действителния URL и уточни, че реалните чат/покупки/турнири са следващи подпроекти.

## Проверка на покритието
Профил/България → 1; state/save → 2–3; възрастови истории/предмети/финали → 4; сцени/детско аудио/интерфейс → 5; проверка и хостинг → 6.
Социални функции, родителска идентичност, покупки, реклами, каталожни продажби и парични състезания са изрично отделени от този план и не се представят като изпълнени.
Реалните тестове със семейства и играчи се провеждат след работеща версия и набрани от собственика участници; тук не се изпращат покани от негово име.
