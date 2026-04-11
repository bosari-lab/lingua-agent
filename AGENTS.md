# LinguaAgent — instrukcja dla agentów AI

## Czym jest ten projekt

LinguaAgent to aplikacja webowa do nauki języków obcych (angielski, francuski, włoski) zbudowana jako **jeden plik HTML** (`index.html`). Właścicielką projektu jest Aleksandra — osoba nietechniczna. Wszystkie wyjaśnienia po polsku, prostym językiem.

**URL produkcyjny:** https://bosari-lab.github.io/lingua-agent  
**Repo:** `bosari-lab/lingua-agent`, branch `main`, plik `index.html`  
**Plik roboczy na Macu:** `/Users/aleksandrabarwasna/Downloads/index.html`

---

## Stack techniczny

- **Jeden plik HTML** — HTML + CSS + JS w `index.html`. Zero frameworków, zero Node.js, zero bundlerów.
- **Vanilla JavaScript** — czysty JS, bez React/Vue
- **CSS Variables** — ciemny motyw
- **IndexedDB** — pliki wideo lokalnie (`openDB`, `saveVideo`, `loadVideo`, `deleteVideo`)
- **localStorage** — lekcje, ustawienia, klucze API, nagrania audio

---

## Zewnętrzne API i klucze

| API | Do czego | localStorage key | Status |
|-----|----------|-----------------|--------|
| **OpenAI TTS** `tts-1-hd` | Głosy lektorów (preferowany) | `lingua_oai_key` | ✅ aktywny |
| **ElevenLabs** `eleven_multilingual_v2` | Alternatywny silnik głosowy | `lingua_el_key` | ✅ aktywny |
| **Anthropic Claude** | Analiza PDF/zdjęć/wideo → lekcja | `lingua_anth_key` | ✅ aktywny |
| **RapidAPI YouTube Transcript** | Transkrypcja z YouTube | `lingua_rapid_key` | ⏳ w konfiguracji |

**Nigdy nie hardkoduj kluczy API w pliku.**

### Modele Anthropic używane w projekcie
- `claude-haiku-4-5-20251001` — szybkie zadania (analiza tekstu, tłumaczenia słów)
- `claude-opus-4-5-20251101` — długie/trudne analizy (wideo, długie PDF-y)

### ElevenLabs voice IDs
- `EXAVITQu4vr4xnSDxMaL` — Sarah (głos żeński, EN)
- `onwK4e9ZLuTAKqWW03F9` — Daniel (głos męski, EN/FR/IT)
- Model: `eleven_multilingual_v2`

### OpenAI TTS głosy
- Żeński (Aleksandra): `S.oaiVoiceFemale` — domyślnie `nova`
- Męski (rozmówca): `S.oaiVoiceMale` — domyślnie `echo`
- Dostępne: nova, shimmer, alloy (żeńskie) | echo, fable, onyx (męskie)

---

## Architektura — ekrany (`S.screen`)

| Ekran | Funkcja |
|-------|---------|
| `home` | Strona główna z wyborem języka |
| `lang` | Lista lekcji danego języka + Moje lekcje |
| `lesson` | Dialog, słowniczek, gramatyka + odtwarzacz wideo |
| `practice` | Ćwiczenia: fiszki (`flashcard`) lub uzupełnianie (`fill`) |
| `results` | Wyniki sesji |
| `settings` | Klucze API, wybór silnika TTS, głosy |
| `audiomgr` | Nagrania MP3/WAV per lekcja (3 sloty) |
| `filmoteka` | Biblioteka filmów + wgrywanie klipów MP4/MP3 |

---

## Stan globalny (obiekt `S`)

Zmiana przez `set({klucz:wartość})` → wywołuje `render()`.

```javascript
S = {
  // Nawigacja
  screen, lang,

  // Ćwiczenia
  lesson, mode, cards, idx, revealed, dir, score, input, submitted,

  // Audio odtwarzanie
  playing, playingLine, abort,
  audioPlaying, audioObj, audioMgrLesson,

  // Dane trwałe (z localStorage)
  lessonAudio,    // { lessonId: {full,female,male} } — base64 dataURL
  lessonVideos,   // { lessonId: videoKey } — klucze IndexedDB
  myLessons,      // { EN:[...], FR:[...], IT:[...] }

  // TTS
  ttsEngine,         // "oai" | "el" | "browser"
  oaiKey, oaiVoiceFemale, oaiVoiceMale,
  elKey, elKeyInput, elMsg,
  anthKey,
  voiceRate,         // 0.7–1.4

  // Upload podręcznika
  showUpload, uploadLang, uploadStep, uploadFile,
  uploadPreview, uploadProcessing, uploadError,

  // Filmoteka
  filmLang, filmUploadStep, filmFile,
  filmProcessing, filmError,
}
```

