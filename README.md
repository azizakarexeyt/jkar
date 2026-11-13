# Nexor Client

Minecraft **1.21.11** icin Fabric istemci modu (Mojang mappings, sadece istemci).
Bu mod yalnizca Nexor'a ozel ozellikleri icerir. FPS modlari, Voice Chat vb. ayri modlardir ve
**NexorMC launcher'i** bunlari otomatik indirip birlikte calistirir (asagida "Paketleme").

## Mimari

```
NexorClient (ClientModInitializer)
 ├─ ModuleManager ── modul listesi, tick, HUD cizimi, nexormc.json kaydet/yukle
 │    └─ Module ── TextModule ── Fps / Coords / Ping / ServerIp / PackDisplay
 │              ├─ Keystrokes, Armor            (ozel cizimli HUD modulleri)
 │              └─ Crosshair, Fullbright, Nametags  (ozellik modulleri)
 ├─ AutoConnect ──── ana menuye ilk giriste nexormc.marthost.net'e baglanir, /nexor connect
 ├─ CustomGuiPause ─ ESC menusu: logo + "Nexor Settings" (Mods'un ustune)
 │    └─ mixin/ScreenBackgroundMixin ── ESC arka plan blur'unu kaldirir
 ├─ gui/HudEditorScreen ── tasi (surukle), boyutlandir (tekerlek), ac/kapa, ayarlar
 └─ mixin/NametagMixin ── kendi isim etiketini 3. sahista gosterir
```

| Istek | Nerede |
|---|---|
| Otomatik baglanti | `AutoConnect.java` (+ `/nexor connect`) |
| ESC seffaf arka plan | `ScreenBackgroundMixin.java` |
| ESC ustte logo | `CustomGuiPause.drawLogo` + `assets/nexormc/textures/gui/logo.png` |
| "Nexor Settings" Mods'un ustunde | `CustomGuiPause.addNexorButton` |
| HUD surukle / boyutlandir | `HudEditorScreen.java` |
| Modul ac/kapa + ayar | `HudEditorScreen.init`, `ModuleManager` |
| Fullbright / Crosshair / Nametags | `Modules.java`, `NexorClient` (crosshair), `NametagMixin` |
| Keystrokes, FPS, XYZ, Ping, Armor, Server IP, Resource Pack | `Modules.java` |

Tus: **Sag Shift** (HUD editoru). Sol Shift Minecraft'ta "egilme" oldugu icin varsayilan yapmadim;
istersen `NexorClient.DEFAULT_KEY`'i `InputConstants.KEY_LSHIFT` yap.

## Derleme (bilgisayarinda Java/Gradle kurmadan)

1. Bu klasoru bir GitHub reposuna yukle.
2. **Actions** sekmesinde `build` calisir (ya da "Run workflow").
3. Bitince **Artifacts > nexor-client** icindeki `nexor-client-1.0.0.jar` dosyasini indir
   (`-sources` olani degil).
4. Jar'i `%APPDATA%\NexorMC\extra-mods\` klasorune koy. NexorMC launcher'i sonraki
   acilista (veya "Modlari yeniden indir" ile) onu oyuna ekler.

Kendi bilgisayarinda derlemek istersen: Java 21 + Gradle 9, `gradle build`.

## BU KOD DERLENIP OYUNDA DENENMEDI

Yazildigi ortamda Minecraft/Gradle yok. Asagidaki API'ler 1.21.11 belgelerine ve
1.21.6-1.21.9 degisikliklerine gore yazildi, ama dogrulanmadi. Derleme hatasi cikarsa buralara bak:

1. `GuiGraphics` (cizim): `drawString(Font, String, x, y, color, shadow)`, `fill`, `renderItem`,
   `renderItemDecorations`, `blit(RenderPipelines.GUI_TEXTURED, id, x, y, u, v, w, h, texW, texH)`.
   Guncel Fabric belgeleri (26.x) bunun yerine `GuiGraphicsExtractor` gosteriyor; 1.21.11'de
   `GuiGraphics` olmasi gerekiyor.
2. `g.pose()` = `Matrix3x2fStack` (`pushMatrix/translate/scale/popMatrix`).
3. `HudElement.render(GuiGraphics, DeltaTracker)` ve `HudElementRegistry.replaceElement`.
4. `Screen` fare metotlari `MouseButtonEvent` aliyor (`mouseClicked/mouseDragged/mouseReleased`).
5. `Screen.renderBackground(GuiGraphics, int, int, float)` ve `LivingEntityRenderer.shouldShowName(T, double)`
   (mixin hedefleri; bulunamazsa mixin sessizce devre disi kalir, oyun cokmez).
6. `Screens.getButtons(screen)` ile ESC menusune buton ekleme ve Mod Menu'nun "Mods" butonunu bulma.
7. `ConnectScreen.startConnecting(Screen, Minecraft, ServerAddress, ServerData, boolean, TransferState)`.

Hata mesajini bana yapistirirsan duzeltirim.

## Logo

`logo.png` simdilik yer tutucu (sari tac + "Nexormc"). Konusmaya gelen gorsel bana ulasmadi.
Kendi logonu **128x64 piksel, seffaf arka planli PNG** olarak ayni yola koy. Farkli boyut
kullanacaksan `CustomGuiPause.LOGO_W/LOGO_H` sabitlerini degistir. Siyah/koyu arka plani
seffaflastirmak icin gorseli bir editorde (Photoshop, Photopea) "arka plani sil" ile PNG kaydet.

## Paketleme / performans (launcher tarafi)

NexorMC.exe su modlari otomatik indirir: Sodium, Lithium, FerriteCore, ModernFix, Entity Culling,
ImmediatelyFast, Sodium Extra, Dynamic FPS, Krypton, Fabric API, Mod Menu, **Simple Voice Chat,
Resourcify, AppleSkin, JEI, ViaFabricPlus** (eski surum sunuculara baglanmak icin).
JVM bayraklari GC iyilestirmesi icin ayarlandi (G1, kisa duraklama).

Notlar:
- "3000+ FPS" garanti edilemez: bu sayi bilgisayara, sahneye ve render mesafesine bagli.
  Bahsedilen modlar vanilla'ya gore FPS'i genelde katlar, ama sabit bir sayi sozu veremem.
- Simple Voice Chat'in calismasi icin **sunucuya da** "Simple Voice Chat" plugin'i/modu kurulmali
  (varsayilan UDP 24454 portu acik olmali).
- ViaVersion **sunucu** tarafidir; istemcide karsiligi ViaFabricPlus'tir (eklendi).
