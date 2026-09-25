# Tito Dimension & Pet Empire — Fabric-мод для Minecraft 1.20.6

Повний вихідний код Fabric-мода на тему «Tito Dimension & Pet Empire»:
чорне цуценя Тіто, понад 200 генерованих варіантів/мутацій, бос Тіто
Титан, портальний блок і кастомний вимір.

## Структура проєкту

```
build.gradle, settings.gradle, gradle.properties   — Gradle/Fabric Loom конфігурація
src/main/java/com/tito/dimension/
  TitoMod.java                     — головний ModInitializer
  entity/
    TitoEntity.java                — базова сутність-супутник (приручення, режими, інвентар, пошук руд)
    ModEntities.java               — реєстрація EntityType + атрибути + природний спавн
    variant/
      TitoVariants.java            — data-driven реєстр 210+ варіантів (категорія × елемент × розмір)
      ZombieTitoEntity.java
      SkeletonTitoEntity.java
      CreeperTitoEntity.java
      EnderTitoEntity.java
    boss/
      TitoTitanEntity.java         — бос: 1000 HP, ServerBossBar, 3 фази атак, нагорода
  item/ModItems.java                — "Кісточка Тіто" (tito_bone)
  block/
    ModBlocks.java
    TitoPortalBlock.java           — телепортація у кастомний вимір
  screen/
    TitoScreenHandler.java         — інвентар супутника (9-18 слотів)
    ModScreenHandlers.java
  network/TitoNetworking.java      — ServerPlayNetworking канал зміни режиму
  event/
    TitoPlayerEvents.java          — видача стартового блока при першому вході
    TitoStarterState.java          — надійне збереження прапорця (PersistentState)
  world/ModDimensions.java         — RegistryKey кастомного виміру
  client/
    TitoModClient.java             — реєстрація рендерів
    TitoEntityRenderer.java
    TitoTitanEntityRenderer.java

src/main/resources/
  fabric.mod.json
  assets/tito_dimension/           — lang, textures, models, blockstates
  data/tito_dimension/             — dimension, dimension_type, biome, loot_table, recipe, tito_variants
```

## Що реалізовано повністю

- Приручення кісткою (`minecraft:bone`) або кісточкою Тіто (`tito_bone`, крафт з 4 кісток + відро молока).
- Режими супутника: Слідувати / Сидіти / Охороняти (перемикання: Sneak + ПКМ; або мережевий пакет для кастомного HUD).
- ПКМ (без Sneak, пуста рука) відкриває інвентар супутника на 18 слотів (`TitoScreenHandler`).
- Пошук руд у радіусі 10 блоків раз на 2 секунди зі звуком + частинками.
- Data-driven система з **210 варіантами** (5 категорій × 14 елементів × 3 розміри), що вирішує вимогу "понад 100 видів" без ручного написання кожного класу.
- Zombie-Tito (агресія, ефекти Голод/Отрута, нічний спавн), Skeleton-Tito (RangedAttackMob, стрільба з лука), Creeper-Tito (набухання + вибух салютом/лутом замість руйнування блоків), Ender-Tito (телепорт при отриманні шкоди, фіолетові частинки).
- Tito Titan: 1000 HP, `ServerBossBar`, масштаб ×9, 3 фази (ударна хвиля з відкиданням → призов зомбі-Тіто → лазерний залп), при смерті — `sendTitle`/`sendSubtitle` усім гравцям поруч, салют частинками, дроп мегалуту + окрема loot_table.
- Портальний блок, що видається автоматично при першому вході (надійно, через `PersistentState`, переживає рестарт сервера).
- Кастомний вимір `tito_dimension:tito_dimension` (data-driven `dimension_type` + `dimension` + `biome`), де природно спавняться лише варіації Тіто.

## Обмеження / що потрібно доробити самостійно

1. **3D-модель.** Через відсутність художніх ассетів рендерер тимчасово
   використовує стандартну `WolfEntityModel` як риг (з анімаціями
   сидіння/бігу вже "з коробки"). Для кастомної моделі підключіть
   GeckoLib (додайте залежність у `build.gradle`) і замініть
   `TitoEntityRenderer`/`TitoTitanEntityRenderer` на `GeoEntityRenderer`
   зі своєю `.geo.json` моделлю та анімаціями.
2. **Текстури 210 варіантів.** Код підтримує довільну кількість
   варіантів, але реальні `.png`-файли для кожного (`textures/entity/tito/<id>.png`)
   потрібно домалювати. Наразі є: `tito_base.png` (з вашого завантаженого
   фото Tito.png) та 5 заглушок для основних мутацій
   (zombie/skeleton/creeper/ender/titan) — усі тимчасово копіюють базове
   зображення, доки не буде готового текстур-атласу під UV-розгортку
   вовчої моделі (64×32).
3. **Портальний блок і предмет "кісточка"** мають лише прості
   програмно згенеровані заглушки текстур (16×16), а не фінальний арт.
4. **Ground-slam/laser** — спрощені (частинки + AoE-урон) заміни повноцінних
   кастомних анімацій/хітбоксів; розширюйте за потреби.
5. Для покриття звичайного логіну (не лише респавну) переконайтесь, що
   `ServerPlayConnectionEvents.JOIN` доступний у версії Fabric API, яку
   ви підключаєте (у build.gradle вказано `0.100.4+1.20.6` — сумісно з
   1.20.6/1.21 API, за потреби відкоригуйте номер версії під ваш
   `minecraft_version`).

## Збірка

1. Встановіть JDK 21 та [Gradle 8.8](https://gradle.org) (або згенеруйте
   wrapper локально командою `gradle wrapper` — бінарник
   `gradle-wrapper.jar` не включений у цю доставку, лише
   `gradle-wrapper.properties`).
2. Відкрийте проєкт в IntelliJ IDEA з плагіном Minecraft Development
   (Fabric), або виконайте:
   ```
   gradle build
   ```
3. Готовий `.jar` з'явиться у `build/libs/tito-dimension-1.0.0.jar` —
   покладіть його у папку `mods` клієнта/сервера з встановленим Fabric
   Loader ≥ 0.15.11 та Fabric API.

## Ідеї для розширення

- Розмноження Тіто (`createChild` наразі повертає `null`).
- Приручення мутованих варіантів (зомбі/скелет/кріпер/ендер) через
  спеціальні "зцілюючі" предмети, аналогічно `WEAKNESS + Golden Apple`
  для зомбі-жителів.
- Структура "Арена Тіто Титана" через `structure_set`/`template_pool`
  замість ручного розміщення.
- GUI-кнопки режимів (замість Sneak+ПКМ) з використанням вже готового
  мережевого каналу `TitoNetworking.SetModePayload`.