---

## Struktura danych lekcji

### Format dialogu — WAŻNE
Aktualny format używa `s/l/t` (NIE `speaker/line/tr`):
```javascript
dialog: [{ s: "Imię rozmówcy", l: "Tekst w języku obcym", t: "Tłumaczenie PL" }]
```

### Pełna struktura lekcji
```javascript
{
  id,           // unikalny string np. "film_1234567890"
  title, level, // "B1"
  topic,        // po polsku
  type: "dialog",
  hasVideo: false,   // true = plik wideo w IndexedDB
  fromFilm: false,   // true = z filmoteki/YouTube
  dialog:  [{ s, l, t }],
  vocab:   [{ w, tr, ipa, pl, ex, exT }],
  fill:    [{ s:"Zdanie z ___ luką", a:"odpowiedź", h:"wskazówka PL" }],
  grammar: [{ rule, ex, exT, note }]
}
```

### Minimalne wymagania jakości dla lekcji z PDF/wideo
- **dialog**: minimum 8 kwestii
- **vocab**: minimum 15 słów/zwrotów
- **fill**: minimum 5 ćwiczeń
- **grammar**: minimum 4 zasady

---

## System audio

### Silnik TTS (`S.ttsEngine`)
- `"oai"` → `oaiSpeak(text, voiceId)` — OpenAI API
- `"el"` → `elSpeak(text, lang, voiceId)` — ElevenLabs API
- `"browser"` → `browserSpeak(text, lang)` — Web Speech API

### Funkcje
- `speak(text, lang, speakerName)` — wybiera engine + głos automatycznie
- `playDialog(dialog, lang)` — sekwencyjne, pauza **180ms** między kwestiami
- `stopDialog()` / `stopAudio()` — zatrzymanie

**KRYTYCZNE:** `stopDialog()` i `stopAudio()` muszą być wywołane przy każdej nawigacji. Brak = dialogi z różnych lekcji grają jednocześnie.

### Nagrania audio per lekcja
`lingua_audio` w localStorage: `{ lessonId: { full, female, male } }` — base64 dataURL

---

## Wgrywanie materiałów

### Ścieżka A: Podręcznik/PDF/zdjęcie (`+ Wgraj tekst`)
Funkcja `processUpload()`:

```javascript
// Zdjęcie
{type:"image", source:{type:"base64", media_type:file.type, data:b64}}

// PDF — DOCELOWO: użyj pdf.js do lokalnej ekstrakcji tekstu
// TERAZ: {type:"document", source:{type:"base64", media_type:"application/pdf", data:b64}}
// TODO: dodać pdf.js (cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js)

// Tekst
{type:"text", text: PROMPT + "\n\nTEKST:\n" + uploadPreview}
```

**Model:** `claude-haiku-4-5-20251001`, `max_tokens: 2000`  
**Problem:** haiku tworzy za mało treści dla długich PDF-ów → zmienić na `claude-opus-4-5-20251101` + `max_tokens: 4000`

**Prompt dla PDF musi wymagać minimum:**
- 15-20 słów w vocab
- 8+ kwestii w dialogu
- 5+ ćwiczeń fill
- 4+ zasad gramatyki

### Ścieżka B: Klip wideo/audio (`🎬 Filmoteka`)
Funkcja `processFilm()`:
- Model: `claude-opus-4-5-20251101`, `max_tokens: 3000`
- Audio → `{type:"document", media_type: file.type, data:b64}`
- Wideo (MP4) → `{type:"document", media_type:"video/mp4", data:b64}`
- Plik wideo → IndexedDB przez `saveVideo(key, blob)`
- Limit: 25MB

### Ścieżka C: YouTube (w trakcie implementacji)
Dwa endpointy RapidAPI do wypróbowania:
1. `youtube-transcript3.p.rapidapi.com/api/transcript?videoId={id}&lang=en`
2. `youtube-transcriptor.p.rapidapi.com/transcript?video_id={id}&lang=en`

Klucz: `S.rapidKey` / localStorage `lingua_rapid_key`

---

## Filmoteka (`FILMS`)

```javascript
{ id, title, type, level, platform, desc, scene }
// youtubeId — do dodania
```

Filmy: EN×5, FR×5, IT×5. Bez konkretnych YouTube ID — do uzupełnienia.

---

## Design system

### Kolory
```css
--bg:#0f0f0f; --bg2:#161616; --bg3:#1e1e1e;
--surface:#1a1a1a; --surface2:#222222;
--border:#2a2a2a; --border2:#383838;
--text:#f0ede8; --text2:#b8b4ac; --text3:#6a6660;
--gold:#c9a96e; --gold2:#e8c990; --gold-dim:#3d2e18;
--ok:#2d5a3d; --ok-text:#7dd8a0;
--err:#5a2d2d; --err-text:#f09090;
```

### Fonty (Google Fonts)
- **Playfair Display** serif — tytuły, fiszki, nagłówki lekcji
- **Jost** sans-serif — UI, przyciski, tekst

### iOS wymagania
```css
html, body { background:#0f0f0f !important; color-scheme:dark; overscroll-behavior:none; }
html { min-height:100%; min-height:-webkit-fill-available; }
```

---

## Zasady pracy z kodem

### Zawsze
- Edytuj `index.html` bezpośrednio
- Po zmianie sprawdź: 1× `<style>`, 1× `<script>`, 1× `</html>` na końcu
- Sprawdź parowanie `{}` w JS
- Powiedz Aleksandrze żeby otworzyła plik w przeglądarce

### Nigdy
- Nie dziel na wiele plików
- Nie używaj frameworków
- Nie hardkoduj kluczy API
- Nie używaj `position:fixed`
- Nie duplikuj funkcji ani `const`
- Nie wklejaj JS za `</html>`

### Najczęstsze błędy (historia)
1. **Brakujący `@` przed `@media`** — psuje WSZYSTKIE style mobilne
2. **CSS wyciekający jako tekst** — brak `<style>`
3. **Ścięty `<link>` przy podmianie CSS** — nagłówek urwany
4. **JS za `</html>`** — przeglądarka ignoruje, wyświetla jako tekst
5. **Zdublowany `</head>` lub `</html>`**
6. **`position:fixed`** — modal znika w Safari
7. **`oninput` w ćwiczeniach fill** — zamyka klawiaturę iOS → użyj `onchange` + `onkeydown`
8. **Duplikaty `const`** — SyntaxError
9. **Stary format dialogu** — nie używaj `speaker/line/tr`, używaj `s/l/t`

---

## Planowane funkcje (TODO)

| Funkcja | Priorytet |
|---------|-----------|
| **PDF.js** — lokalna ekstrakcja tekstu, nie wysyłaj PDF do API | 🔴 wysoki |
| **Lepsza analiza PDF** — opus model + wymóg min. 15 słów | 🔴 wysoki |
| **Własne lekcje z PDF** — dodać przyciski Fiszki/Ćwiczenia | 🔴 wysoki |
| **YouTube transkrypcja** — RapidAPI z dwoma fallback endpointami | 🔴 wysoki |
| **Podświetlanie słów** — `.word-active` podczas odtwarzania | 🟡 średni |
| **Popup tłumaczenia** — klik na słowo → IPA + PL + "Znam/Uczę się" | 🟡 średni |
| **Zakładki w lekcji** — Dialog / Słówka / Gramatyka | 🟡 średni |
| **Lista słówek dwujęzyczna** — EN→PL i PL→EN w zakładce Słówka | 🟡 średni |
| **YouTube ID per klip** — konkretne filmy w Filmotece | 🟢 niski |
| **Git auto-push** — Claude Code commituje do GitHub | 🟢 niski |

---

## Kontekst użytkownika

- **Aleksandra** — uczy się EN/FR/IT, baza PL, osoba nietechniczna
- Urządzenia: iPhone Safari + Mac Chrome
- GitHub: `bosari-lab`
- Klucze: OpenAI ✅, Anthropic ✅, ElevenLabs ✅, RapidAPI ⏳
- Preferuje elegancki ciemny design — nie zmieniaj wyglądu bez pytania
