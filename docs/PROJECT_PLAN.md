# PROJECT_PLAN.md

**Praca inżynierska:** „Zastosowanie narzędzi sztucznej inteligencji do klasyfikacji kart do gry"
**Autor:** Bartosz Bujanowicz
**Uczelnia:** Polsko-Japońska Akademia Technik Komputerowych
**Wersja dokumentu:** 1.1
**Data:** 2026-09-11
**Status:** plan przyjęty do realizacji — obowiązuje od Fazy 1

---

## Jak czytać ten dokument

To jest **plan**, nie sprawozdanie. Nie zawiera żadnych wyników pomiarów — wszystkie miejsca,
w których docelowo znajdą się liczby, są oznaczone jako `[DO ZMIERZENIA]`.

Konwencje użyte w dokumencie:

| Oznaczenie | Znaczenie |
|---|---|
| `[DO ZMIERZENIA]` | wartość, którą trzeba uzyskać z rzeczywistego eksperymentu |
| `[DO WERYFIKACJI]` | fakt zewnętrzny (np. zawartość publicznego datasetu), który trzeba sprawdzić samodzielnie |
| `[DECYZJA]` | miejsce wymagające świadomego wyboru — z rekomendacją i uzasadnieniem |
| `Fx-Ty` | zadanie nr `y` w fazie nr `x` |
| `Rn` | ryzyko nr `n` z sekcji 18 |
| `Xn` | rozszerzenie opcjonalne nr `n` z sekcji 21 |
| **DoD** | *Definition of Done* — warunek uznania zadania lub fazy za ukończone |

**Koncepcja wyjściowa** — używane w tekście określenie wstępnego opisu projektu (proponowany
pipeline, struktura katalogów, wstępny spis treści pracy), sporządzonego przed powstaniem
tego planu. Miejsca, w których plan od niej odchodzi, są oznaczone jawnie i opatrzone
uzasadnieniem — sekcje 3.1, 8.1, 13.1, 17.0 i 19.1.

---

## Spis treści

1. [Opis problemu](#1-opis-problemu)
2. [Cele projektu](#2-cele-projektu)
3. [Analiza i wybór architektury](#3-analiza-i-wybór-architektury)
4. [Technologie](#4-technologie)
5. [Dataset](#5-dataset)
6. [Data augmentation](#6-data-augmentation)
7. [Korekcja perspektywy](#7-korekcja-perspektywy)
8. [Modele klasyfikacyjne](#8-modele-klasyfikacyjne)
9. [Confidence, kalibracja i obsługa błędów](#9-confidence-kalibracja-i-obsługa-błędów)
10. [Metryki](#10-metryki)
11. [Eksperymenty](#11-eksperymenty)
12. [Aplikacja](#12-aplikacja)
13. [Struktura repozytorium](#13-struktura-repozytorium)
14. [Reprodukowalność](#14-reprodukowalność)
15. [Testy](#15-testy)
16. [Wymagania sprzętowe](#16-wymagania-sprzętowe)
17. [Fazy implementacji](#17-fazy-implementacji)
18. [Ryzyka](#18-ryzyka)
19. [Powiązanie z pracą inżynierską](#19-powiązanie-z-pracą-inżynierską)
20. [Zakres MVP](#20-zakres-mvp)
21. [Rozszerzenia](#21-rozszerzenia)
22. [Rekomendowany plan i pierwszy krok](#22-rekomendowany-plan-i-pierwszy-krok)

---

## Założenia przyjęte na podstawie ustaleń

Plan został dostosowany do trzech ustaleń poczynionych przed jego napisaniem:

1. **Dane pochodzą głównie z publicznych datasetów**, uzupełnione własnymi zdjęciami tam,
   gdzie okaże się to konieczne. Sekcja 5 opiera się więc na strategii „publiczny dataset
   + własny zbiór testowy", a nie na budowaniu wszystkiego od zera. Dodano eksperyment
   **E11 (domain gap)**, który zamienia to ograniczenie w wartość naukową.

2. **Talie kart są już w posiadaniu.** Nie ma więc ryzyka opóźnienia po stronie sprzętu;
   w Fazie 1 pozostaje jedynie krótka **weryfikacja talii** pod kątem wymagań z sekcji 5.7
   (`F1-T6`). Fazy zależne od własnych zdjęć pozostają przesunięte na Fazę 7, ponieważ
   dopiero działający system pokazuje, jakiego materiału naprawdę potrzeba.

3. **Czas pracy jest bardzo nierówny** — zrywy zamiast równomiernego tempa. Konsekwencje
   dla całego planu:
   - zadania mają rozmiar **1–4 h**, żeby mieściły się w jednym wieczorze,
   - każda faza jest **niezależnym blokiem** kończącym się działającym artefaktem,
   - po każdej fazie powstaje wpis w dzienniku `docs/JOURNAL.md`, żeby powrót po
     dwutygodniowej przerwie nie wymagał odtwarzania kontekstu z pamięci,
   - kolejność faz jest tak dobrana, aby **po Fazie 6 istniał już działający system
     end-to-end** — nawet jeśli projekt utknie później, jest co pokazać i o czym pisać.

---

## 1. Opis problemu

### 1.1 Czym jest problem

Zadanie polega na automatycznym rozpoznaniu, **jakie karty do gry** i **w jakim miejscu**
znajdują się na obrazie z kamery lub w pliku graficznym. Wynikiem ma być lista kart
(np. `A♠`, `7♥`, `10♦`) wraz z ich położeniem i miarą pewności rozpoznania.

Formalnie łączymy dwa klasyczne zadania widzenia komputerowego:

- **detekcję obiektów** (*object detection*) — znalezienie i zlokalizowanie kart na obrazie,
- **klasyfikację obrazów** (*image classification*) — przypisanie każdej wykrytej karcie
  jednej z 52 etykiet.

### 1.2 Dlaczego to jest nietrywialne

Na pierwszy rzut oka karty wyglądają na obiekt łatwy: prostokątne, wysokokontrastowe,
o ustalonym rozmiarze i skończonej liczbie wariantów. W praktyce zadanie ma kilka
własności, które czynią je dobrym materiałem inżynierskim.

**a) Informacja rozpoznawcza jest skoncentrowana na małej powierzchni.**
O tożsamości karty decyduje przede wszystkim **indeks narożny** — figura i symbol koloru
w rogu — zajmujący około 3–5% powierzchni karty. Przy karcie zajmującej 100×140 px w kadrze
indeks ma około 10×20 px. To bezpośrednio przekłada się na wymagania co do rozdzielczości
wejściowej klasyfikatora i jest tematem eksperymentu **E8**.

**b) Klasy są wizualnie bardzo podobne.**
Różnica między `6♦` a `9♦` to obrót o 180°, a między `♠` a `♣` — detal kilkupikselowy.
Różnica między `10♥` a `10♦` sprowadza się wyłącznie do kształtu symbolu, przy identycznym
kolorze. Dzięki temu macierz pomyłek jest interesująca analitycznie, a nie tylko formalnym
wymogiem — rozdział o analizie wyników ma z czego korzystać.

**c) Kolor jest cechą krytyczną i jednocześnie najbardziej wrażliwą na oświetlenie.**
Rozróżnienie ♥/♦ od ♠/♣ opiera się głównie na barwie czerwonej. W ciepłym oświetleniu
żarowym czerń nabiera odcienia brązowego, a przy zimnym świetle LED czerwień traci
nasycenie. Ma to dwie konsekwencje: ostrożność przy augmentacji barwy (sekcja 6) oraz
sensowny eksperyment odpornościowy **E10a**.

**d) Karty są obiektami płaskimi obserwowanymi pod dowolnym kątem.**
Karta leżąca na stole i widziana z boku ulega silnemu skrótowi perspektywicznemu.
To otwiera drogę do klasycznego CV — transformacji perspektywicznej — i do eksperymentu **E5**.

**e) Karty na stole nakładają się na siebie.**
W realnych układach (rozdanie pokerowe, wachlarz w ręce) karty częściowo się zasłaniają.
Karty projektuje się tak, aby indeks narożny pozostawał widoczny w wachlarzu, ale detektor
musi radzić sobie z częściowym zasłonięciem — bada to **E10c**.

**f) Powierzchnia kart jest błyszcząca.**
Standardowe karty mają powłokę powodującą odblaski, które potrafią całkowicie zasłonić
indeks. To realne źródło błędów, które trzeba wykryć w analizie i opisać w ograniczeniach.

### 1.3 Dlaczego to jest problem AI, a nie tylko CV

Można wyobrazić sobie rozwiązanie czysto klasyczne: progowanie, wykrycie konturów,
wycięcie indeksu i dopasowanie do 52 szablonów metodą korelacji. Takie podejścia istnieją
i działają — **w kontrolowanych warunkach**. Ich słabości (wrażliwość na oświetlenie, tło,
wzór talii, odblaski, rozdzielczość) są dokładnie tym, co uczenie maszynowe adresuje.

Dlatego w tym projekcie **klasyczne CV nie jest odrzucone, lecz świadomie użyte jako punkt
odniesienia**. Porównanie „klasyczne CV kontra sieć neuronowa" na tym samym zbiorze
testowym (eksperyment **E7**) jest jednym z mocniejszych punktów pracy: pokazuje, że wybór
AI wynika z pomiaru, a nie z mody.

### 1.4 Zastosowania praktyczne

Kontekst, który warto przywołać we wstępie pracy:

- automatyczne sędziowanie i archiwizacja rozdań w turniejach kart,
- systemy antyoszustwowe w kasynach,
- narzędzia treningowe i analityczne dla graczy,
- aplikacje wspierające osoby niewidome i słabowidzące w grach karcianych,
- digitalizacja rozgrywek na potrzeby analizy statystycznej.

---

## 2. Cele projektu

### 2.1 Cel główny

Zaprojektowanie, implementacja i **eksperymentalna ewaluacja** kompletnego systemu
komputerowego rozpoznawania kart do gry, łączącego detekcję obiektów z klasyfikacją opartą
na konwolucyjnych sieciach neuronowych, działającego w czasie rzeczywistym z kamery
internetowej.

### 2.2 Cele szczegółowe

**Inżynierskie — system musi:**

| # | Cel | Gdzie weryfikowany |
|---|---|---|
| C1 | Wczytywać obraz z pliku oraz strumień z kamery | Faza 6, Faza 12 |
| C2 | Wykrywać wszystkie karty w kadrze i podawać ich położenie | Faza 5, metryki detekcji |
| C3 | Wycinać wykryte karty i normalizować je geometrycznie | Faza 3 |
| C4 | Klasyfikować każdą kartę do jednej z 52 klas | Faza 4, metryki klasyfikacji |
| C5 | Zwracać skalibrowaną miarę pewności i odrzucać niepewne rozpoznania | Faza 8, E9 |
| C6 | Obsługiwać wiele kart naraz, także częściowo zasłoniętych | E10c |
| C7 | Działać w czasie rzeczywistym, cel ≥ 15 FPS `[DO ZMIERZENIA]` | Faza 12 |
| C8 | Prezentować wyniki w GUI ze statystykami i historią | Faza 11 |
| C9 | Odmawiać odpowiedzi zamiast zgadywać, gdy dane są niewystarczające | Faza 8 |

**Badawcze — praca musi:**

| # | Cel |
|---|---|
| B1 | Porównać co najmniej trzy architektury klasyfikatora na wspólnym zbiorze testowym |
| B2 | Zbadać wpływ dekompozycji problemu (52 klasy kontra figura + kolor) na jakość i szybkość |
| B3 | Zmierzyć wpływ korekcji perspektywy na skuteczność klasyfikacji |
| B4 | Porównać architekturę jedno- i dwuetapową |
| B5 | Zbadać transfer learning względem uczenia od zera |
| B6 | Wyznaczyć próg pewności metodą opartą na danych, a nie arbitralnie |
| B7 | Zmierzyć różnicę skuteczności między danymi publicznymi a własnymi (domain gap) |
| B8 | Przeprowadzić analizę błędów w oparciu o macierz pomyłek |

**Metodologiczne — proces musi:**

| # | Cel |
|---|---|
| M1 | Każdy eksperyment odtwarzalny na podstawie zapisanej konfiguracji i ziarna losowego |
| M2 | Zbiór testowy nieużywany do strojenia hiperparametrów |
| M3 | Podział danych wykluczający przeciek informacji (sekcja 5.5) |
| M4 | Wszystkie liczby w pracy pochodzące z zapisanych, powtarzalnych uruchomień |

### 2.3 Czego projekt świadomie NIE obejmuje

Jasne wyznaczenie granic chroni przed rozrostem zakresu i jest wartościowym fragmentem
rozdziału o ograniczeniach:

- rozpoznawanie **rewersów** kart — system ma je wykrywać jako obiekty i odrzucać jako
  nierozpoznawalne, a nie identyfikować,
- rozpoznawanie jokerów i talii nietypowych (tarot, gry kolekcjonerskie),
- śledzenie kart w czasie i rekonstrukcja przebiegu gry (poza opcjonalnym rozszerzeniem X2),
- ocena układów pokerowych i logika reguł gry,
- praca na urządzeniach mobilnych i wdrożenie produkcyjne.

---

## 3. Analiza i wybór architektury

### 3.1 Trzy możliwe architektury

Przed przyjęciem pipeline'u z koncepcji wyjściowej warto rozważyć alternatywy. Istnieją trzy sensowne
podejścia do tego zadania.

#### Architektura A — jednoetapowa: YOLO z 52 klasami

```
OBRAZ → YOLO (52 klasy) → bounding box + klasa + confidence → WYNIK
```

Detektor jednocześnie lokalizuje i rozpoznaje kartę. To rozwiązanie najczęściej spotykane
w gotowych projektach internetowych.

**Zalety:** jeden model, jedno przejście przez sieć, najniższa latencja, najprostszy kod,
brak propagacji błędów między etapami.

**Wady:** wymaga anotacji bounding boxów **z podziałem na 52 klasy** — czyli dataset
detekcyjny musi mieć w każdym obrazie oznaczone, która to konkretnie karta. Silna
nierównowaga klas, bo w typowym zdjęciu jest kilka kart z 52 możliwych. Architektura jest
„czarną skrzynką" — nie da się zbadać wpływu korekcji perspektywy ani porównać
klasyfikatorów, bo klasyfikacja jest wtopiona w detektor. **Redukuje pracę inżynierską do
jednego eksperymentu: „wytrenowałem YOLO".**

#### Architektura B — dwuetapowa: detektor klasowo-agnostyczny + klasyfikator

```
OBRAZ → preprocessing → YOLO (1 klasa: „karta") → NMS → crop
      → wykrycie narożników → korekcja perspektywy → normalizacja
      → KLASYFIKATOR CNN (52 klasy) → softmax → kalibracja
      → decyzja accept/reject → agregacja → GUI
```

Detektor odpowiada wyłącznie na pytanie „gdzie jest karta", klasyfikator na pytanie
„jaka to karta".

**Zalety:**
- Dataset detekcyjny ma **jedną klasę** — anotacja jest wielokrotnie tańsza, a nierównowaga
  klas znika.
- Dwa niezależne moduły można **badać osobno**: dokładnie to, czego wymaga część
  eksperymentalna pracy (E1–E5, E7, E8).
- Klasyfikator uczy się na wyprostowanych, znormalizowanych wycinkach — łatwiejszy problem
  niż klasyfikacja w dowolnej perspektywie.
- **Pasuje do dostępnych danych publicznych** (sekcja 5): istnieją osobno gotowe zbiory
  klasyfikacyjne (wycinki kart, 52–53 klasy) i osobno zbiory detekcyjne. Architektura
  dwuetapowa pozwala wykorzystać jedne i drugie bez konwersji.
- Moduł detekcji można podmienić na klasyczne CV bez dotykania klasyfikatora — stąd
  bezpłatny baseline do E7 i awaryjne wyjście, gdyby trening YOLO się nie powiódł.

**Wady:** dwa modele do wytrenowania i utrzymania, wyższa latencja (detekcja + N klasyfikacji),
błędy detekcji propagują się dalej (nieznaleziona karta jest nie do odzyskania).

#### Architektura C — hybrydowa: detekcja narożników / punktów kluczowych

```
OBRAZ → sieć wykrywająca 4 narożniki każdej karty → homografia → klasyfikator
```

Zamiast bounding boxa sieć przewiduje cztery punkty kluczowe, co daje korekcję perspektywy
„za darmo" i dokładniejsze wycinki dla kart pod dużym kątem.

**Zalety:** najdokładniejsza geometria, elegancko rozwiązuje problem obróconych kart
(bounding box obróconej karty zawiera dużo tła).

**Wady:** anotacja punktów kluczowych jest droższa niż bounding boxów, **nie ma gotowych
publicznych datasetów w tym formacie**, a implementacja jest wyraźnie trudniejsza. Przy
ograniczonym i nierównym budżecie czasu to zbyt duże ryzyko.

### 3.2 Porównanie

| Kryterium | A — jednoetapowa | B — dwuetapowa | C — punkty kluczowe |
|---|---|---|---|
| Trudność implementacji | niska | średnia | wysoka |
| Koszt anotacji własnych danych | wysoki (52 klasy) | niski (1 klasa) | bardzo wysoki |
| Dostępność danych publicznych | średnia | **wysoka** | bardzo niska |
| Latencja | najniższa | średnia | średnia |
| Modularność i testowalność | niska | **wysoka** | średnia |
| Liczba możliwych eksperymentów | 2–3 | **10+** | 4–5 |
| Jakość przy kartach pod dużym kątem | średnia | dobra (z korekcją) | najlepsza |
| Ryzyko projektowe | niskie | niskie | **wysokie** |
| Wartość dla pracy inżynierskiej | niska | **wysoka** | wysoka, ale ryzykowna |

### 3.3 `[DECYZJA]` Rekomendacja

**Architektura B (dwuetapowa) jako główna, architektura A jako baseline porównawczy.**

Uzasadnienie:

1. **Architektura B jest jedyną, która umożliwia zaplanowaną część eksperymentalną.**
   Eksperymenty E1, E2, E3, E5, E8 wymagają możliwości wymiany samego klasyfikatora
   przy niezmienionym detektorze. W architekturze A to niewykonalne.

2. **Architektura B najlepiej pasuje do dostępnych źródeł danych.** Publiczne zbiory
   klasyfikacyjne (gotowe wycinki kart) można wykorzystać bezpośrednio, bez ręcznej
   anotacji bounding boxów dla 52 klas.

3. **Architektura A nie znika z projektu — zostaje jako punkt odniesienia.** Eksperyment
   **E6** porównuje ją z B pod względem dokładności end-to-end i FPS. Dzięki temu wybór
   architektury zostaje w pracy **uzasadniony pomiarem**, a nie deklaracją. To jeden
   z mocniejszych fragmentów rozdziału o projekcie systemu.

4. **Architektura C zostaje opisana w pracy jako rozważona alternatywa** wraz z powodem
   odrzucenia (koszt anotacji, brak danych, ryzyko czasowe) i wskazana w rozdziale
   „możliwości rozwoju". To dokładnie ten rodzaj analizy, którego oczekuje się w pracy
   inżynierskiej.

### 3.4 Docelowy pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│  ŹRÓDŁO:  plik obrazu  │  katalog  │  wideo  │  kamera (webcam)  │
└──────────────────────────────┬──────────────────────────────────┘
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│  PREPROCESSING                                                   │
│  zmiana rozmiaru z zachowaniem proporcji (letterbox),           │
│  konwersja BGR→RGB, normalizacja                                 │
│  opcjonalnie: CLAHE, korekta gamma  → badane w E10a              │
└──────────────────────────────┬──────────────────────────────────┘
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│  DETEKCJA — interfejs CardDetector (wymienny)                    │
│    ├─ ContourDetector   (klasyczne CV — baseline, E7)            │
│    └─ YoloDetector      (YOLO, 1 klasa „karta" — produkcyjny)    │
│  wynik: lista (bbox, det_confidence)                             │
│  filtracja: próg pewności, NMS, filtr proporcji i pola bboxa     │
└──────────────────────────────┬──────────────────────────────────┘
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│  WYCIĘCIE (crop) z marginesem, obcięcie do granic obrazu         │
└──────────────────────────────┬──────────────────────────────────┘
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│  KOREKCJA PERSPEKTYWY  (opcjonalna — przełącznik, E5)            │
│  kontur → approxPolyDP → 4 narożniki → uporządkowanie            │
│  → cv2.getPerspectiveTransform → warpPerspective                 │
│  → kanoniczny obraz karty (pion, proporcja ok. 1 : 1,4)          │
│  fallback: gdy nie znaleziono czworokąta → prosty crop           │
└──────────────────────────────┬──────────────────────────────────┘
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│  KLASYFIKACJA — interfejs CardClassifier (wymienny)              │
│    ├─ SimpleCNN         (własna sieć — baseline, E1)             │
│    ├─ ResNet18          (transfer learning, E1/E2)               │
│    ├─ MobileNetV3-Small (wariant szybki, E1)                     │
│    └─ MultiHead         (wspólny backbone, głowy 13 + 4, E3)     │
│  wynik: rozkład prawdopodobieństw po klasach                     │
└──────────────────────────────┬──────────────────────────────────┘
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│  KALIBRACJA I DECYZJA                                            │
│  temperature scaling → skalibrowane prawdopodobieństwo           │
│  próg wyznaczony z krzywej ryzyko–pokrycie (E9)                  │
│  wynik: PEWNA KLASYFIKACJA  |  NIEPEWNA KLASYFIKACJA             │
└──────────────────────────────┬──────────────────────────────────┘
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│  AGREGACJA I ANALIZA                                             │
│  deduplikacja kart, statystyki sesji, opcjonalnie: głosowanie    │
│  po klatkach w trybie wideo (X2), analiza pokrycia talii (X1)    │
└──────────────────────────────┬──────────────────────────────────┘
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│  PREZENTACJA: GUI (PySide6) │ CLI │ zapis wyników do JSON/CSV    │
└─────────────────────────────────────────────────────────────────┘
```

**Kluczowa cecha projektowa:** każdy blok o wymiennej implementacji (detekcja,
klasyfikacja, korekcja perspektywy) jest ukryty za interfejsem abstrakcyjnym i wybierany
z pliku konfiguracyjnego. Dzięki temu eksperyment to **zmiana konfiguracji, a nie
przepisywanie kodu** — co przy nierównym budżecie czasu jest różnicą między wykonalnym
a niewykonalnym planem badawczym.

---

## 4. Technologie

### 4.1 Język i środowisko

| Kryterium | Python | C++ / OpenCV | MATLAB |
|---|---|---|---|
| Ekosystem CV/DL | najbogatszy | dobry | ograniczony |
| Szybkość prototypowania | wysoka | niska | średnia |
| Dostępność modeli pretrenowanych | pełna | ograniczona | ograniczona |
| Wydajność inferencji | wystarczająca (backend C++) | najwyższa | niska |
| Licencja | otwarta | otwarta | komercyjna |

**Wybór: Python.** Uzasadnienie: PyTorch, Ultralytics, OpenCV i scikit-learn eksponują
interfejs pythonowy, a właściwe obliczenia i tak wykonują w skompilowanym kodzie C++/CUDA.
Narzut interpretera jest pomijalny wobec czasu inferencji sieci.

`[DECYZJA]` **Wersja Pythona: 3.12.**
Na maszynie wykryto **Python 3.14.7**. PyTorch i Ultralytics nie publikują dla tej wersji
stabilnych pakietów binarnych `[DO WERYFIKACJI — sprawdzić aktualny stan przed instalacją]`,
a kompilacja ze źródeł na Windows jest bardzo czasochłonna. Projekt należy prowadzić
w osobnym środowisku wirtualnym na Pythonie 3.12, nie ruszając systemowej instalacji 3.14.
To zadanie `F1-T1`.

### 4.2 Framework uczenia głębokiego

| Kryterium | PyTorch | TensorFlow / Keras |
|---|---|---|
| Czytelność kodu treningowego | wysoka (pętla jawna) | niższa (`model.fit`) |
| Kontrola nad eksperymentem | pełna | ograniczona bez callbacków |
| Integracja z Ultralytics YOLO | natywna | brak |
| Materiał do opisu w pracy | jawna pętla ucząca dobrze się opisuje | mniej |
| Wsparcie CUDA na Windows | dobre | słabsze |

**Wybór: PyTorch.** Decydujący jest trzeci wiersz — Ultralytics YOLO jest zbudowany na
PyTorchu, więc jeden framework obsługuje oba modele. Dodatkowo jawna pętla ucząca jest
łatwiejsza do opisania w rozdziale implementacyjnym niż wywołanie `model.fit()`.

### 4.3 Detektor obiektów

Rozważane rodziny modeli:

| Model | Typ | Szybkość | Trudność treningu | Uwagi |
|---|---|---|---|---|
| Faster R-CNN | dwuetapowy | niska | średnia | wysoka dokładność, nie nadaje się do real-time na tym sprzęcie |
| SSD / RetinaNet | jednoetapowy | średnia | średnia | wypierane przez nowsze rodziny |
| YOLO (Ultralytics) | jednoetapowy | **wysoka** | **niska** | gotowy pipeline treningowy, eksport, ewaluacja |
| RT-DETR | transformer | średnia | wysoka | wymaga więcej danych, dłuższy trening |
| Klasyczne kontury (OpenCV) | brak uczenia | bardzo wysoka | brak | baseline, brak odporności |

`[DECYZJA]` **YOLO w wariancie `n` (nano) jako model produkcyjny, wariant `s` (small)
jako porównanie, klasyczne kontury jako baseline.**

Uzasadnienie doboru rozmiaru — celowo nie „bo YOLO jest popularne":

1. **Zadanie detekcji jest w tym projekcie łatwe.** Wykrywamy **jedną klasę** obiektów
   dużych, wysokokontrastowych, o sztywnym kształcie prostokątnym i stałej proporcji boków.
   To najprostszy możliwy scenariusz detekcji. Model o dużej pojemności nie ma tu czego
   dołożyć, a zwiększa ryzyko przeuczenia na małym zbiorze.
2. **Wymóg czasu rzeczywistego.** Wariant nano na RTX 3060 Ti pozostawia budżet czasowy dla
   N klasyfikacji na klatkę — w architekturze dwuetapowej to detekcja i klasyfikacja dzielą
   się budżetem 66 ms przy 15 FPS `[DO ZMIERZENIA]`.
3. **Krótki czas treningu** oznacza możliwość przeprowadzenia wielu przebiegów
   eksperymentalnych w nierównym budżecie czasu.
4. **Porównanie `n` kontra `s` jest samo w sobie wynikiem** — jeśli wariant `s` nie poprawia
   mAP, mamy empiryczne potwierdzenie tezy z punktu 1 i dobry akapit do pracy.

`[DO WERYFIKACJI]` Konkretna generacja (YOLOv8 / YOLO11 / nowsza) — wybrać najnowszą
stabilną w Ultralytics w momencie rozpoczęcia Fazy 5 i **zapisać dokładną wersję pakietu**
w `requirements.txt` oraz w konfiguracji eksperymentu. Zmiana generacji w trakcie projektu
unieważnia porównywalność wyników.

> **Uwaga licencyjna — istotna dla pracy dyplomowej.**
> Pakiet Ultralytics jest udostępniany na licencji **AGPL-3.0** (z płatną alternatywą
> komercyjną). Dla pracy inżynierskiej i projektu niekomercyjnego nie stanowi to przeszkody,
> ale **licencję należy wymienić w pracy** w rozdziale o wykorzystanych narzędziach.
> Jeżeli licencja copyleft okazałaby się problemem, alternatywą jest własna implementacja
> detektora w czystym PyTorchu lub pozostanie przy detektorze konturowym — dlatego moduł
> detekcji jest w projekcie wymienny. `[DO WERYFIKACJI — potwierdzić aktualną licencję]`

### 4.4 Klasyczne przetwarzanie obrazu

**OpenCV** — bezalternatywny wybór dla operacji progowania, wykrywania konturów,
transformacji perspektywicznej i obsługi kamery. Dojrzały, dobrze udokumentowany,
z licencją Apache 2.0.

### 4.5 Interfejs graficzny

| Biblioteka | Wygląd | Wideo w czasie rzeczywistym | Krzywa uczenia | Licencja |
|---|---|---|---|---|
| Tkinter | przestarzały | słabe | najniższa | wbudowana |
| **PySide6 (Qt)** | **profesjonalny** | **bardzo dobre** | średnia | **LGPL** |
| PyQt6 | profesjonalny | bardzo dobre | średnia | GPL/komercyjna |
| Streamlit / Gradio | nowoczesny | ograniczone | najniższa | Apache 2.0 |
| Kivy | nietypowy | dobre | wysoka | MIT |

**Wybór: PySide6.** Uzasadnienie:
- w odróżnieniu od PyQt6 licencja **LGPL** nie nakłada wymogu otwarcia kodu aplikacji,
- obsługa strumienia wideo w osobnym wątku z sygnałami/slotami Qt jest wzorcowa i dobrze
  się opisuje w rozdziale implementacyjnym (problem synchronizacji GUI z akwizycją klatek
  to konkretna treść inżynierska),
- aplikacja desktopowa lepiej pasuje do scenariusza „kamera + stół" niż aplikacja webowa,
- Streamlit odpada, bo strumień z kamery w czasie rzeczywistym jest w nim niewygodny
  i dodaje zależność od przeglądarki.

**Ograniczenie ryzyka:** logika systemu jest w całości w module `pipeline`, a GUI jest
tylko cienką warstwą prezentacji. Jeśli Faza 11 okaże się czasochłonna, projekt pozostaje
w pełni funkcjonalny przez CLI, a GUI można ograniczyć do minimum.

### 4.6 Pozostałe biblioteki i uzasadnienie każdej z nich

| Biblioteka | Do czego | Dlaczego niezbędna |
|---|---|---|
| NumPy | operacje na tablicach | podstawa całego stosu |
| Albumentations | augmentacja obrazów | augmentacje geometryczne z jednoczesną transformacją bboxów; szybsza niż torchvision |
| scikit-learn | metryki, macierz pomyłek, kalibracja | gotowe, sprawdzone implementacje — nie ma sensu pisać własnych |
| Matplotlib | wykresy do pracy | pełna kontrola nad wyglądem rysunku publikacyjnego |
| Pandas | zestawienia wyników eksperymentów | agregacja wyników z wielu przebiegów do tabel |
| PyYAML | konfiguracje | format czytelny i wersjonowalny w gicie |
| pytest | testy | standard w ekosystemie Pythona |
| tqdm | paski postępu | drobiazg, ale przy długich treningach realnie potrzebny |

**Biblioteki świadomie odrzucone** (zgodnie z zasadą „nie komplikuj bez powodu"):

| Odrzucone | Powód |
|---|---|
| PyTorch Lightning | ukrywa pętlę uczącą, którą chcemy opisać w pracy; dodaje warstwę abstrakcji bez zysku przy dwóch modelach |
| Weights & Biases / MLflow | zależność od usługi zewnętrznej; katalog `experiments/` z plikami YAML i CSV w gicie w pełni wystarcza i jest bardziej reprodukowalny |
| Hydra | nadmiarowa wobec prostego wczytywania YAML-a |
| DVC | sensowne przy dużych zespołach; tutaj wystarczy zapis identyfikatorów i sum kontrolnych datasetu |
| Docker | dodatkowa złożoność bez korzyści przy jednym stanowisku; `requirements.txt` z przypiętymi wersjami wystarcza |
| ONNX Runtime / TensorRT | tylko jeśli okaże się, że nie osiągamy celu FPS — rozszerzenie X4, nie element MVP |

---

## 5. Dataset

> Sekcja dostosowana do ustalenia: **dane pochodzą głównie z publicznych datasetów**,
> uzupełnione własnymi zdjęciami tam, gdzie to konieczne.

### 5.1 Strategia trójźródłowa

Oparcie się wyłącznie na pobranym datasecie ma jedną poważną wadę: praca inżynierska
sprowadzałaby się wtedy do „pobrałem gotowy zbiór i wytrenowałem model", co jest słabym
wynikiem niezależnie od osiągniętej dokładności. Jednocześnie budowanie całego zbioru od
zera przy nierównym budżecie czasu jest nierealne.

Rozwiązaniem jest podział ról między trzy źródła danych:

| Źródło | Rola | Nakład pracy | Udział |
|---|---|---|---|
| **A. Publiczne datasety** | zbiór **treningowy** i walidacyjny — masa danych | niski (pobranie + audyt) | ~80% |
| **B. Dane syntetyczne** (kompozycja) | rozszerzenie zbioru detekcyjnego o sceny wielokartowe | średni (skrypt generujący) | ~15% |
| **C. Własne zdjęcia** | **zbiór testowy** — pomiar rzeczywistej skuteczności | średni (2–3 sesje) | ~5% |

**To jest kluczowa decyzja metodologiczna całego projektu.** Zbiór testowy pochodzący
z innego źródła niż treningowy oznacza, że mierzymy **rzeczywistą zdolność generalizacji**,
a nie dopasowanie do rozkładu danych publicznych. Różnica między wynikiem na teście
publicznym a na teście własnym to **domain gap** — i to jest samodzielny, wartościowy wynik
naukowy (eksperyment **E11**), a nie porażka projektu.

Trudność zamieniona w wartość: gdyby wszystkie dane pochodziły z jednego źródła, praca nie
miałaby czego zmierzyć w tym obszarze.

### 5.2 Kandydaci na publiczne zbiory danych

`[DO WERYFIKACJI]` — poniższe pozycje należy sprawdzić samodzielnie w Fazie 2. Podane są
jako kierunek poszukiwań, **nie jako potwierdzone fakty**; liczby obrazów i klas trzeba
odczytać ze źródła, nie przyjmować z tego dokumentu.

**Do klasyfikacji (wycinki pojedynczych kart):**

| Kandydat | Gdzie szukać | Co sprawdzić |
|---|---|---|
| Zbiory typu „Cards Image Dataset — Classification" | Kaggle | liczba klas (uwaga na 53 z jokerem — trzeba usunąć), rozdzielczość, czy podział train/val/test już istnieje, czy obrazy nie są augmentowanymi kopiami |
| Wycinki wygenerowane z datasetów detekcyjnych | Roboflow Universe | możliwość wycięcia kart po bboxach i zbudowania własnego zbioru klasyfikacyjnego |
| Skany/rendery talii | Wikimedia Commons, projekty open source z grafiką kart | idealne jako źródło do generowania danych syntetycznych (punkt 5.4) |

**Do detekcji (sceny z bounding boxami):**

| Kandydat | Gdzie szukać | Co sprawdzić |
|---|---|---|
| Zbiory „playing cards detection" | Roboflow Universe | format anotacji (YOLO/COCO), liczba klas, licencja, czy zbiór nie jest już zaugmentowany |
| Projekty generujące dane syntetyczne kart | GitHub | metoda kompozycji, możliwość ponownego użycia pomysłu |

**Kryteria wyboru zbioru — w kolejności ważności:**

1. **Licencja** pozwalająca na użycie w pracy dyplomowej i jasno wskazane źródło.
2. **Brak wstępnej augmentacji.** Wiele zbiorów z platform typu Roboflow jest publikowanych
   już po augmentacji, co uniemożliwia uczciwe przeprowadzenie eksperymentu E4 i grozi
   przeciekiem (punkt 5.3).
3. **Standardowa talia 52 kart** o typowym kroju, zbliżonym do talii posiadanej (sekcja 5.7).
4. **Rozdzielczość** wystarczająca, by indeks narożny był czytelny.
5. **Zrównoważenie klas** — zbliżona liczba przykładów na kartę.

### 5.3 Audyt datasetu — obowiązkowy krok przed treningiem

To najczęściej pomijany etap w projektach studenckich i jednocześnie ten, który najbardziej
podnosi wiarygodność pracy. Publiczne zbiory bywają wadliwe, a wady te **zawyżają wyniki**.

Skrypt `scripts/audit_dataset.py` musi sprawdzić i zaraportować:

| Kontrola | Dlaczego to ma znaczenie | Jak sprawdzić |
|---|---|---|
| **Duplikaty i near-duplicates** | ten sam obraz w zbiorze treningowym i testowym zawyża dokładność o kilka do kilkunastu punktów procentowych | hash percepcyjny (pHash/dHash), próg odległości Hamminga |
| **Augmentowane kopie rozrzucone po podziałach** | ten sam obraz obrócony trafia do train, a jego oryginał do test — klasyczny przeciek w zbiorach z Roboflow | analiza nazw plików i porównanie pHash |
| **Rozkład klas** | nierównowaga wpływa na interpretację dokładności | histogram liczby przykładów na klasę |
| **Błędne etykiety** | zbiory internetowe zawierają pomyłki | ręczny przegląd losowej próbki (np. 100 obrazów) i macierz pomyłek po pierwszym treningu |
| **Rozdzielczość i proporcje** | zbyt małe wycinki uniemożliwiają odczyt indeksu | statystyki rozmiarów |
| **Obecność jokerów i kart spoza standardu** | 53. klasa psuje spójność problemu | przegląd listy klas |
| **Poprawność bboxów (detekcja)** | boxy poza kadrem, zerowe pola, złe klasy | walidacja geometryczna |

**DoD audytu:** raport `docs/DATASET.md` zawierający wyniki wszystkich powyższych kontroli
z konkretnymi liczbami `[DO ZMIERZENIA]` oraz decyzję: zbiór przyjęty, przyjęty po
czyszczeniu, albo odrzucony.

Sam audyt jest materiałem na **podrozdział pracy** — pokazuje warsztat i świadomość
metodologiczną.

### 5.4 Dane syntetyczne — kompozycja scen

Publiczne zbiory klasyfikacyjne dostarczają wycinków pojedynczych kart, ale scen
wielokartowych w różnych układach zwykle brakuje. Zamiast fotografować tysiące układów
ręcznie, można je **wygenerować**:

```
wycinek karty (znany label)  +  losowe tło  +  losowa homografia
+  losowe oświetlenie  +  losowe przesłonięcie innymi kartami
                        ↓
    scena wielokartowa Z AUTOMATYCZNIE WYLICZONYM bounding boxem
```

Bounding box wynika z przekształcenia narożników karty przez tę samą homografię, którą
nałożono na obraz — anotacja jest więc **darmowa i idealnie dokładna**.

**Co daje:**
- dowolnie dużo scen detekcyjnych bez ręcznej anotacji,
- pełną kontrolę nad trudnością (stopień zasłonięcia, kąt, oświetlenie) — co bezpośrednio
  zasila eksperymenty **E10b** i **E10c**,
- zrównoważony rozkład klas.

**Czego nie daje i o czym trzeba uczciwie napisać:**
- brak realistycznych odblasków, głębi ostrości, szumu sensora i rozmycia ruchu,
- „przyklejony" wygląd kart na tle (brak cieni kontaktowych, jeśli ich nie zasymulujemy),
- ryzyko, że model nauczy się artefaktów generatora zamiast cech kart.

Dlatego dane syntetyczne służą **wyłącznie do treningu**, nigdy do testowania, a ich wpływ
jest przedmiotem osobnego eksperymentu **E12**.

**Rekomendowane elementy generatora** (`src/cardvision/synth/`):
- tła: fotografie różnych powierzchni (stół, obrus, blat, dywan) — z publicznych zbiorów
  tekstur lub własnych zdjęć,
- losowa homografia w zakresie odpowiadającym kątom obserwacji 0–60° od pionu,
- losowa jasność, kontrast i temperatura barwowa w umiarkowanym zakresie,
- symulacja cienia pod kartą (rozmyty ciemny kształt z przesunięciem),
- losowe nakładanie 1–10 kart z kontrolowanym stopniem zasłonięcia.

### 5.5 Podział na zbiory — i jak nie popsuć wyników

**Zasada nadrzędna: podział musi być rozłączny na poziomie *sesji/źródła*, nie pojedynczego
obrazu.**

Najczęstsze błędy, których trzeba uniknąć:

| Błąd | Skutek | Zapobieganie |
|---|---|---|
| Losowy podział klatek z jednego nagrania wideo | sąsiednie klatki są niemal identyczne — obraz z train i jego bliźniak z test; dokładność zawyżona nawet o kilkanaście punktów | podział **po nagraniu**, nie po klatce |
| Ten sam fizyczny egzemplarz karty w train i test | model uczy się zarysowań i zagięć konkretnego egzemplarza | jeśli w posiadaniu są dwie talie: talia A do treningu, talia B do testu |
| Augmentowane kopie w różnych podziałach | przeciek | audyt pHash z punktu 5.3 |
| Strojenie hiperparametrów na zbiorze testowym | zawyżony, nieuczciwy wynik końcowy | zbiór testowy używany **wyłącznie raz**, na końcu każdej fazy eksperymentalnej |

**Docelowy podział:**

| Zbiór | Źródło | Proporcja | Do czego |
|---|---|---|---|
| `train` | publiczne + syntetyczne | ~70% danych publicznych | uczenie |
| `val` | publiczne (rozłącznie ze źródłem train) | ~15% | wybór modelu, early stopping, strojenie |
| `test_public` | publiczne (rozłącznie) | ~15% | porównywalność z literaturą |
| `test_own` | **wyłącznie własne zdjęcia** | osobny zbiór | **główny wynik pracy** — rzeczywista skuteczność |

Podział zapisujemy jako listy plików w `data/splits/*.json` (wersjonowane w gicie —
patrz `.gitignore`), a nie jako fizyczne kopiowanie plików do katalogów. Dzięki temu
podział jest odtwarzalny i widoczny w historii zmian.

### 5.6 Warianty rozmiaru zbioru

#### Wariant minimalny — prototyp (Fazy 3–4)

| Element | Ilość | Skąd |
|---|---|---|
| Wycinki kart do klasyfikacji | ~50–100 na klasę × 52 klasy | publiczny dataset |
| Sceny detekcyjne | ~300 | publiczny dataset |
| Własny zbiór testowy | ~50 zdjęć | telefon, jedna sesja |

Cel: udowodnić, że pipeline działa. Wyniki nie nadają się do pracy, służą wyłącznie
weryfikacji implementacji.

#### Wariant docelowy — praca inżynierska (rekomendowany)

| Element | Ilość | Skąd | Szacowany nakład |
|---|---|---|---|
| Wycinki kart do klasyfikacji | wszystko, co daje wybrany zbiór po czyszczeniu, minimum ~150 na klasę | publiczny | 3–4 h (pobranie + audyt) |
| Sceny detekcyjne rzeczywiste | ~500–1000 | publiczny | 2 h |
| Sceny detekcyjne syntetyczne | ~5000 | generator | 4 h (napisanie skryptu), potem minuty |
| **Własny zbiór testowy** | **~300–400 wycinków kart + ~100 scen** | **własne zdjęcia** | **6–8 h w 2–3 sesjach** |

**Jak zebrać własny zbiór testowy w 6–8 h — metoda oparta na wideo:**

Fotografowanie 52 kart pojedynczo jest żmudne. Zamiast tego nagrywa się **krótkie filmy**
i wyodrębnia z nich klatki:

1. Rozłóż wszystkie 52 karty odkryte na stole w siatce.
2. Nagraj 30–60 s wideo, powoli przesuwając telefon nad stołem, zmieniając kąt i wysokość.
3. Powtórz w 3–4 różnych warunkach: światło dzienne, żarowe ciepłe, LED zimne, przyciemnione.
4. Powtórz na 2–3 różnych tłach (jasny blat, ciemny obrus, wzorzysta powierzchnia).
5. Nagraj kilka scen z układami: 2 karty, 5 kart, wachlarz w ręce, karty częściowo nachodzące.
6. Skryptem wyodrębnij co N-tą klatkę (`scripts/extract_frames.py`).

Jedna 45-sekundowa sesja przy próbkowaniu co 15. klatkę daje ~90 obrazów.
Osiem sesji to ~700 obrazów przy około 2 h nagrywania.

> **Uwaga krytyczna:** klatki z jednego nagrania **muszą trafić w całości do jednego
> podziału**. Metadane sesji (identyfikator nagrania, oświetlenie, tło, talia) zapisujemy
> w `data/metadata/sessions.csv` — to jednocześnie dane wejściowe do eksperymentów
> E10a i E10b, które wymagają pogrupowania wyników według warunków.

Anotacja własnych scen detekcyjnych: przy jednej klasie „karta" oznaczenie 100 obrazów
w narzędziu typu LabelImg / CVAT zajmuje około 1,5–2 h. To akceptowalny koszt — i dokładnie
tu widać oszczędność wynikającą z wyboru architektury dwuetapowej. Przy architekturze
jednoetapowej trzeba by oznaczać każdą kartę jedną z 52 etykiet.

#### Wariant rozbudowany — jeśli zostanie czas

- druga i trzecia talia o odmiennym kroju → eksperyment generalizacji międzytaliowej,
- własny zbiór treningowy → porównanie „trening na danych publicznych" kontra „trening na
  własnych" (rozszerzenie E11),
- nagrania z samego webcama → domknięcie pętli: trening i test na docelowym sprzęcie.

### 5.7 Talie użyte w projekcie — weryfikacja (`F1-T6`)

Talie są już w posiadaniu, więc zadanie sprowadza się do **opisania ich i sprawdzenia, czy
nie mają cech unieważniających założenia planu**. Weryfikację wykonuje się raz, w Fazie 1,
a jej wynik zapisuje w `docs/DATASET.md` — bo od właściwości talii zależą trzy rzeczy:
zasadność augmentacji obrotem o 180° (sekcja 6.2), porównywalność z publicznymi zbiorami
(sekcja 5.2) i interpretacja wyników na własnym zbiorze testowym.

**Lista kontrolna dla każdej posiadanej talii:**

| Cecha | Wartość oczekiwana | Konsekwencja odstępstwa |
|---|---|---|
| Rozmiar | poker size, 63×88 mm | bridge size (57×89 mm) zmienia proporcje wycinka — trzeba to uwzględnić przy normalizacji geometrycznej |
| Krój indeksów | anglo-amerykański, indeksy standardowej wielkości | indeksy „dla seniorów" mają inny rozkład cech niż dane publiczne — powiększa domain gap mierzony w E11 |
| Kolory indeksów | dwa (czerwony, czarny) | talia czterokolorowa (pik czarny, kier czerwony, karo niebieskie, trefl zielony) **całkowicie zmienia zadanie rozpoznawania koloru** — nie nadaje się jako talia główna |
| Symetria obrotu o 180° | figury dwugłowe, oczka symetryczne | asymetria unieważnia augmentację obrotem (sekcja 6.2) — trzeba ją wyłączyć |
| Powierzchnia | standardowa powłoka papierowa | talie plastikowe o silnym połysku dają nadmierne odblaski; jeśli taka jest w użyciu, należy to odnotować jako warunek akwizycji w `sessions.csv` |

**Jak wykorzystać posiadane talie:**

- **Talia główna** — ta, która najlepiej spełnia powyższą listę; służy do nagrań
  składających się na własny zbiór testowy.
- **Talia druga (jeśli jest dostępna i ma odmienny krój figur lub rewers)** — używana
  **wyłącznie w zbiorze testowym**, do pomiaru generalizacji międzytaliowej (rozszerzenie
  X3, ryzyko R13). Jeśli dostępna jest tylko jedna talia, X3 odpada, a ograniczenie
  „wyniki zmierzone na jednym wzorze talii" trafia wprost do rozdziału o ograniczeniach
  pracy — jest to uczciwy wynik, nie brak.

`[DO WERYFIKACJI]` — liczba i właściwości posiadanych talii; uzupełnić po wykonaniu `F1-T6`.

### 5.8 Licencje i uczciwość źródeł

Dla każdego użytego zbioru w `docs/DATASET.md` należy odnotować: nazwę, adres źródła,
autora, licencję, datę pobrania, liczbę obrazów i sumę kontrolną archiwum. W pracy
inżynierskiej musi znaleźć się podrozdział z tym zestawieniem. Zbiory o nieokreślonej
licencji należy odrzucić — ryzyko formalne przewyższa korzyść.

---

## 6. Data augmentation

### 6.1 Zasada

Augmentacja ma symulować **zmienność, która rzeczywiście występuje w docelowym
zastosowaniu**. Transformacja tworząca obrazy niemożliwe w rzeczywistości nie zwiększa
odporności — zużywa pojemność modelu na uczenie się sytuacji, które nigdy nie nastąpią,
i może aktywnie szkodzić.

Dlatego **żadna augmentacja nie jest włączana automatycznie**. Każda poniżej jest oceniona
osobno.

### 6.2 Analiza poszczególnych transformacji

#### Obrót o 180° — **TAK, kluczowa**

- **Sens:** karta leżąca na stole jest równie prawdopodobna „do góry nogami".
- **Dlaczego bezpieczna:** standardowe karty są projektowane jako **symetryczne względem
  obrotu o 180°** — figury są dwugłowe, a układ oczek na kartach numerycznych jest
  zaprojektowany tak, by po obrocie wyglądał tak samo. Obrót o 180° daje więc obraz
  wizualnie prawidłowy.
- **Ryzyko:** żadne, przy założeniu standardowej talii. **`[DO WERYFIKACJI]` — sprawdź
  wizualnie na posiadanych taliach (`F1-T6`), czy obrót o 180° faktycznie nie zmienia
  wyglądu**; nietypowe talie bywają asymetryczne.
- **Parametry:** prawdopodobieństwo 0,5.

#### Odbicie lustrzane poziome / pionowe — **NIE**

To pytanie pojawia się w koncepcji wyjściowej i odpowiedź jest jednoznacznie negatywna.

- **Symbole kolorów są symetryczne** względem odbicia pionowej osi (♥, ♦, ♠, ♣ — każdy
  z nich wygląda po odbiciu tak samo lub prawie tak samo). Gdyby chodziło tylko o kolory,
  odbicie byłoby nieszkodliwe.
- **Ale indeksy nie są.** Litery `A`, `J`, `Q`, `K` oraz cyfry `2`, `3`, `4`, `5`, `6`,
  `7`, `9` po odbiciu lustrzanym stają się znakami, które **nie istnieją na żadnej karcie**.
  Model uczyłby się rozpoznawać lustrzane `7` jako `7`, co osłabia cechę faktycznie
  odróżniającą figury.
- **Dodatkowe ryzyko:** odbicie mogłoby uczynić lustrzane `2` łudząco podobnym do innych
  znaków, wprowadzając sztuczne pomyłki.
- **Ważne rozróżnienie:** obrót o 180° to złożenie odbicia poziomego i pionowego — i jest
  bezpieczny, mimo że każde z odbić z osobna nie jest. Warto to w pracy wyjaśnić,
  bo wygląda paradoksalnie, a wyjaśnienie jest eleganckie: karty są symetryczne względem
  obrotu, ale nie względem odbicia.
- **Werdykt:** wyłączyć, zarówno dla klasyfikatora, jak i dla detektora. W konfiguracji
  YOLO oznacza to ustawienie `fliplr = 0.0` i `flipud = 0.0` — **domyślnie Ultralytics
  ma `fliplr = 0.5`, więc trzeba to jawnie nadpisać.** To pułapka, która po cichu psuje
  wyniki i jest dobrym materiałem na akapit w pracy.

#### Obrót o dowolny kąt — **TAK, ale zakres zależy od gałęzi eksperymentu**

- **Gałąź z korekcją perspektywy:** klasyfikator dostaje wycinki w kanonicznej orientacji
  pionowej. Duże obroty byłyby poza rozkładem danych. Sensowny zakres to **±10–15°**,
  symulujący niedokładność prostowania.
- **Gałąź bez korekcji perspektywy (E5):** karta może być pod dowolnym kątem, więc potrzebny
  jest **pełny zakres 0–360°**.
- **Ryzyko:** obrót wprowadza puste narożniki. Wypełniać kolorem tła lub odbiciem
  krawędzi — nigdy czernią, która tworzy sztuczny wysoki kontrast.

#### Transformacja perspektywiczna — **TAK, istotna**

- **Sens:** bezpośrednio symuluje obserwację karty pod kątem — główne źródło zmienności
  w tym zadaniu.
- **Parametry:** umiarkowane zniekształcenie, odpowiadające odchyleniu do ~45° od pionu.
  Zbyt silne skróty czynią indeks nieczytelnym i wprowadzają szum etykietowy.
- **Uwaga:** w gałęzi z korekcją perspektywy ta augmentacja ma mniejszy sens dla
  klasyfikatora (wejście jest już wyprostowane), ale pozostaje kluczowa dla **detektora**.

#### Skalowanie i przycinanie — **TAK**

- **Sens:** karty pojawiają się w różnej odległości od kamery.
- **Parametry:** skala 0,8–1,2; przycięcie losowe z zachowaniem całego indeksu narożnego.
- **Ryzyko istotne:** agresywne przycinanie może usunąć indeks, zostawiając obraz,
  z którego karta jest nierozpoznawalna, ale wciąż opatrzony etykietą. To wprowadza szum
  do zbioru treningowego. Przycięcie musi być ograniczone albo świadomie kontrolowane.

#### Jasność i kontrast — **TAK**

- **Sens:** bezpośrednio symuluje różne oświetlenie — jeden z kluczowych czynników (E10a).
- **Parametry:** jasność ±25%, kontrast ±25%.
- **Ryzyko:** przy skrajnych wartościach czerwień może stać się nieodróżnialna od czerni —
  co niszczy cechę rozpoznawania koloru. Zakres należy dobrać ostrożnie i zweryfikować
  wizualnie na próbce.

#### Zmiana barwy (hue) — **NIE (lub bardzo ograniczona)**

- **To najbardziej niebezpieczna augmentacja w tym projekcie.** Rozróżnienie ♥/♦ od ♠/♣
  opiera się na czerwieni. Przesunięcie odcienia o kilkanaście stopni może zamienić
  czerwone kiery w wizualnie brązowe lub fioletowe, a przy większym przesunięciu — uczynić
  je nieodróżnialnymi od czerni.
- **Werdykt:** wyłączyć albo ograniczyć do ±0,02 w skali znormalizowanej (praktycznie
  nieodczuwalne). W YOLO odpowiada to parametrowi `hsv_h`, którego **wartość domyślną
  trzeba obniżyć**.
- **Nasycenie (`hsv_s`) i jasność (`hsv_v`)** można pozostawić w umiarkowanym zakresie —
  symulują różne balanse bieli bez zmiany samego odcienia.

#### Konwersja do skali szarości — **NIE, poza jednym wyjątkiem**

- Usuwa informację o kolorze, czyli połowę sygnału potrzebnego do klasyfikacji koloru karty.
- **Wyjątek:** jako **osobny eksperyment diagnostyczny** — wytrenowanie modelu wyłącznie
  na obrazach w skali szarości odpowiada na pytanie, w jakim stopniu model opiera się na
  barwie, a w jakim na kształcie symbolu. Wynik jest ciekawy analitycznie i dobrze pasuje
  do rozdziału o analizie wyników. Nie jest to jednak augmentacja produkcyjna.

#### Rozmycie (blur) — **TAK, umiarkowanie**

- **Sens:** kamera internetowa produkuje rozmycie ruchu i błędy autofokusa. To realne
  zjawisko w scenariuszu docelowym.
- **Parametry:** rozmycie Gaussa o małym promieniu, prawdopodobieństwo ~0,2.
- **Ryzyko:** silne rozmycie czyni indeks nieczytelnym — znów szum etykietowy.

#### Szum — **TAK, umiarkowanie**

- **Sens:** szum sensora przy słabym oświetleniu, wyraźnie widoczny w tanich kamerach.
- **Parametry:** szum gaussowski o małej wariancji, prawdopodobieństwo ~0,2.

#### Symulacja przesłonięcia (cutout / random erasing) — **TAK, ostrożnie**

- **Sens:** karty zasłaniają się nawzajem, palce zasłaniają fragmenty, występują odblaski.
- **Parametry:** małe prostokąty, łącznie do ~10% powierzchni.
- **Ryzyko poważne:** jeśli wymazany fragment trafi w indeks narożny, obraz staje się
  nierozpoznawalny przy zachowanej etykiecie. **Rekomendacja:** ograniczyć obszar
  wymazywania do środkowej części karty, z wyłączeniem narożników — to prosta modyfikacja,
  a znacząco redukuje szum etykietowy.

#### Mixup / CutMix — **NIE**

- Mieszanie dwóch kart tworzy obraz, który nie odpowiada żadnej rzeczywistej karcie,
  z etykietą mieszaną. Przy klasach różniących się drobnymi detalami to raczej zaszumia
  zadanie niż je reguluje. Nie ma uzasadnienia przy zbiorze tej wielkości.

#### Mozaika (YOLO `mosaic`) — **TAK, ale tylko dla detektora**

- Sklejanie czterech obrazów w jeden zwiększa liczbę obiektów na obraz i uczy detektor
  radzić sobie z obiektami przy krawędziach. Dla detekcji jednoklasowej to sensowne.
- Standardową praktyką jest wyłączanie mozaiki w ostatnich epokach treningu
  (`close_mosaic`), żeby model zakończył uczenie na obrazach o naturalnym rozkładzie.

### 6.3 Podsumowanie polityki augmentacji

| Transformacja | Klasyfikator | Detektor | Uzasadnienie skrótowo |
|---|---|---|---|
| Obrót 180° | **TAK** (p=0,5) | TAK | karty symetryczne względem obrotu |
| Odbicie poziome | **NIE** | **NIE** | lustrzane cyfry i litery nie istnieją |
| Odbicie pionowe | **NIE** | **NIE** | jw. |
| Obrót dowolny | ±15° (z korekcją) / 0–360° (bez) | ±15° | zależne od gałęzi eksperymentu |
| Perspektywa | umiarkowana | **TAK** | główne źródło zmienności |
| Skala / przycięcie | TAK (0,8–1,2) | TAK | różna odległość od kamery |
| Jasność / kontrast | TAK (±25%) | TAK | różne oświetlenie |
| Hue | **NIE / ±0,02** | **NIE / ±0,02** | niszczy rozróżnienie czerwony–czarny |
| Nasycenie | TAK (umiarkowanie) | TAK | balans bieli |
| Skala szarości | **NIE** (poza diagnostyką) | NIE | usuwa cechę krytyczną |
| Rozmycie | TAK (p=0,2) | TAK | rozmycie ruchu z webcama |
| Szum | TAK (p=0,2) | TAK | szum sensora |
| Cutout | TAK (bez narożników) | NIE | zasłonięcia i odblaski |
| Mixup / CutMix | **NIE** | **NIE** | tworzy obrazy nieistniejące |
| Mozaika | nie dotyczy | TAK | więcej obiektów na obraz |

Konfiguracja augmentacji jest zapisywana w `configs/augmentation/*.yaml` i stanowi zmienną
niezależną eksperymentu **E4**.

---

## 7. Korekcja perspektywy

### 7.1 Na czym polega

Karta jest obiektem **płaskim**, więc jej obraz w kamerze jest związany z rzeczywistym
kształtem przekształceniem rzutowym (homografią). Znając położenie czterech narożników na
obrazie i wiedząc, że w rzeczywistości tworzą prostokąt o proporcji około 1 : 1,4
(63×88 mm), można wyznaczyć macierz homografii i **odwrócić** zniekształcenie, otrzymując
obraz karty „widzianej na wprost".

Algorytm:

1. **Wycinek** z bounding boxa detektora, z niewielkim marginesem.
2. **Segmentacja karty od tła** — progowanie adaptacyjne lub Otsu na obrazie w skali
   szarości, ewentualnie z operacjami morfologicznymi domykającymi kontur.
3. **Wykrycie konturów** (`cv2.findContours`), wybór konturu o największym polu.
4. **Aproksymacja wielokątem** (`cv2.approxPolyDP`) z tolerancją proporcjonalną do obwodu.
   Akceptujemy wynik tylko wtedy, gdy otrzymamy **dokładnie 4 wierzchołki** i kontur jest
   wypukły.
5. **Uporządkowanie narożników** w ustalonej kolejności (lewy górny, prawy górny, prawy
   dolny, lewy dolny) — na podstawie sum i różnic współrzędnych.
6. **Wyznaczenie orientacji:** dłuższy bok musi stać się pionowy, żeby wynik był w stałej
   proporcji. Zostaje niejednoznaczność obrotu o 180°, ale — jak wykazano w sekcji 6.2 —
   jest ona nieszkodliwa, bo karta jest względem takiego obrotu symetryczna.
7. **Transformacja** (`cv2.getPerspectiveTransform` + `cv2.warpPerspective`) do stałego
   rozmiaru wyjściowego, np. 160×224 px (proporcja ≈ 1 : 1,4).

### 7.2 Ścieżka awaryjna

Wykrycie czworokąta zawodzi, gdy karta jest częściowo zasłonięta, leży na tle o zbliżonej
jasności albo gdy odblask przerywa krawędź. Moduł **musi** mieć zdefiniowane zachowanie
w takim przypadku:

```
jeśli znaleziono poprawny czworokąt  → prostowanie perspektywiczne
w przeciwnym razie                   → zwykły crop z bboxa, przeskalowany do tego samego rozmiaru
                                       + oznaczenie wyniku flagą rectified = False
```

Flaga `rectified` jest zapisywana wraz z wynikiem. Pozwala to później sprawdzić,
**jak często prostowanie się udaje** i **czy nieudane przypadki mają gorszą dokładność
klasyfikacji** — to samodzielny, ciekawy wynik, który wchodzi do analizy E5.

### 7.3 Czy to naprawdę poprawi wyniki?

Uczciwa odpowiedź brzmi: **nie wiadomo z góry, i właśnie dlatego jest to eksperyment.**

Argumenty za poprawą:
- klasyfikator dostaje wejście o znormalizowanej geometrii, więc nie musi zużywać pojemności
  na uczenie się niezmienniczości względem perspektywy,
- indeks narożny zawsze znajduje się w tym samym miejscu obrazu,
- efektywnie zwiększa się rozdzielczość istotnego fragmentu (skrócony perspektywicznie
  fragment zostaje rozciągnięty).

Argumenty przeciw:
- sieci konwolucyjne z odpowiednią augmentacją potrafią same nauczyć się tolerancji na
  umiarkowane zniekształcenia,
- prostowanie wprowadza **własne błędy**: nieprecyzyjne narożniki dają zniekształcony obraz,
  a interpolacja przy silnym rozciągnięciu degraduje jakość,
- gdy prostowanie zawodzi (zasłonięcia), pipeline staje się niejednorodny,
- dodatkowy koszt obliczeniowy na każdą kartę w każdej klatce.

Eksperyment **E5** rozstrzyga to pomiarem. Warto zauważyć, że **negatywny wynik jest równie
wartościowy** — stwierdzenie „korekcja perspektywy nie poprawiła dokładności, a kosztowała
X ms na kartę, dlatego w wersji końcowej jej nie zastosowano" jest w pełni poprawnym
wnioskiem inżynierskim, poprzedzonym rzetelnym pomiarem.

---

## 8. Modele klasyfikacyjne

### 8.1 Dekompozycja problemu — 52 klasy, czy figura i kolor osobno?

Koncepcja wyjściowa wskazuje dwa podejścia. Warto rozważyć **trzy**.

#### Podejście A — jeden model, 52 klasy

Jedna sieć, jedna warstwa wyjściowa o 52 neuronach, softmax po wszystkich klasach.

#### Podejście B — dwa niezależne modele: 13 figur + 4 kolory

Dwie osobne sieci, dwa treningi, wynik łączony jako iloczyn kartezjański.

**Istotna własność matematyczna:** przy niezależności błędów dokładność karty wynosi
w przybliżeniu iloczyn dokładności obu modeli. Przy 98% na figurze i 99% na kolorze daje to
około 97% na karcie — **czyli mniej niż każdy z modeli z osobna**. To niepozorna, ale ważna
obserwacja i dobry materiał na akapit w pracy.

#### Podejście C — jeden wspólny backbone, dwie głowy klasyfikacyjne

```
        obraz karty
             │
      wspólny backbone (ekstrakcja cech)
             │
      ┌──────┴──────┐
      ▼             ▼
głowa figury    głowa koloru
  (13 klas)       (4 klasy)
```

Jedno przejście przez sieć, dwie funkcje straty sumowane (ewentualnie z wagami).
Podejście nieobecne w koncepcji wyjściowej, a **prawdopodobnie najlepsze z trzech**.

#### Porównanie

| Kryterium | A (52 klasy) | B (dwa modele) | C (multi-head) |
|---|---|---|---|
| Liczba modeli do wytrenowania | 1 | 2 | 1 |
| Liczba przejść przy inferencji | 1 | **2** | 1 |
| Trudność implementacji | najniższa | średnia | średnia |
| Efektywna liczba przykładów na klasę | najmniejsza (÷52) | największa (÷13, ÷4) | największa |
| Propagacja błędów | brak | **iloczyn dokładności** | brak (jeden przebieg) |
| Wykorzystanie struktury problemu | **brak** — traktuje ♠A i ♠2 jako klasy niezwiązane | pełne | pełne |
| Interpretowalność błędów | trudniejsza | łatwa (osobno figura, osobno kolor) | **łatwa** |
| Wymóg pamięci | najmniejszy | podwójny | najmniejszy |
| Wartość dla pracy | punkt odniesienia | pokazuje pułapkę iloczynu | pokazuje właściwe rozwiązanie |

#### `[DECYZJA]` Rekomendacja

**Zaimplementować wszystkie trzy i porównać w eksperymencie E3**, przyjmując A jako
domyślne w MVP (najprostsze, najszybciej działające), a C jako kandydata na wersję końcową.

Uzasadnienie: różnice między nimi są niewielkie implementacyjnie — to ta sama sieć
z inną warstwą wyjściową i inną funkcją straty — a eksperyment E3 jest jednym
z ciekawszych w całej pracy. Hipoteza wyjściowa jest taka, że C wypadnie najlepiej, bo
łączy zalety obu podejść, ale **nie należy tego zakładać z góry**; przy zbiorze z równą
liczbą przykładów na każdą z 52 klas przewaga dekompozycji może okazać się pozorna.

Klucz techniczny: **wspólne mapowanie klas**. Jedno źródło prawdy
(`src/cardvision/classification/labels.py`) definiuje 52 karty i przekłada je w obie strony
na parę (figura, kolor). Wszystkie trzy podejścia korzystają z tego samego mapowania,
co gwarantuje porównywalność wyników i jest oczywistym kandydatem na test jednostkowy.

### 8.2 Architektury do porównania

| Model | Typ | Rząd wielkości parametrów | Rola w projekcie |
|---|---|---|---|
| **SimpleCNN** | własna sieć, 3–4 bloki konwolucyjne | setki tysięcy | baseline — „ile da się osiągnąć bez transfer learningu" |
| **ResNet18** | pretrenowany na ImageNet | ~11 mln | główny kandydat produkcyjny |
| **MobileNetV3-Small** | pretrenowany, zoptymalizowany pod szybkość | ~2,5 mln | kandydat do trybu real-time |
| EfficientNet-B0 | pretrenowany | ~5 mln | opcjonalnie, jeśli zostanie czas |

`[DO WERYFIKACJI]` dokładne liczby parametrów — odczytać programowo z modelu
(`sum(p.numel() for p in model.parameters())`) i wstawić do pracy zmierzone wartości,
nie przybliżenia z tej tabeli.

#### SimpleCNN — propozycja architektury

```
Wejście 3 × 224 × 160
  ├─ Conv(3→32, 3×3) + BatchNorm + ReLU + MaxPool(2)
  ├─ Conv(32→64, 3×3) + BatchNorm + ReLU + MaxPool(2)
  ├─ Conv(64→128, 3×3) + BatchNorm + ReLU + MaxPool(2)
  ├─ Conv(128→256, 3×3) + BatchNorm + ReLU + AdaptiveAvgPool(1)
  ├─ Dropout(0,4)
  └─ Linear(256 → liczba klas)
```

Rola: pokazać, ile daje sama architektura konwolucyjna bez wiedzy przeniesionej z ImageNetu.
Dodatkowa zaleta: jest to jedyny model, którego **każdą warstwę można w pracy opisać
i uzasadnić** — dobry materiał na rozdział o sieciach CNN.

#### ResNet18

Uzasadnienie wyboru zamiast głębszych wariantów: przy 52 klasach i zbiorze rzędu
kilku–kilkunastu tysięcy obrazów ResNet50 lub głębszy niemal na pewno się przeuczy,
a przy tym spowolni inferencję. ResNet18 to najczęściej stosowany punkt równowagi
i naturalny wybór referencyjny.

#### MobileNetV3-Small

Uzasadnienie: architektura zaprojektowana pod ograniczone zasoby (konwolucje separowalne,
bloki squeeze-and-excitation). Jeżeli E1 wykaże zbliżoną dokładność do ResNet18 przy
istotnie krótszym czasie inferencji, to **ten model trafi do wersji produkcyjnej** — i będzie
to decyzja uzasadniona pomiarem, dokładnie tak, jak wymaga tego praca inżynierska.

### 8.3 Transfer learning

Uzasadnienie zastosowania: wczesne warstwy sieci pretrenowanych na ImageNecie wykrywają
krawędzie, narożniki i proste kształty — cechy w pełni użyteczne przy rozpoznawaniu
symboli i cyfr na kartach. Przy zbiorze liczącym tysiące, a nie miliony obrazów, uczenie
tych warstw od zera jest marnowaniem danych.

**Zaplanowany protokół (dwuetapowy):**

| Etap | Co robimy | Współczynnik uczenia | Liczba epok |
|---|---|---|---|
| 1. Warm-up | zamrożenie całego backbone'u, uczenie wyłącznie nowej warstwy klasyfikującej | wyższy (rząd 1e-3) | kilka |
| 2. Fine-tuning | odmrożenie ostatnich bloków (lub całości), uczenie z niskim LR | niższy (rząd 1e-4) | kilkanaście–kilkadziesiąt |

Warianty do porównania w **E2**:
- (a) trening od zera (losowa inicjalizacja),
- (b) backbone zamrożony, uczona tylko głowa,
- (c) pełny fine-tuning z pretrenowanych wag.

**Uwaga techniczna:** modele pretrenowane oczekują normalizacji statystykami ImageNetu.
Pominięcie tego kroku jest typowym błędem powodującym słabe wyniki fine-tuningu
i warto o nim wspomnieć w rozdziale implementacyjnym.

### 8.4 Parametry treningu — punkt wyjścia

Poniższe wartości to **wartości startowe do strojenia na zbiorze walidacyjnym**, a nie
wyniki. Ostateczne parametry każdego eksperymentu zapisywane są w jego konfiguracji.

| Parametr | Wartość startowa | Uwagi |
|---|---|---|
| Rozdzielczość wejścia | 224 × 160 (H × W) | proporcja karty; badane w E8 |
| Rozmiar batcha | 64 | do dopasowania pod 8 GB VRAM |
| Optymalizator | AdamW | dobra wartość domyślna |
| LR (od zera) | 1e-3 | |
| LR (fine-tuning) | 1e-4 | |
| Harmonogram LR | cosine annealing | |
| Weight decay | 1e-4 | |
| Funkcja straty | cross-entropy z label smoothing 0,1 | smoothing ogranicza nadmierną pewność — istotne dla sekcji 9 |
| Epoki | 30–50 z early stopping | |
| Kryterium early stopping | strata walidacyjna, cierpliwość 7 epok | |
| Mieszana precyzja (AMP) | włączona | przyspiesza trening na RTX 3060 Ti |
| Ziarno losowe | 42, zapisywane w konfiguracji | |

---

## 9. Confidence, kalibracja i obsługa błędów

### 9.1 Problem: softmax to nie prawdopodobieństwo

Wyjście warstwy softmax bywa nazywane „prawdopodobieństwem", ale sieci neuronowe są
notorycznie **przesadnie pewne siebie**: model potrafi zwrócić 0,99 dla obrazu, który
w ogóle nie przedstawia karty. Użycie surowego softmaxu jako miary pewności bez weryfikacji
byłoby błędem metodologicznym.

Dlatego system realizuje trzy kroki: **pomiar kalibracji → kalibracja → wyznaczenie progu**.

### 9.2 Krok 1 — pomiar kalibracji

- **Wykres niezawodności** (*reliability diagram*): obserwacje grupowane w koszyki według
  deklarowanej pewności; dla każdego koszyka nanoszona jest rzeczywista dokładność.
  Model idealnie skalibrowany leży na przekątnej.
- **Expected Calibration Error (ECE)**: średnia ważona różnic między deklarowaną pewnością
  a rzeczywistą dokładnością w koszykach. Jedna liczba `[DO ZMIERZENIA]`, którą można
  porównywać między modelami.

Oba wykresy trafiają wprost do pracy.

### 9.3 Krok 2 — kalibracja metodą temperature scaling

Najprostsza skuteczna metoda: przed softmaxem logity dzielone są przez skalar `T`,
dobierany przez minimalizację straty **na zbiorze walidacyjnym** (nigdy testowym).

Zalety: jeden parametr, nie zmienia kolejności klas (więc **dokładność pozostaje
niezmieniona** — poprawia się wyłącznie jakość samej miary pewności), implementacja
to kilkadziesiąt linii kodu.

### 9.4 Krok 3 — wyznaczenie progu na podstawie danych (E9)

Próg **nie jest** wybierany arbitralnie. Procedura:

1. Na zbiorze walidacyjnym obliczyć skalibrowaną pewność dla każdej próbki.
2. Dla progów z zakresu 0,50–0,99 (co 0,01) wyznaczyć:
   - **pokrycie** (*coverage*) — odsetek próbek, dla których model udziela odpowiedzi,
   - **dokładność warunkową** (*selective accuracy*) — dokładność wyłącznie na próbkach
     zaakceptowanych.
3. Wykreślić **krzywą ryzyko–pokrycie**.
4. Wybrać próg realizujący przyjęty wcześniej cel jakościowy, np.:
   *„najniższy próg, przy którym dokładność na zaakceptowanych wynosi ≥ 99%"* —
   maksymalizujemy pokrycie przy zadanym poziomie ryzyka.
5. Zweryfikować wybrany próg **jednorazowo** na zbiorze testowym.

Wynikiem jest konkretna liczba `[DO ZMIERZENIA]` z pełnym uzasadnieniem procedury —
znacznie mocniejszy materiał niż „przyjęto próg 0,8".

**Do rozważenia dodatkowo:** margines między najwyższym a drugim co do wielkości
prawdopodobieństwem bywa lepszym sygnałem niepewności niż samo maksimum. Porównanie obu
miar to tani dodatek do E9.

### 9.5 Prezentacja użytkownikowi

```
┌─────────────────────────────┐   ┌─────────────────────────────┐
│  7 ♥                        │   │  ?  NIEPEWNA KLASYFIKACJA   │
│  Pewność: 98,3%             │   │  Najlepsza hipoteza: 6 ♦    │
│  ● PEWNA                    │   │  Pewność: 53,1% (< próg)    │
└─────────────────────────────┘   └─────────────────────────────┘
```

Zasada nadrzędna: **system woli odmówić odpowiedzi, niż podać błędną.** W zastosowaniach
takich jak sędziowanie rozdania błędne rozpoznanie jest kosztowniejsze niż brak rozpoznania.

### 9.6 Katalog sytuacji błędnych i reakcji systemu

| Sytuacja | Gdzie wykrywana | Reakcja systemu |
|---|---|---|
| Brak karty w kadrze | detektor nie zwraca żadnego obiektu | komunikat „Nie wykryto kart" |
| Karta częściowo poza kadrem | bbox styka się z krawędzią obrazu | ostrzeżenie „karta poza kadrem", klasyfikacja z obniżonym zaufaniem |
| Dwie nachodzące karty | wysokie IoU między bboxami | NMS; jeżeli po NMS pozostaje nakładanie > próg — ostrzeżenie o możliwym przesłonięciu |
| Karta odwrócona rewersem | klasyfikator zwraca niską pewność, obraz bez indeksu | „nierozpoznana / rewers" |
| Bardzo słabe oświetlenie | średnia jasność wycinka poniżej progu | ostrzeżenie „zbyt słabe oświetlenie" + sugestia poprawy |
| Rozmazany obraz | wariancja laplasjanu poniżej progu | ostrzeżenie „obraz nieostry", w trybie kamery — pominięcie klatki |
| Obiekt niebędący kartą | pewność poniżej progu lub odrzucenie przez filtr proporcji bboxa | odrzucenie |
| Karty wizualnie podobne | mały margines między dwiema najlepszymi klasami | „niepewna klasyfikacja" z podaniem obu hipotez |
| Pewność poniżej progu | moduł decyzyjny | „Nie udało się jednoznacznie rozpoznać karty" |
| Brak dostępu do kamery | warstwa akwizycji | czytelny komunikat i degradacja do trybu plikowego |
| Brak pliku wag modelu | inicjalizacja | czytelny komunikat z instrukcją, gdzie umieścić plik |

Dwie proste heurystyki warte wyróżnienia, bo są tanie i skuteczne:

- **Filtr geometryczny bboxa:** proporcja boków karty jest znana (~1 : 1,4 z tolerancją na
  perspektywę). Obiekty o skrajnie odbiegających proporcjach można odrzucić przed
  klasyfikacją — eliminuje to część fałszywych detekcji przy zerowym koszcie.
- **Detektor rozmycia oparty na wariancji laplasjanu:** jedna linia w OpenCV, a pozwala
  automatycznie pomijać nieostre klatki w trybie kamery.

---

## 10. Metryki

### 10.1 Detekcja

| Metryka | Co mierzy | Dlaczego w tym projekcie istotna |
|---|---|---|
| **IoU** | pokrycie predykcji z prawdą | podstawa pozostałych metryk; próg 0,5 |
| **Precision** | jaki odsetek detekcji jest poprawny | fałszywe detekcje generują błędne karty w wyniku |
| **Recall** | jaki odsetek kart wykryto | **najważniejsza metryka detekcji w tym systemie** — nieznalezionej karty nie da się odzyskać na dalszych etapach |
| **mAP@50** | jakość detekcji przy łagodnym progu IoU | metryka główna, porównywalna z literaturą |
| **mAP@50:95** | jakość przy rosnących wymaganiach co do dokładności lokalizacji | raportowana dla porządku, ale **o ograniczonej użyteczności w tym zadaniu** — patrz niżej |
| **FPS** | przepustowość | wymóg czasu rzeczywistego (C7) |

**Które metryki są tu mniej istotne i dlaczego — to należy w pracy wyjaśnić:**

`mAP@50:95` premiuje bardzo dokładne dopasowanie bounding boxa. W naszym pipelinie
bounding box służy wyłącznie do **wycięcia karty przed klasyfikacją**, i to z zapasem
marginesu. Box o IoU 0,6 wycina kartę równie użytecznie jak box o IoU 0,9. Metryka jest
więc raportowana dla kompletności, ale wnioski projektowe wyciągamy z `mAP@50` i przede
wszystkim z **metryki end-to-end** (punkt 10.3).

### 10.2 Klasyfikacja

| Metryka | Co mierzy | Uwagi |
|---|---|---|
| **Accuracy** | odsetek poprawnych klasyfikacji | wystarczająca **tylko przy zrównoważonym zbiorze**; przy nierównowadze myląca |
| **Precision / Recall / F1 per klasa** | jakość dla każdej z 52 kart osobno | ujawnia klasy problematyczne — podstawa analizy błędów |
| **Macro-F1** | średnia F1 po klasach | traktuje wszystkie karty jednakowo niezależnie od liczności |
| **Top-3 accuracy** | czy poprawna klasa jest w trójce najlepszych | diagnostyczna: pozwala odróżnić „model nie ma pojęcia" od „model waha się między podobnymi kartami" |
| **Macierz pomyłek** | wzorce mylenia klas | najważniejsze narzędzie analityczne — sekcja 10.4 |
| **ECE** | jakość kalibracji pewności | sekcja 9.2 |
| **Czas inferencji** | ms na kartę | do budżetu czasowego real-time |

**Metryki dodatkowe wynikające ze struktury problemu** — warte wyliczenia, bo dają wgląd
niedostępny z samej dokładności 52-klasowej:

- **dokładność figury** (ignorując kolor) — czy model myli się co do wartości,
- **dokładność koloru** (ignorując figurę) — czy model myli się co do koloru,
- **dokładność barwy karty** (czerwona kontra czarna) — czy błędy koloru są „w obrębie
  barwy" (♥↔♦, ♠↔♣), czy „poprzez barwę" (♥↔♠).

Ten ostatni rozkład jest szczególnie wymowny: przewaga pomyłek w obrębie tej samej barwy
oznaczałaby, że model dobrze radzi sobie z kolorem, a myli kształty symboli — zupełnie inny
wniosek niż odwrotny rozkład, i inne zalecenia naprawcze.

### 10.3 Metryka end-to-end (własna)

Ani metryki detekcji, ani klasyfikacji osobno nie mówią, jak dobry jest **cały system**.
Dlatego definiujemy metrykę główną:

> **Dokładność end-to-end na poziomie karty:** odsetek kart obecnych na zdjęciu, które
> system (a) wykrył z IoU ≥ 0,5 wobec prawdy, (b) poprawnie sklasyfikował, oraz (c) zwrócił
> z pewnością powyżej progu.

Uzupełniająco raportujemy:
- **odsetek pominiętych kart** (niewykrytych),
- **odsetek fałszywych kart** (detekcji niebędących kartami),
- **odsetek odrzuceń** (wykrytych, ale poniżej progu pewności),
- **całkowitą latencję** na klatkę wraz z rozbiciem na etapy: preprocessing / detekcja /
  prostowanie / klasyfikacja / postprocessing.

**To jest liczba, którą podaje się w podsumowaniu pracy** — nie sama dokładność
klasyfikatora na wyciętych kartach, bo ta jest zawsze optymistyczna.

### 10.4 Macierz pomyłek — plan analizy

Generowana automatycznie (`src/cardvision/evaluation/confusion.py`) w trzech wariantach:

1. **52 × 52** — pełna, jako załącznik do pracy (czytelna tylko jako mapa cieplna),
2. **13 × 13** — dla samej figury, czytelna w tekście,
3. **4 × 4** — dla samego koloru, czytelna w tekście.

Analiza ma odpowiedzieć na pytania:
- które pary kart mylą się najczęściej i czy da się to wyjaśnić wyglądem
  (hipotezy do sprawdzenia: `6`↔`9`, `♠`↔`♣`, `♥`↔`♦`, `Q`↔`K`),
- czy pomyłki koncentrują się na figurach, czy na kolorach,
- czy istnieją klasy systematycznie słabe (i czy pokrywa się to z ich licznością w zbiorze),
- czy błędy korelują z warunkami akwizycji (oświetlenie, kąt) — łączone z metadanymi sesji.

Wszystkie wykresy generuje jeden skrypt (`scripts/make_report.py`) do katalogu
`experiments/<ID>/plots/`, w formacie nadającym się do wklejenia do pracy (wektorowy PDF
lub PNG w wysokiej rozdzielczości, czcionka dopasowana rozmiarem do tekstu pracy).

---

## 11. Eksperymenty

### 11.1 Zasady wspólne dla wszystkich eksperymentów

1. **Jedna zmienna niezależna na eksperyment.** Wszystko inne pozostaje identyczne —
   inaczej wynik jest niemożliwy do zinterpretowania.
2. **Wspólny zbiór testowy.** Wszystkie warianty w obrębie jednego eksperymentu oceniane
   są na dokładnie tym samym zbiorze.
3. **Ustalone ziarno losowe**, zapisywane w konfiguracji przebiegu.
4. **Powtarzalność:** eksperymenty, w których różnice mogą być małe (E1, E3, E4), wykonać
   **3 razy z różnymi ziarnami** i raportować średnią wraz z odchyleniem standardowym.
   Bez tego nie da się odróżnić rzeczywistej przewagi od szumu — a to najczęstsza słabość
   części eksperymentalnej prac dyplomowych.
5. **Wszystko zapisywane:** konfiguracja, metryki, wykresy, wersje bibliotek, czas
   wykonania, identyfikator commita.
6. **Wyniki negatywne są pełnoprawnymi wynikami** i trafiają do pracy tak samo jak pozytywne.

### 11.2 Katalog eksperymentów

---

#### E1 — Porównanie architektur klasyfikatora

| | |
|---|---|
| **Hipoteza** | Modele pretrenowane (ResNet18, MobileNetV3) osiągną wyższą dokładność niż własna sieć trenowana od zera, przy tym samym zbiorze i budżecie epok. |
| **Zmienna niezależna** | architektura klasyfikatora: SimpleCNN / ResNet18 / MobileNetV3-Small |
| **Zmienne zależne** | accuracy, macro-F1, czas inferencji na kartę, czas treningu, liczba parametrów |
| **Stałe** | zbiór, podział, augmentacja, rozdzielczość, liczba epok, optymalizator |
| **Wykonanie** | 3 modele × 3 ziarna = 9 przebiegów |
| **Prezentacja** | tabela ze średnią ± odchylenie; wykres punktowy dokładność kontra czas inferencji |
| **Oczekiwany wniosek** | wskazanie modelu o najlepszym stosunku jakości do kosztu; podstawa wyboru modelu produkcyjnego |

---

#### E2 — Transfer learning kontra uczenie od zera

| | |
|---|---|
| **Hipoteza** | Inicjalizacja wagami z ImageNetu znacząco przyspieszy zbieżność i podniesie końcową dokładność, szczególnie przy mniejszym zbiorze treningowym. |
| **Zmienna niezależna** | strategia inicjalizacji: (a) od zera, (b) backbone zamrożony, (c) pełny fine-tuning |
| **Zmienne zależne** | accuracy, liczba epok do osiągnięcia zbieżności, przebieg krzywych uczenia |
| **Wykonanie** | ResNet18 w trzech wariantach; dodatkowo powtórzenie na 25% zbioru treningowego, żeby zmierzyć, jak przewaga transfer learningu zależy od ilości danych |
| **Prezentacja** | krzywe uczenia (strata i dokładność w funkcji epoki) na wspólnym wykresie; tabela wyników końcowych |
| **Oczekiwany wniosek** | ilościowe uzasadnienie użycia transfer learningu; obserwacja zależności przewagi od rozmiaru zbioru |

---

#### E3 — Dekompozycja problemu: 52 klasy kontra figura + kolor

| | |
|---|---|
| **Hipoteza** | Podejście multi-head (wspólny backbone, dwie głowy) przewyższy zarówno klasyfikator 52-klasowy, jak i dwa niezależne modele, ponieważ wykorzystuje strukturę problemu bez podwajania kosztu inferencji. |
| **Zmienna niezależna** | struktura wyjścia: A (52 klasy) / B (dwa niezależne modele 13 + 4) / C (multi-head 13 + 4) |
| **Zmienne zależne** | dokładność karty, dokładność figury, dokładność koloru, czas inferencji, liczba parametrów |
| **Stałe** | backbone (ResNet18), zbiór, augmentacja, protokół treningu |
| **Wykonanie** | 3 warianty × 3 ziarna |
| **Prezentacja** | tabela porównawcza; osobne macierze pomyłek dla figury i koloru |
| **Oczekiwany wniosek** | rozstrzygnięcie, czy dekompozycja pomaga; empiryczna weryfikacja przewidywania, że w wariancie B dokładność karty jest w przybliżeniu iloczynem dokładności składowych |

---

#### E4 — Wpływ augmentacji danych

| | |
|---|---|
| **Hipoteza** | Augmentacja geometryczna da największy przyrost odporności; augmentacja fotometryczna pomoże głównie na własnym zbiorze testowym (inne oświetlenie niż w danych publicznych). |
| **Zmienna niezależna** | zestaw augmentacji: (0) brak, (1) tylko geometryczne, (2) tylko fotometryczne, (3) pełny zestaw z sekcji 6.3, (4) pełny zestaw **z błędnie włączonym odbiciem poziomym** |
| **Zmienne zależne** | accuracy na `test_public` **i osobno** na `test_own` |
| **Wykonanie** | 5 wariantów × 3 ziarna |
| **Prezentacja** | wykres słupkowy z dwiema seriami (test publiczny i własny) |
| **Oczekiwany wniosek** | ilościowe uzasadnienie przyjętej polityki augmentacji. **Wariant (4) jest tu celowo** — pozwala empirycznie potwierdzić rozumowanie z sekcji 6.2 o szkodliwości odbicia lustrzanego. Jeśli wariant (4) wypadnie gorzej od (3), jest to bezpośredni dowód eksperymentalny na poparcie decyzji projektowej. |

---

#### E5 — Wpływ korekcji perspektywy

| | |
|---|---|
| **Hipoteza** | Korekcja perspektywy poprawi dokładność klasyfikacji, a przyrost będzie rósł wraz z kątem obserwacji karty. |
| **Zmienna niezależna** | włączona / wyłączona korekcja perspektywy przed klasyfikacją |
| **Zmienne zależne** | dokładność klasyfikacji, dokładność w podziale na przedziały kąta, czas przetwarzania jednej karty, odsetek udanych prostowań |
| **Wykonanie** | ten sam model trenowany i testowany w obu wariantach; dodatkowo rozbicie wyników według przedziałów kąta (0–15°, 15–30°, 30–45°, > 45°) na podstawie metadanych sesji |
| **Prezentacja** | wykres dokładności w funkcji przedziału kąta, dwie serie; tabela kosztu czasowego |
| **Oczekiwany wniosek** | decyzja, czy korekcja trafia do wersji produkcyjnej. **Wynik negatywny jest w pełni akceptowalny** i wtedy równie wartościowy — pod warunkiem podania kosztu czasowego, który dzięki temu oszczędzamy |

---

#### E6 — Architektura jednoetapowa kontra dwuetapowa

| | |
|---|---|
| **Hipoteza** | Architektura jednoetapowa (YOLO z 52 klasami) będzie szybsza, ale mniej dokładna od dwuetapowej, ze względu na mniejszą efektywną liczbę przykładów na klasę i brak normalizacji geometrycznej wejścia. |
| **Zmienna niezależna** | architektura systemu: jednoetapowa / dwuetapowa |
| **Zmienne zależne** | dokładność end-to-end na poziomie karty (sekcja 10.3), FPS, całkowity czas treningu obu wariantów |
| **Wykonanie** | wytrenowanie YOLO z 52 klasami na tym samym zbiorze detekcyjnym i porównanie z pełnym pipelinem dwuetapowym na tym samym zbiorze testowym |
| **Prezentacja** | tabela; wykres dokładność end-to-end kontra FPS |
| **Oczekiwany wniosek** | **uzasadnienie kluczowej decyzji architektonicznej pracy pomiarem, a nie deklaracją.** To najważniejszy eksperyment z punktu widzenia rozdziału o projekcie systemu |

---

#### E7 — Detektor klasyczny kontra uczony

| | |
|---|---|
| **Hipoteza** | Detektor konturowy dorówna YOLO w warunkach kontrolowanych (jednolite tło, dobre oświetlenie, karty rozdzielone), ale wyraźnie ustąpi w warunkach trudnych (wzorzyste tło, słabe światło, karty nachodzące). |
| **Zmienna niezależna** | implementacja detektora: ContourDetector / YoloDetector |
| **Zmienne zależne** | precision, recall, mAP@50, FPS — **z rozbiciem na warunki akwizycji** |
| **Wykonanie** | oba detektory na tym samym zbiorze testowym, wyniki grupowane według metadanych sesji |
| **Prezentacja** | tabela z podziałem na warunki; przykładowe obrazy z zaznaczonymi błędami obu metod |
| **Oczekiwany wniosek** | **empiryczne uzasadnienie sensu stosowania uczenia maszynowego w tym zadaniu** — bezpośrednia odpowiedź na pytanie z sekcji 1.3. Doskonały materiał na rozdział wprowadzający |

---

#### E8 — Wpływ rozdzielczości wejściowej klasyfikatora

| | |
|---|---|
| **Hipoteza** | Istnieje próg rozdzielczości, poniżej którego indeks narożny przestaje być czytelny i dokładność gwałtownie spada; powyżej tego progu przyrosty są niewielkie, a koszt obliczeniowy rośnie kwadratowo. |
| **Zmienna niezależna** | rozdzielczość wejścia: 64×46, 96×69, 128×91, 160×114, 224×160, 320×229 |
| **Zmienne zależne** | accuracy, czas inferencji, zużycie pamięci |
| **Wykonanie** | ResNet18, 6 wariantów, jednakowy protokół treningu |
| **Prezentacja** | wykres dokładności i czasu inferencji w funkcji rozdzielczości (dwie osie Y) |
| **Oczekiwany wniosek** | wybór rozdzielczości produkcyjnej z jawnym kompromisem jakość–szybkość; powiązanie wyniku z analizą z sekcji 1.2a o rozmiarze indeksu narożnego |

---

#### E9 — Kalibracja i wyznaczenie progu pewności

| | |
|---|---|
| **Hipoteza** | Surowy softmax będzie przesadnie pewny (ECE istotnie > 0); temperature scaling poprawi kalibrację bez zmiany dokładności; istnieje próg dający dokładność warunkową ≥ 99% przy pokryciu powyżej 90%. |
| **Zmienna niezależna** | próg akceptacji (0,50–0,99) oraz obecność kalibracji |
| **Zmienne zależne** | pokrycie, dokładność warunkowa, ECE, liczba błędów zaakceptowanych |
| **Wykonanie** | procedura z sekcji 9.4; porównanie miary „maksimum softmax" z miarą „margines dwóch najlepszych" |
| **Prezentacja** | wykres niezawodności przed i po kalibracji; krzywa ryzyko–pokrycie z zaznaczonym wybranym progiem |
| **Oczekiwany wniosek** | **konkretna, uzasadniona wartość progu** `[DO ZMIERZENIA]` używana w wersji produkcyjnej |

---

#### E10 — Odporność systemu (rodzina trzech eksperymentów)

Wspólna zasada: system oceniany jest na **własnym zbiorze testowym**, a wyniki grupowane
według metadanych sesji z `data/metadata/sessions.csv`. Dlatego rejestrowanie warunków przy
zbieraniu danych (Faza 7) jest krytyczne — bez tego ta rodzina eksperymentów jest niewykonalna.

**E10a — Warunki oświetleniowe**
- Zmienna niezależna: światło dzienne / żarowe ciepłe / LED zimne / przyciemnione.
- Hipoteza: największy spadek dotknie **dokładności koloru**, a nie figury, ponieważ
  rozróżnienie czerwony–czarny zależy od barwy oświetlenia.
- Dodatkowo: sprawdzenie, czy wstępne przetwarzanie (CLAHE, korekta balansu bieli)
  redukuje ten spadek — to prosta interwencja o potencjalnie dużym efekcie.

**E10b — Kąt obserwacji**
- Zmienna niezależna: przedziały kąta 0–15°, 15–30°, 30–45°, powyżej 45°.
- Hipoteza: dokładność spada monotonicznie wraz z kątem; korekcja perspektywy (E5)
  spłaszcza ten spadek.
- Wynik: wyznaczenie **granicy stosowalności systemu** — wartościowa treść rozdziału
  o ograniczeniach.

**E10c — Liczba kart i przesłonięcia**
- Zmienna niezależna: 1 / 2 / 5 / 10 kart rozdzielonych oraz układy z częściowym
  nakładaniem (wachlarz, karty nachodzące).
- Zmienne zależne: recall detekcji, dokładność end-to-end, czas przetwarzania klatki.
- Hipoteza: recall detekcji spada przy nakładaniu, natomiast dokładność klasyfikacji
  wykrytych kart pozostaje wysoka; czas rośnie liniowo z liczbą kart.
- Wynik ma bezpośredni wpływ na budżet czasu rzeczywistego (C7).

---

#### E11 — Domain gap: dane publiczne kontra własne

| | |
|---|---|
| **Hipoteza** | Model wytrenowany wyłącznie na danych publicznych osiągnie istotnie niższą dokładność na własnym zbiorze testowym niż na publicznym; różnica ta jest miarą domain gap. Dostrojenie na niewielkiej próbce własnych danych zlikwiduje większość tej różnicy. |
| **Zmienna niezależna** | źródło danych treningowych: (a) tylko publiczne, (b) publiczne + dostrojenie na małej próbce własnych danych |
| **Zmienne zależne** | accuracy na `test_public` i na `test_own`, wielkość różnicy |
| **Wykonanie** | wariant (b) wymaga wydzielenia z własnych danych małej porcji treningowej **rozłącznej z `test_own` na poziomie sesji** |
| **Prezentacja** | wykres słupkowy dwóch zbiorów testowych dla obu wariantów; przykłady obrazów klasyfikowanych błędnie tylko na własnym zbiorze |
| **Oczekiwany wniosek** | **ilościowa ocena przenoszalności modeli uczonych na danych publicznych do rzeczywistego zastosowania.** Bezpośrednio odpowiada na główne ograniczenie przyjętej strategii danych i zamienia je w wynik naukowy |

---

#### E12 — Dane syntetyczne kontra rzeczywiste

| | |
|---|---|
| **Hipoteza** | Dodanie danych syntetycznych do zbioru treningowego detektora poprawi recall na scenach wielokartowych; trening wyłącznie na danych syntetycznych da wynik wyraźnie gorszy od treningu mieszanego. |
| **Zmienna niezależna** | skład zbioru treningowego detektora: (a) tylko rzeczywiste, (b) tylko syntetyczne, (c) mieszane |
| **Zmienne zależne** | mAP@50, recall, precision na rzeczywistym zbiorze testowym |
| **Wykonanie** | trzy treningi YOLO o zrównanej liczbie obrazów |
| **Prezentacja** | tabela; przykłady błędów charakterystycznych dla modelu uczonego wyłącznie na danych syntetycznych |
| **Oczekiwany wniosek** | ocena opłacalności generowania danych syntetycznych — praktyczna wskazówka inżynierska |

---

### 11.3 Priorytetyzacja eksperymentów

Przy nierównym budżecie czasu nie wszystkie eksperymenty muszą powstać. Kolejność
ważności dla pracy inżynierskiej:

| Priorytet | Eksperymenty | Uzasadnienie |
|---|---|---|
| **Krytyczne** (muszą być) | E1, E3, E4, E9 | podstawa rozdziału eksperymentalnego; wszystkie wykonalne bez własnego zbioru danych |
| **Bardzo ważne** | E2, E5, E11 | uzasadniają kluczowe decyzje projektowe |
| **Ważne** | E6, E7, E8 | uzasadniają architekturę i wybór technologii |
| **Uzupełniające** | E10a, E10b, E10c, E12 | rozdział o odporności i ograniczeniach; wymagają własnego zbioru |

Minimalny zestaw dający wartościową pracę: **E1, E2, E3, E4, E5, E9** — sześć eksperymentów
wykonalnych wyłącznie na danych publicznych, bez zależności od Fazy 7.

---

## 12. Aplikacja

### 12.1 Scenariusze użycia

| Scenariusz | Wejście | Wynik |
|---|---|---|
| Analiza pojedynczego zdjęcia | plik obrazu | obraz z ramkami, nazwami kart i pewnością |
| Analiza wsadowa | katalog obrazów | raport CSV/JSON dla wszystkich plików |
| Analiza wideo | plik wideo | wideo z nałożonymi wynikami + statystyki |
| Tryb kamery | strumień z webcama | podgląd w czasie rzeczywistym z nałożonymi wynikami |
| Tryb eksperymentu | konfiguracja YAML | metryki i wykresy w katalogu eksperymentu |

Pierwsze cztery obsługuje GUI, wszystkie pięć — CLI.

### 12.2 Układ interfejsu

```
┌──────────────────────────────────────────────────────────────────────┐
│  Rozpoznawanie kart do gry                                   ─ □ ✕  │
├──────────────────────────────────────────────────────────────────────┤
│ [ Otwórz obraz ] [ Otwórz wideo ] [ Kamera ▸ ] [ Ustawienia ] [ ⏺ ]  │
├────────────────────────────────────────────┬─────────────────────────┤
│                                            │  WYKRYTE KARTY          │
│                                            │  ┌────────────────────┐ │
│           PODGLĄD OBRAZU / KAMERY          │  │ ♥ 7    98,3%  ●    │ │
│                                            │  │ ♠ A    97,1%  ●    │ │
│         (ramki + etykiety + pewność)       │  │ ♦ 10   95,8%  ●    │ │
│                                            │  │ ?  —   53,1%  ○    │ │
│                                            │  └────────────────────┘ │
│                                            ├─────────────────────────┤
│                                            │  STATYSTYKI             │
│                                            │  Wykryte karty:    4    │
│                                            │  Pewne:            3    │
│                                            │  Śr. pewność:  97,1%    │
│                                            │  Czas detekcji:  — ms   │
│                                            │  Czas klasyf.:   — ms   │
│                                            │  Łącznie:        — ms   │
│                                            │  FPS:            —      │
├────────────────────────────────────────────┴─────────────────────────┤
│  HISTORIA   [12:04:31] 4 karty  │  [12:03:58] 2 karty  │  [ Eksport ] │
└──────────────────────────────────────────────────────────────────────┘
```

Wartości statystyk są w tym szkicu celowo puste — **wypełni je działający system**,
nie ten dokument.

### 12.3 Panel ustawień

Elementy sterujące odpowiadające bezpośrednio parametrom badanym w eksperymentach — dzięki
temu GUI staje się także narzędziem demonstracyjnym podczas obrony pracy:

- wybór modelu klasyfikatora (lista dostępnych wag),
- wybór detektora (konturowy / YOLO),
- przełącznik korekcji perspektywy,
- suwak progu pewności (z zaznaczoną wartością wyznaczoną w E9),
- suwak progu pewności detekcji,
- przełącznik wyświetlania miary pewności i identyfikatorów obiektów.

### 12.4 Architektura aplikacji — wątki

Naiwna implementacja odczytująca klatki w wątku GUI zamraża interfejs. Poprawne rozwiązanie:

```
Wątek GUI (główny)          Wątek akwizycji           Wątek inferencji
      │                            │                          │
      │  start                     │                          │
      ├───────────────────────────>│                          │
      │                       odczyt klatki                   │
      │                            ├─── kolejka (rozmiar 1) ──>│
      │                            │                     przetwarzanie
      │<────── sygnał: wynik ──────┼──────────────────────────┤
      │  odświeżenie widoku        │                          │
```

Kluczowe decyzje projektowe:
- **kolejka o rozmiarze 1 z nadpisywaniem** — przy wolniejszej inferencji niż akwizycji
  pomijamy zaległe klatki zamiast budować rosnące opóźnienie; system pokazuje stan bieżący,
  a nie sprzed sekund,
- komunikacja wyłącznie przez sygnały i sloty Qt (bezpieczne wątkowo),
- pomiar czasu każdego etapu osobno — zasila statystyki i eksperymenty wydajnościowe.

Ten fragment jest dobrym materiałem na podrozdział implementacyjny, bo pokazuje realny
problem inżynierski i jego rozwiązanie.

### 12.5 Historia i eksport

Każda analiza zapisywana jest jako rekord: znacznik czasu, źródło, lista kart z pewnością,
czasy etapów, użyta konfiguracja. Format: JSON Lines w `results/history.jsonl` (dopisywanie
bez przepisywania pliku). Eksport do CSV na żądanie.

Zastosowanie podwójne: funkcja użytkowa aplikacji **oraz** źródło danych do analizy
wydajnościowej w pracy.

---

## 13. Struktura repozytorium

### 13.1 Uwagi do struktury z koncepcji wyjściowej

Struktura zaproponowana w koncepcji wyjściowej jest sensowna, ale wymaga trzech korekt:

1. **Kod źródłowy w `src/`, nie w `app/`.** Układ `src-layout` wymusza instalację pakietu
   (`pip install -e .`), dzięki czemu importy działają jednakowo w skryptach, testach
   i notatnikach — bez modyfikowania `sys.path`. To eliminuje najczęstsze źródło problemów
   „u mnie działa".
2. **Rozdzielenie `scripts/` od `src/`.** `src/` zawiera logikę wielokrotnego użytku,
   `scripts/` — punkty wejścia uruchamiane z wiersza poleceń. Bez tego rozdziału kod
   biblioteczny miesza się z kodem uruchomieniowym i przestaje być testowalny.
3. **`experiments/` przechowuje wyniki, `configs/experiments/` — definicje.** Nie należy
   trzymać obu w jednym miejscu: definicje są wersjonowane i edytowane, wyniki są
   generowane i przyrastają.

### 13.2 Struktura docelowa

```
Cards_Classification_Problem/
│
├── README.md                       # opis projektu i instrukcja uruchomienia
├── .gitignore                      # (istnieje)
├── pyproject.toml                  # metadane pakietu + konfiguracja narzędzi
├── requirements.txt                # zależności z przypiętymi wersjami
│
├── configs/                        # WSZYSTKIE parametry — żadnych liczb w kodzie
│   ├── paths.yaml                  # ścieżki lokalne (nie wersjonowane, wzór w repo)
│   ├── detector/
│   │   ├── contour.yaml            # progi detektora klasycznego
│   │   ├── yolo_n.yaml
│   │   └── yolo_s.yaml
│   ├── classifier/
│   │   ├── simple_cnn.yaml
│   │   ├── resnet18.yaml
│   │   ├── mobilenetv3.yaml
│   │   └── multihead.yaml
│   ├── augmentation/
│   │   ├── none.yaml
│   │   ├── geometric.yaml
│   │   ├── photometric.yaml
│   │   └── full.yaml
│   ├── pipeline/
│   │   └── default.yaml            # konfiguracja systemu produkcyjnego
│   └── experiments/                # definicje eksperymentów E1–E12
│       ├── e01_architectures.yaml
│       ├── e02_transfer.yaml
│       └── ...
│
├── data/                           # (ignorowane, poza splits/ i metadata/)
│   ├── raw/                        # pobrane zbiory publiczne, nietknięte
│   │   ├── public_classification/
│   │   └── public_detection/
│   ├── own/                        # własne nagrania i zdjęcia
│   │   ├── videos/
│   │   └── frames/
│   ├── interim/                    # etapy pośrednie (wycięte karty, wyprostowane)
│   ├── synthetic/                  # sceny wygenerowane
│   ├── processed/                  # dane gotowe do treningu
│   ├── splits/                     # WERSJONOWANE — definicje podziałów
│   │   ├── classification_v1.json
│   │   └── detection_v1.json
│   └── metadata/                   # WERSJONOWANE
│       ├── sessions.csv            # opis sesji: oświetlenie, tło, talia, kąt
│       └── sources.md              # źródła i licencje zbiorów
│
├── docs/
│   ├── PROJECT_PLAN.md             # ten dokument
│   ├── DATASET.md                  # raport z audytu danych (Faza 2)
│   ├── EXPERIMENTS.md              # rejestr przeprowadzonych eksperymentów
│   ├── JOURNAL.md                  # dziennik prac — kluczowy przy nierównym tempie
│   ├── THESIS_OUTLINE.md           # struktura pracy inżynierskiej
│   ├── img/                        # rysunki do dokumentacji
│   └── Zastosowanie ... .docx      # tekst pracy inżynierskiej
│
├── experiments/                    # WYNIKI (config + metryki + wykresy w gicie)
│   └── 2026-09-20_e01_resnet18_s42/
│       ├── config.yaml             # pełna kopia konfiguracji przebiegu
│       ├── environment.json        # wersje bibliotek, GPU, commit
│       ├── metrics/
│       │   ├── summary.json
│       │   ├── per_class.csv
│       │   └── training_log.csv
│       ├── plots/                  # wykresy — trafiają wprost do pracy
│       └── weights/                # (ignorowane przez git)
│
├── models/                         # wagi produkcyjne (ignorowane, konfiguracje śledzone)
│   ├── detector/
│   └── classifier/
│
├── notebooks/                      # wyłącznie eksploracja, nigdy logika systemu
│   ├── 01_dataset_exploration.ipynb
│   └── 02_error_analysis.ipynb
│
├── results/                        # wyniki użytkowe aplikacji
│   ├── history.jsonl
│   └── exports/
│
├── scripts/                        # punkty wejścia CLI
│   ├── download_data.py
│   ├── audit_dataset.py            # audyt z sekcji 5.3
│   ├── extract_frames.py           # klatki z nagrań własnych
│   ├── build_splits.py             # generowanie podziałów
│   ├── generate_synthetic.py       # generator scen syntetycznych
│   ├── train_classifier.py
│   ├── train_detector.py
│   ├── evaluate.py
│   ├── run_experiment.py           # uruchamia eksperyment z pliku YAML
│   ├── make_report.py              # wykresy i tabele do pracy
│   └── run_app.py                  # start GUI
│
├── src/
│   └── cardvision/
│       ├── __init__.py
│       ├── config.py               # wczytywanie i walidacja konfiguracji
│       ├── labels.py               # 52 karty, mapowania figura/kolor — jedno źródło prawdy
│       │
│       ├── io/
│       │   ├── image_source.py     # jednolity interfejs: plik / katalog / wideo / kamera
│       │   └── writers.py          # zapis JSON/CSV/obrazów z wynikami
│       │
│       ├── preprocessing/
│       │   ├── basic.py            # zmiana rozmiaru, letterbox, normalizacja
│       │   ├── enhance.py          # CLAHE, gamma, balans bieli (E10a)
│       │   └── quality.py          # detekcja rozmycia, ocena jasności
│       │
│       ├── detection/
│       │   ├── base.py             # abstrakcyjny CardDetector
│       │   ├── contour.py          # detektor klasyczny (baseline, E7)
│       │   ├── yolo.py             # detektor YOLO
│       │   └── postprocess.py      # NMS, filtry geometryczne
│       │
│       ├── geometry/
│       │   ├── corners.py          # wykrywanie i porządkowanie narożników
│       │   └── rectify.py          # transformacja perspektywiczna
│       │
│       ├── classification/
│       │   ├── base.py             # abstrakcyjny CardClassifier
│       │   ├── models.py           # SimpleCNN, ResNet18, MobileNetV3, MultiHead
│       │   ├── dataset.py          # Dataset i DataLoader
│       │   ├── augment.py          # polityki augmentacji z sekcji 6
│       │   ├── train.py            # pętla ucząca
│       │   └── calibration.py      # temperature scaling, ECE
│       │
│       ├── synth/
│       │   ├── compose.py          # kompozycja scen syntetycznych
│       │   └── backgrounds.py
│       │
│       ├── pipeline/
│       │   ├── card_pipeline.py    # spina wszystko — SERCE SYSTEMU
│       │   └── result.py           # typy wyników (CardResult, FrameResult)
│       │
│       ├── evaluation/
│       │   ├── metrics_det.py      # IoU, mAP, precision, recall
│       │   ├── metrics_cls.py      # accuracy, F1, top-k
│       │   ├── metrics_e2e.py      # metryka end-to-end z sekcji 10.3
│       │   ├── confusion.py        # macierze pomyłek
│       │   └── plots.py            # wykresy publikacyjne
│       │
│       ├── app/                    # GUI (PySide6)
│       │   ├── main_window.py
│       │   ├── camera_worker.py    # wątek akwizycji
│       │   ├── inference_worker.py # wątek inferencji
│       │   └── widgets/
│       │
│       └── utils/
│           ├── seed.py             # ustawianie ziarna dla wszystkich bibliotek
│           ├── timing.py           # pomiary czasu etapów
│           ├── logging.py
│           └── experiment.py       # tworzenie katalogu przebiegu, zapis środowiska
│
└── tests/
    ├── conftest.py
    ├── fixtures/                   # małe obrazy testowe (wyjątek w .gitignore)
    ├── unit/
    └── integration/
```

### 13.3 Zasady architektoniczne

1. **Zero magicznych liczb w kodzie.** Każdy próg, rozmiar i współczynnik pochodzi
   z konfiguracji. Umożliwia to eksperymentowanie bez modyfikacji kodu.
2. **Interfejsy abstrakcyjne w punktach wymienności.** `CardDetector` i `CardClassifier`
   definiują kontrakt; implementacje są wybierane po nazwie z konfiguracji.
3. **`pipeline/card_pipeline.py` nie wie, jaki detektor i klasyfikator dostał.** Dzięki
   temu ta sama ścieżka kodu obsługuje wszystkie warianty eksperymentalne — nie ma ryzyka,
   że wariant A i B różnią się czymś więcej niż badaną zmienną.
4. **GUI nie zawiera logiki.** Wywołuje wyłącznie pipeline. Gwarantuje to, że wyniki z GUI
   i z CLI są identyczne.
5. **Notatniki nie zawierają logiki systemu** — służą wyłącznie eksploracji. Wszystko,
   co ma być powtarzalne, żyje w `src/`.

---

## 14. Reprodukowalność

### 14.1 Katalog przebiegu eksperymentu

Każde uruchomienie tworzy katalog `experiments/<data>_<eksperyment>_<wariant>_s<ziarno>/`
zawierający:

| Plik | Zawartość |
|---|---|
| `config.yaml` | **pełna, rozwinięta** konfiguracja — nie odwołania do innych plików |
| `environment.json` | wersja Pythona, wersje kluczowych bibliotek, model GPU, sterownik CUDA, hash commita, informacja o niezacommitowanych zmianach |
| `metrics/summary.json` | metryki końcowe |
| `metrics/per_class.csv` | metryki dla każdej z 52 klas |
| `metrics/training_log.csv` | strata i metryki po każdej epoce |
| `plots/` | wykresy |
| `weights/` | punkty kontrolne (poza gitem) |

Dzięki temu odpowiedź na pytanie „z jakimi parametrami wykonano ten eksperyment" jest
zawsze dostępna w jednym pliku, także po pół roku.

### 14.2 Determinizm

`src/cardvision/utils/seed.py` ustawia ziarno dla: modułu `random`, NumPy, PyTorcha (CPU
i CUDA) oraz konfiguruje deterministyczne algorytmy cuDNN.

**Uwaga uczciwościowa:** pełny determinizm na GPU wymaga wyłączenia niedeterministycznych
jąder obliczeniowych, co spowalnia trening. Rekomendacja: włączyć determinizm dla
eksperymentów raportowanych w pracy, a dla wstępnych prób pozostawić tryb szybki — i **ten
fakt odnotować w pracy**, zamiast deklarować pełną powtarzalność bez pokrycia.

### 14.3 Wersjonowanie danych

Bez narzędzi typu DVC, prosto i wystarczająco:
- `data/metadata/sources.md` — nazwa zbioru, adres, autor, licencja, data pobrania,
  liczba plików, suma kontrolna SHA-256 archiwum,
- `data/splits/*.json` z numerem wersji — zmiana podziału oznacza nowy plik, nigdy edycję
  istniejącego,
- `config.yaml` przebiegu zawiera nazwę użytego pliku podziału.

### 14.4 Rejestr eksperymentów

`docs/EXPERIMENTS.md` — tabela uzupełniana po każdym przebiegu: identyfikator, data, badana
zmienna, kluczowy wynik, wniosek, ścieżka do katalogu wyników. To jednocześnie szkielet
rozdziału eksperymentalnego pracy.

### 14.5 Dziennik prac

`docs/JOURNAL.md` — wpis po każdej sesji pracy: data, co zrobiono, co nie działa,
**co jest następnym krokiem**. Przy nierównym tempie pracy to najważniejszy plik w projekcie:
pozwala wrócić po trzech tygodniach przerwy bez odtwarzania kontekstu z pamięci.

Format wpisu:

```markdown
## 2026-09-20 — Faza 2, zadania F2-T1..T3
**Zrobione:** pobrano zbiór X, uruchomiono audyt, znaleziono 412 duplikatów.
**Problemy:** zbiór zawiera jokery (53. klasa) — usunięte.
**Następny krok:** F2-T4 — zbudować podział train/val/test i zweryfikować rozłączność.
```

---

## 15. Testy

### 15.1 Filozofia

Testy nie mają tu na celu pokrycia kodu w 100%. Mają zabezpieczyć **cztery obszary,
w których błąd jest cichy i unieważnia wyniki eksperymentów** — a więc i całą pracę.

### 15.2 Testy jednostkowe

| Moduł | Co testujemy | Dlaczego to krytyczne |
|---|---|---|
| `labels.py` | wzajemna odwracalność mapowań karta ↔ (figura, kolor); kompletność 52 klas; stabilność kolejności indeksów | **błąd tutaj przesuwa wszystkie etykiety i unieważnia każdy eksperyment**, nie powodując żadnego widocznego błędu |
| `geometry/corners.py` | porządkowanie 4 punktów dla różnych kolejności wejściowych, obrotów i przypadków zdegenerowanych | ciche pomylenie narożników daje obraz obrócony lub odbity |
| `geometry/rectify.py` | prostowanie syntetycznie zniekształconego prostokąta odtwarza oryginał; zachowanie proporcji; ścieżka awaryjna | jw. |
| `detection/postprocess.py` | NMS na przygotowanych przypadkach; filtr proporcji | błąd usuwa poprawne detekcje |
| `evaluation/metrics_*.py` | metryki na ręcznie policzonych przykładach (znane IoU, znana macierz pomyłek) | **błędna implementacja metryki daje fałszywe wyniki w pracy** |
| `classification/calibration.py` | temperature scaling nie zmienia kolejności klas; ECE = 0 dla modelu idealnie skalibrowanego | |
| `preprocessing/basic.py` | letterbox zachowuje proporcje; odwracalność przekształcenia współrzędnych | błędne przeliczenie współrzędnych przesuwa bboxy |
| `config.py` | wczytanie i walidacja; błąd przy nieznanym kluczu | literówka w konfiguracji nie może po cichu wracać do wartości domyślnej |

Priorytet bezwzględny mają `labels.py` i `evaluation/`. To są miejsca, gdzie błąd nie
powoduje awarii, tylko fałszywe liczby w pracy.

### 15.3 Testy integracyjne

| Test | Zakres |
|---|---|
| Pipeline na obrazie testowym | plik → detekcja → wycięcie → prostowanie → klasyfikacja → wynik o oczekiwanej strukturze |
| Wymienność implementacji | ten sam obraz przez `ContourDetector` i `YoloDetector` — oba zwracają wynik zgodny z kontraktem |
| Obsługa przypadków brzegowych | obraz bez kart, obraz całkowicie czarny, plik uszkodzony, obraz 1×1 px |
| Ścieżka konfiguracji | uruchomienie z każdego pliku w `configs/pipeline/` kończy się powodzeniem |
| Determinizm | dwa uruchomienia z tym samym ziarnem dają identyczny wynik |

### 15.4 Testy praktyczne (manualne, protokołowane)

Scenariusze wykonywane ręcznie przed zamknięciem fazy, z zapisem wyniku w `docs/JOURNAL.md`:

| # | Scenariusz | Kryterium powodzenia |
|---|---|---|
| P1 | 1 karta, dobre światło, na wprost | rozpoznana poprawnie, pewność powyżej progu |
| P2 | 5 kart rozłożonych | wszystkie wykryte i rozpoznane |
| P3 | 10 kart | wszystkie wykryte; czas w budżecie |
| P4 | Karty nachodzące na siebie | brak zawieszenia; niepewne oznaczone jako niepewne |
| P5 | Słabe oświetlenie | ostrzeżenie o jakości obrazu lub odrzucenie zamiast błędnej klasy |
| P6 | Karta pod kątem powyżej 45° | prostowanie działa lub następuje kontrolowany fallback |
| P7 | Karta rewersem do góry | brak fałszywej klasyfikacji z wysoką pewnością |
| P8 | Obiekt niebędący kartą (telefon, kubek) | odrzucony |
| P9 | Tryb kamery, 60 s ciągłej pracy | brak wycieku pamięci, stabilny FPS |
| P10 | Brak podłączonej kamery | czytelny komunikat, brak awarii aplikacji |

---

## 16. Wymagania sprzętowe

### 16.1 Sprzęt dostępny (wykryty na stanowisku)

| Element | Parametr | Ocena |
|---|---|---|
| CPU | Intel Core i5-12600KF | wystarczający z zapasem |
| GPU | NVIDIA GeForce RTX 3060 Ti, 8 GB VRAM | **w pełni wystarczający** do całego projektu |
| RAM | 16 GB | wystarczający, przy dużych zbiorach warto ograniczyć `num_workers` |
| System | Windows 11 | obsługiwany przez cały stos |

**Wniosek:** projekt w całości wykonalny lokalnie. Nie ma potrzeby korzystania z Google Colab
ani zasobów chmurowych, co upraszcza reprodukowalność i eliminuje ryzyko utraty sesji.

### 16.2 Szacowane zapotrzebowanie

| Zasób | Szacunek | Uwagi |
|---|---|---|
| VRAM — trening klasyfikatora | 3–5 GB przy batchu 64 i wejściu 224×160 | mieści się; przy problemach zmniejszyć batch lub włączyć AMP |
| VRAM — trening YOLO (wariant `n`, obraz 640) | 4–6 GB przy batchu 16 | mieści się |
| RAM | 8–12 GB podczas treningu | `num_workers = 4` jako punkt wyjścia |
| Dysk — zbiory publiczne | 5–15 GB | `[DO WERYFIKACJI]` po wyborze zbioru |
| Dysk — dane własne i syntetyczne | 10–20 GB | nagrania wideo zajmują najwięcej |
| Dysk — punkty kontrolne i eksperymenty | 10–20 GB | przy kilkudziesięciu przebiegach |
| **Dysk łącznie** | **50–70 GB zapasu** | warto sprawdzić przed startem |

### 16.3 Czasy — do zmierzenia

Czasy treningu i inferencji **nie są tu podawane**, ponieważ nie zostały zmierzone.
Do wypełnienia po Fazach 4 i 5:

| Pomiar | Wartość |
|---|---|
| Czas treningu SimpleCNN (30 epok) | `[DO ZMIERZENIA]` |
| Czas treningu ResNet18 (fine-tuning, 30 epok) | `[DO ZMIERZENIA]` |
| Czas treningu YOLO wariant `n` (100 epok) | `[DO ZMIERZENIA]` |
| Inferencja klasyfikatora (na kartę, GPU) | `[DO ZMIERZENIA]` |
| Inferencja detektora (na klatkę, GPU) | `[DO ZMIERZENIA]` |
| FPS pipeline'u end-to-end, 5 kart w kadrze | `[DO ZMIERZENIA]` |

### 16.4 Wariant bez wydajnego GPU

Na wypadek awarii sprzętu lub pracy na innym komputerze:

- klasyfikator w wariancie MobileNetV3-Small przy rozdzielczości 128×91 trenuje się na CPU
  w akceptowalnym czasie,
- detektor konturowy nie wymaga GPU w ogóle — pipeline pozostaje w pełni funkcjonalny,
- YOLO wariant `n` w trybie inferencji działa na CPU, choć z niższym FPS,
- ostateczność: Google Colab do samego treningu, z pobraniem wag na maszynę lokalną.

Ten wariant warto opisać w pracy jako analizę wymagań wdrożeniowych systemu.

---

## 17. Fazy implementacji

### 17.0 Uwagi do kolejności faz

Kolejność różni się od tej zaproponowanej w koncepcji wyjściowej w trzech miejscach — każda zmiana ma
uzasadnienie:

1. **Klasyfikator (Faza 4) przed detektorem YOLO (Faza 5).**
   Powody: (a) publiczne dane klasyfikacyjne są dostępne od razu i nie wymagają anotacji;
   (b) klasyfikacja 52 kart to właściwy temat pracy, detekcja jednej klasy „karta" jest
   zadaniem łatwiejszym; (c) detektor konturowy z Fazy 3 pozwala uzyskać działający
   pipeline end-to-end **zanim** YOLO w ogóle powstanie. Dzięki temu ryzyko „nie udało się
   wytrenować YOLO, nie mam nic" znika.

2. **Korekcja perspektywy (część Fazy 3) przed oboma modelami.**
   Jest niezbędna do przygotowania wycinków dla klasyfikatora i nie zależy od żadnego
   modelu — to czyste OpenCV.

3. **Własny zbiór testowy (Faza 7) po pierwszej integracji (Faza 6).**
   Dopiero mając działający system wiadomo, jakich danych naprawdę potrzeba i jakie
   przypadki są trudne. Zbieranie danych „w ciemno" na początku groziłoby nagraniem
   materiału nieprzydatnego.

### 17.1 Mapa zależności

```
F1 środowisko
 └─> F2 dane
      ├─> F3 preprocessing + geometria + detektor konturowy
      │    ├─> F4 klasyfikator ──┐
      │    └─> F5 detektor YOLO ─┤
      │                          └─> F6 INTEGRACJA (MVP) ──┬─> F7 własne dane ─┐
      │                                                     ├─> F8 confidence  │
      │                                                     ├─> F11 GUI ─> F12 kamera
      │                                                     └─> F9 eksperymenty klasyfikacji
      │                                                              └─> F10 eksperymenty systemu <─┘
      └───────────────────────────────────────────────> F13 testy (równolegle)
                                                        F14 rozszerzenia (opcjonalne)
                                                        F15 finalizacja
```

**Punkt krytyczny: Faza 6.** Po jej ukończeniu istnieje działający system end-to-end.
Wszystko, co następuje później, podnosi jakość i wartość naukową, ale projekt jest już
obronny. Do Fazy 6 należy dojść bez zbaczania na boki.

### 17.2 Szacunkowy budżet czasu

| Faza | Temat | Szacunek | Skumulowany |
|---|---|---|---|
| F1 | Środowisko i szkielet | 8–10 h | 10 h |
| F2 | Dane: pozyskanie i audyt | 10–14 h | 24 h |
| F3 | Preprocessing, geometria, detektor klasyczny | 12–16 h | 40 h |
| F4 | Klasyfikator | 14–18 h | 58 h |
| F5 | Detektor YOLO | 12–16 h | 74 h |
| **F6** | **Integracja — MVP** | **10–12 h** | **86 h** |
| F7 | Własny zbiór testowy | 10–14 h | 100 h |
| F8 | Confidence i obsługa błędów | 10–12 h | 112 h |
| F9 | Eksperymenty klasyfikacji | 20–26 h | 138 h |
| F10 | Eksperymenty systemowe | 20–26 h | 164 h |
| F11 | GUI | 16–20 h | 184 h |
| F12 | Tryb kamery | 10–14 h | 198 h |
| F13 | Testy i jakość | 8–12 h | 210 h |
| F14 | Rozszerzenia | 0–20 h | 230 h |
| F15 | Finalizacja | 12–16 h | 246 h |

**To są szacunki nakładu pracy, nie pomiary.** Realnie należy doliczyć narzut na problemy
techniczne — przyjmij margines 30%. Zakres obowiązkowy (F1–F10, F13, F15) to około
150–190 h; GUI i tryb kamery (F11, F12) dokładają 26–34 h.

---

### FAZA 0 — Analiza wymagań i plan `[UKOŃCZONA]`

**Cel:** ustalić zakres, architekturę i metodologię przed napisaniem pierwszej linii kodu.

**Rezultat:** ten dokument oraz `README.md`.

**Kryterium przejścia:** ustalona architektura dwuetapowa, lista technologii i strategia
danych.

---

### FAZA 1 — Środowisko i szkielet projektu

**Cel:** działające, odtwarzalne środowisko oraz szkielet repozytorium, w którym można
zacząć pisać kod bez późniejszej reorganizacji.

**Dlaczego teraz:** wykryty na maszynie Python 3.14 jest niezgodny ze stosem PyTorch —
to trzeba rozwiązać, zanim cokolwiek innego ruszy.

**Szacunek:** 8–10 h

| ID | Zadanie | h | Wynik |
|---|---|---|---|
| F1-T1 | Zainstalować Pythona 3.12 obok istniejącego 3.14; utworzyć `.venv` w katalogu projektu | 1 | działający interpreter 3.12 |
| F1-T2 | Zainstalować PyTorch z obsługą CUDA odpowiednią dla RTX 3060 Ti; zweryfikować `torch.cuda.is_available()` | 1 | GPU widoczne z PyTorcha |
| F1-T3 | Zainstalować pozostałe zależności; wygenerować `requirements.txt` z **przypiętymi wersjami** | 1 | `requirements.txt` |
| F1-T4 | Utworzyć strukturę katalogów z sekcji 13.2 wraz z plikami `.gitkeep` | 1 | szkielet repozytorium |
| F1-T5 | `pyproject.toml` + `pip install -e .`; sprawdzić `import cardvision` z dowolnego katalogu | 1 | pakiet instalowalny |
| F1-T6 | **Zweryfikować posiadane talie** według listy kontrolnej z sekcji 5.7; zapisać opis (rozmiar, krój, symetria obrotu) w `docs/DATASET.md` | 0,5 | talie opisane, augmentacja obrotem potwierdzona |
| F1-T7 | Zaimplementować `src/cardvision/labels.py` — 52 karty, mapowania w obie strony | 1,5 | moduł etykiet |
| F1-T8 | Napisać testy jednostkowe dla `labels.py` (kompletność, odwracalność, stabilność indeksów) | 1 | testy przechodzą |
| F1-T9 | Zaimplementować `utils/seed.py` i `utils/experiment.py` (katalog przebiegu, zapis środowiska) | 1,5 | podstawa reprodukowalności |
| F1-T10 | Skonfigurować `pytest`, uruchomić pełny zestaw testów; założyć `docs/JOURNAL.md` | 0,5 | `pytest` zielony |
| F1-T11 | Pierwszy commit; ustalić konwencję commitów | 0,5 | historia gita rozpoczęta |

**Pliki:** `pyproject.toml`, `requirements.txt`, `src/cardvision/labels.py`,
`src/cardvision/utils/{seed,experiment}.py`, `tests/unit/test_labels.py`, `docs/JOURNAL.md`

**Weryfikacja:**
```
python -c "import torch; print(torch.__version__, torch.cuda.is_available())"   → True
python -c "import cardvision; print(cardvision.__version__)"                    → działa
pytest                                                                          → wszystkie zielone
```

**DoD fazy:** środowisko działa na GPU, pakiet się importuje, testy `labels.py` przechodzą,
pierwszy commit w repozytorium.

**Kryterium przejścia:** wszystkie trzy komendy weryfikacyjne kończą się powodzeniem.

| Możliwy problem | Reakcja |
|---|---|
| Brak wersji PyTorcha dla Pythona 3.12 z CUDA | zejść do Pythona 3.11 — dobrze wspierany przez cały stos |
| Konflikt wersji CUDA ze sterownikiem | zainstalować wariant PyTorcha dopasowany do sterownika; ostatecznie wariant CPU do czasu naprawy |
| `import cardvision` nie działa | sprawdzić, czy `pip install -e .` wykonano w aktywnym `.venv` |

---

### FAZA 2 — Dane: pozyskanie i audyt

**Cel:** mieć zweryfikowany, opisany i podzielony zbiór danych, o którym wiadomo, co
zawiera i skąd pochodzi.

**Dlaczego teraz:** wszystkie kolejne fazy zależą od danych, a wady zbioru wykryte później
unieważniają wykonaną pracę.

**Szacunek:** 10–14 h

| ID | Zadanie | h | Wynik |
|---|---|---|---|
| F2-T1 | Przegląd dostępnych publicznych zbiorów wg kryteriów z sekcji 5.2; wybór kandydatów | 2 | lista kandydatów |
| F2-T2 | Pobranie zbioru klasyfikacyjnego; zapis źródła, licencji i sumy kontrolnej w `data/metadata/sources.md` | 1 | dane w `data/raw/` |
| F2-T3 | Pobranie zbioru detekcyjnego; jw. | 1 | dane w `data/raw/` |
| F2-T4 | Implementacja `scripts/audit_dataset.py` — kontrole z tabeli w sekcji 5.3 | 3 | narzędzie audytu |
| F2-T5 | Uruchomienie audytu; **ręczny przegląd losowej próbki 100 obrazów** pod kątem błędnych etykiet | 2 | raport z liczbami |
| F2-T6 | Czyszczenie: usunięcie duplikatów, jokerów, obrazów uszkodzonych i o zbyt niskiej rozdzielczości | 1,5 | oczyszczony zbiór |
| F2-T7 | `scripts/build_splits.py` — podział train/val/test z zachowaniem zasad z sekcji 5.5 | 2 | `data/splits/*.json` |
| F2-T8 | Weryfikacja rozłączności podziałów (pHash między zbiorami — musi wyjść zero kolizji) | 1 | dowód braku przecieku |
| F2-T9 | Napisanie `docs/DATASET.md` — pełny raport z audytu | 1,5 | dokument do pracy |

**Pliki:** `scripts/{audit_dataset,build_splits}.py`, `data/splits/*.json`,
`data/metadata/{sources.md,sessions.csv}`, `docs/DATASET.md`

**Weryfikacja:**
- histogram klas pokazuje wszystkie 52 klasy z rozsądną licznością,
- kontrola pHash między `train` i `test` zwraca zero kolizji,
- ręczny przegląd 100 obrazów: liczba błędnych etykiet `[DO ZMIERZENIA]` i akceptowalna.

**DoD fazy:** `docs/DATASET.md` opisuje zbiór liczbowo, podziały są zapisane i zweryfikowane
jako rozłączne.

**Kryterium przejścia:** zero kolizji pHash między podziałami. Jeśli kolizje występują —
**nie przechodzić dalej**, poprawić podział.

| Możliwy problem | Reakcja |
|---|---|
| Żaden publiczny zbiór nie spełnia kryteriów | zbudować zbiór klasyfikacyjny przez wycięcie kart ze zbioru detekcyjnego; ostatecznie przenieść nacisk na dane własne i przyspieszyć Fazę 7 |
| Zbiór jest już zaugmentowany | odrzucić, albo zidentyfikować oryginały po pHash i pracować tylko na nich; jeśli niemożliwe — zaznaczyć w pracy, że E4 wykonano na innym zbiorze |
| Silna nierównowaga klas | ważenie klas w funkcji straty lub `WeightedRandomSampler`; odnotować w `DATASET.md` |
| Zbiór zawiera 53 klasy z jokerem | usunąć jokera; odnotować decyzję |
| Niejasna licencja | odrzucić zbiór — ryzyko formalne przy pracy dyplomowej jest zbyt duże |

---

### FAZA 3 — Preprocessing, geometria i detektor klasyczny

**Cel:** działający, klasowo-agnostyczny detektor kart oparty na klasycznym CV oraz moduł
prostowania perspektywy. Po tej fazie można wycinać i normalizować karty bez żadnego modelu
uczonego.

**Dlaczego teraz:** to fundament, od którego zależą obie ścieżki (klasyfikator i YOLO),
a jednocześnie punkt odniesienia do eksperymentu E7 i zabezpieczenie na wypadek problemów
z uczeniem detektora.

**Szacunek:** 12–16 h

| ID | Zadanie | h | Wynik |
|---|---|---|---|
| F3-T1 | `io/image_source.py` — jednolity interfejs źródła: plik, katalog, wideo, kamera | 2 | wspólne wejście dla całego systemu |
| F3-T2 | `preprocessing/basic.py` — letterbox, zmiana rozmiaru, normalizacja, przeliczanie współrzędnych | 2 | podstawa preprocessingu |
| F3-T3 | Testy jednostkowe `basic.py` — zachowanie proporcji, odwracalność przeliczenia współrzędnych | 1 | testy zielone |
| F3-T4 | `detection/base.py` — abstrakcyjny interfejs `CardDetector` (kontrakt wejścia i wyjścia) | 1 | punkt wymienności |
| F3-T5 | `detection/contour.py` — detektor konturowy: progowanie, kontury, filtr pola i proporcji | 3 | działający baseline detekcji |
| F3-T6 | `geometry/corners.py` — wykrywanie i porządkowanie narożników, wyznaczanie orientacji | 2 | moduł narożników |
| F3-T7 | `geometry/rectify.py` — homografia i prostowanie, ze ścieżką awaryjną i flagą `rectified` | 2 | moduł prostowania |
| F3-T8 | Testy jednostkowe geometrii — syntetycznie zniekształcony prostokąt musi zostać odtworzony | 1,5 | testy zielone |
| F3-T9 | `preprocessing/quality.py` — detekcja rozmycia (wariancja laplasjanu), ocena jasności | 1 | filtry jakości |
| F3-T10 | Skrypt poglądowy: obraz wejściowy → wykryte i wyprostowane karty zapisane jako pliki | 1,5 | wizualna weryfikacja |

**Pliki:** `src/cardvision/{io,preprocessing,detection,geometry}/*`, `tests/unit/test_geometry.py`

**Weryfikacja:** na 20 obrazach testowych ze zbioru publicznego uruchomić skrypt poglądowy
i **obejrzeć wyniki**. Odsetek poprawnie wyprostowanych kart: `[DO ZMIERZENIA]`.

**DoD fazy:** dla obrazu z kilkoma kartami na kontrastowym tle system zwraca wyprostowane
wycinki kart; testy geometrii przechodzą.

**Kryterium przejścia:** wizualna kontrola potwierdza, że wycinki są kartami w orientacji
pionowej, z czytelnym indeksem narożnym.

| Możliwy problem | Reakcja |
|---|---|
| Progowanie zawodzi na wzorzystym tle | progowanie adaptacyjne zamiast Otsu; dodać operacje morfologiczne; docelowo to właśnie ma naprawić YOLO — i jest to wynik do E7 |
| `approxPolyDP` zwraca inną liczbę wierzchołków niż 4 | dobrać tolerancję proporcjonalnie do obwodu; przy 5–6 wierzchołkach użyć `minAreaRect` jako przybliżenia |
| Karta wyprostowana „na leżąco" | poprawić logikę orientacji w `corners.py`; dodać test regresyjny |
| Karty stykające się dają jeden kontur | filtr proporcji odrzuci taki obiekt; przypadek do udokumentowania jako ograniczenie detektora klasycznego |

---

### FAZA 4 — Klasyfikator kart

**Cel:** wytrenowany klasyfikator 52 klas z udokumentowanym wynikiem na zbiorze walidacyjnym.

**Dlaczego teraz:** to rdzeń tematu pracy. Dane są gotowe (F2), wycinki można wygenerować (F3).

**Szacunek:** 14–18 h

| ID | Zadanie | h | Wynik |
|---|---|---|---|
| F4-T1 | `classification/dataset.py` — Dataset wczytujący wycinki wg pliku podziału | 2 | ładowanie danych |
| F4-T2 | `classification/augment.py` — polityki augmentacji z sekcji 6 jako konfiguracje YAML | 2 | 4 polityki augmentacji |
| F4-T3 | Wizualna weryfikacja augmentacji: zapisać siatkę 64 zaugmentowanych obrazów i **obejrzeć je** | 1 | pewność, że augmentacja nie psuje danych |
| F4-T4 | `classification/models.py` — SimpleCNN | 2 | model bazowy |
| F4-T5 | `classification/models.py` — ResNet18 i MobileNetV3 z podmienioną głową | 1,5 | modele pretrenowane |
| F4-T6 | `classification/models.py` — wariant MultiHead (13 + 4) | 1,5 | model do E3 |
| F4-T7 | `classification/train.py` — pętla ucząca: AMP, harmonogram LR, early stopping, logowanie | 3 | trener |
| F4-T8 | `scripts/train_classifier.py` — uruchamianie z konfiguracji, zapis katalogu przebiegu | 1,5 | punkt wejścia CLI |
| F4-T9 | Pierwszy pełny trening (ResNet18, augmentacja pełna); zapis wyników | 1,5 | pierwszy wynik `[DO ZMIERZENIA]` |
| F4-T10 | `evaluation/metrics_cls.py` + `evaluation/confusion.py`; testy jednostkowe metryk | 2 | metryki i macierz pomyłek |
| F4-T11 | Analiza pierwszej macierzy pomyłek — czy pomyłki są sensowne, czy sygnalizują błąd w danych | 1 | wnioski w `JOURNAL.md` |

**Pliki:** `src/cardvision/classification/*`, `src/cardvision/evaluation/{metrics_cls,confusion}.py`,
`scripts/train_classifier.py`, `configs/classifier/*.yaml`, `configs/augmentation/*.yaml`

**Weryfikacja:** dokładność na zbiorze walidacyjnym `[DO ZMIERZENIA]`; macierz pomyłek nie
wykazuje wzorców świadczących o błędzie w etykietach (np. systematycznego przesunięcia klas).

**DoD fazy:** klasyfikator trenuje się do końca, zapisuje wagi i metryki, generuje macierz
pomyłek; wynik jest wyraźnie lepszy od losowego (1/52 ≈ 1,9%).

**Kryterium przejścia:** dokładność walidacyjna na poziomie użytecznym dla pipeline'u.
Jeżeli jest bardzo niska — **nie iść dalej**, tylko zdiagnozować: najczęstsze przyczyny to
błędne mapowanie etykiet, brak normalizacji ImageNet przy modelu pretrenowanym, albo
uszkodzone wycinki.

| Możliwy problem | Reakcja |
|---|---|
| Przeuczenie (duża luka train–val) | zwiększyć augmentację, dodać dropout i weight decay, rozważyć zamrożenie większej części backbone'u |
| Model nie uczy się w ogóle | sprawdzić LR, sprawdzić normalizację, spróbować przeuczyć celowo 10 obrazów — jeśli się nie da, błąd jest w kodzie, nie w danych |
| Kilka klas systematycznie mylonych | sprawdzić ich liczność i obejrzeć przykłady — częsta przyczyna to błędne etykiety w zbiorze publicznym |
| Trening trwa zbyt długo | zmniejszyć rozdzielczość (i tak badana w E8), zwiększyć batch, upewnić się, że AMP jest włączone |

---

### FAZA 5 — Detektor YOLO

**Cel:** wytrenowany detektor jednoklasowy o zmierzonej jakości, wymienialny z detektorem
konturowym bez zmian w pozostałym kodzie.

**Szacunek:** 12–16 h

| ID | Zadanie | h | Wynik |
|---|---|---|---|
| F5-T1 | Konwersja zbioru detekcyjnego do formatu YOLO ze **zredukowaniem wszystkich klas do jednej** („karta") | 2 | zbiór jednoklasowy |
| F5-T2 | `synth/compose.py` — generator scen syntetycznych z automatycznym wyliczaniem bboxów | 4 | generator |
| F5-T3 | Wygenerowanie ~5000 scen syntetycznych; wizualna kontrola poprawności bboxów | 1,5 | `data/synthetic/` |
| F5-T4 | Konfiguracja treningu YOLO — **jawne wyłączenie `fliplr` i `flipud`, obniżenie `hsv_h`** | 1 | `configs/detector/yolo_n.yaml` |
| F5-T5 | Trening wariantu `n`; zapis metryk i krzywych | 2 | wagi + mAP `[DO ZMIERZENIA]` |
| F5-T6 | `detection/yolo.py` — implementacja interfejsu `CardDetector` | 1,5 | detektor wymienny |
| F5-T7 | `evaluation/metrics_det.py` + testy jednostkowe (IoU na ręcznie policzonych przykładach) | 2 | metryki detekcji |
| F5-T8 | Analiza błędów: przegląd fałszywych detekcji i kart pominiętych | 1,5 | wnioski w `JOURNAL.md` |
| F5-T9 | Trening wariantu `s` do porównania | 1,5 | dane do sekcji 4.3 |

**Weryfikacja:** mAP@50 `[DO ZMIERZENIA]`, recall `[DO ZMIERZENIA]`; wizualna kontrola
detekcji na obrazach testowych.

**DoD fazy:** `YoloDetector` zwraca wyniki zgodne z tym samym kontraktem co `ContourDetector`;
metryki zapisane w katalogu przebiegu.

**Kryterium przejścia:** recall detekcji na tyle wysoki, by pipeline miał sens —
karty pominięte przez detektor są nie do odzyskania na dalszych etapach.

| Możliwy problem | Reakcja |
|---|---|
| Model uczony na danych syntetycznych zawodzi na rzeczywistych | zwiększyć realizm generatora (cienie, szum, rozmycie); zmienić proporcję danych syntetycznych do rzeczywistych — to wprost eksperyment E12 |
| Karty stykające się wykrywane jako jeden obiekt | obniżyć próg IoU w NMS; rozważyć detekcję obróconych prostokątów |
| Niedobór VRAM | zmniejszyć batch, zmniejszyć rozmiar obrazu wejściowego z 640 do 512 |
| Niska jakość mimo poprawnego treningu | zweryfikować konwersję anotacji — najczęstsze źródło błędu to pomylona kolejność współrzędnych |

---

### FAZA 6 — Integracja end-to-end `[KAMIEŃ MILOWY: MVP]`

**Cel:** jeden spójny system: obraz na wejściu, lista rozpoznanych kart na wyjściu.

**Dlaczego to punkt krytyczny:** po tej fazie projekt jest obronny nawet gdyby prace
zatrzymały się na dłużej. Wszystko dalsze podnosi jakość, ale nie warunkuje istnienia pracy.

**Szacunek:** 10–12 h

| ID | Zadanie | h | Wynik |
|---|---|---|---|
| F6-T1 | `pipeline/result.py` — typy wyników (`CardResult`, `FrameResult`) z pomiarami czasu etapów | 1,5 | kontrakt wyników |
| F6-T2 | `pipeline/card_pipeline.py` — spięcie: preprocessing → detekcja → wycięcie → prostowanie → klasyfikacja | 3 | **serce systemu** |
| F6-T3 | `config.py` — wczytywanie konfiguracji i budowa komponentów po nazwie z YAML | 2 | wymienność bez zmian w kodzie |
| F6-T4 | `io/writers.py` — zapis obrazu z naniesionymi ramkami oraz wyników do JSON/CSV | 1,5 | wyjście systemu |
| F6-T5 | `scripts/predict.py` — CLI dla pliku, katalogu i wideo | 1,5 | punkt wejścia |
| F6-T6 | Test integracyjny: pełny pipeline na obrazie z zestawu testowego | 1 | test zielony |
| F6-T7 | Testy przypadków brzegowych: brak kart, obraz czarny, plik uszkodzony | 1 | odporność |
| F6-T8 | Uruchomienie scenariuszy P1–P4 z sekcji 15.4; zapis wyników | 1 | protokół testów |

**Weryfikacja:**
```
python scripts/predict.py --input przyklad.jpg --config configs/pipeline/default.yaml
```
powinno wypisać listę kart z pewnościami i zapisać obraz z naniesionymi ramkami.

**DoD fazy:** jedna komenda przetwarza obraz i zwraca poprawne karty; działa zarówno
z detektorem konturowym, jak i z YOLO (zmiana wyłącznie w pliku konfiguracji).

**Kryterium przejścia:** scenariusze P1 i P2 przechodzą. **To jest moment na commit
oznaczony tagiem `mvp`.**

| Możliwy problem | Reakcja |
|---|---|
| Detekcja działa, klasyfikacja nie — mimo dobrego wyniku walidacyjnego | najczęstsza przyczyna: wycinki z pipeline'u wyglądają inaczej niż dane treningowe (inny preprocessing, inna normalizacja, inny kanał kolorów BGR/RGB). Zapisać wycinki z pipeline'u i porównać wizualnie z danymi treningowymi |
| Przetwarzanie jest bardzo wolne | zmierzyć czas etapów osobno — dopiero potem optymalizować; nie zgadywać, gdzie jest wąskie gardło |
| Wyniki różnią się między uruchomieniami | sprawdzić, czy ziarno jest ustawiane i czy model jest w trybie `eval()` |

---

### FAZA 7 — Własny zbiór testowy

**Cel:** zbiór testowy zebrany samodzielnie, opisany metadanymi warunków akwizycji —
podstawa głównego wyniku pracy i wszystkich eksperymentów odpornościowych.

**Dlaczego teraz:** dopiero mając działający system (F6) wiadomo, jakie przypadki są trudne
i jakiego materiału naprawdę potrzeba. Korzysta z talii opisanych w `F1-T6`.

**Szacunek:** 10–14 h (w tym ~2 h nagrywania)

| ID | Zadanie | h | Wynik |
|---|---|---|---|
| F7-T1 | Zaplanować sesje: macierz oświetlenie × tło × układ; przygotować `sessions.csv` z pustymi wierszami | 1 | plan akwizycji |
| F7-T2 | Nagrać sesje 1–4: wszystkie 52 karty rozłożone, cztery warunki oświetleniowe | 1,5 | nagrania |
| F7-T3 | Nagrać sesje 5–8: różne tła i różne kąty obserwacji | 1,5 | nagrania |
| F7-T4 | Nagrać sesje 9–12: układy wielokartowe, wachlarz, karty nachodzące, druga talia | 1,5 | nagrania |
| F7-T5 | `scripts/extract_frames.py` — wyodrębnianie co N-tej klatki z zapisem identyfikatora sesji | 1,5 | klatki + metadane |
| F7-T6 | Uzupełnić `data/metadata/sessions.csv`: oświetlenie, tło, talia, przybliżony kąt, liczba kart | 1 | metadane do E10 |
| F7-T7 | Anotacja bounding boxów dla ~100 scen (jedna klasa „karta") w LabelImg lub CVAT | 2,5 | zbiór detekcyjny własny |
| F7-T8 | Wygenerowanie wycinków klasyfikacyjnych i **ręczne nadanie im etykiet** | 2 | zbiór klasyfikacyjny własny |
| F7-T9 | Budowa `data/splits/own_test_v1.json` z zachowaniem rozłączności **na poziomie sesji** | 1 | podział własny |
| F7-T10 | Pierwsza ewaluacja obecnego systemu na własnym zbiorze — punkt odniesienia dla E11 | 1 | wynik `[DO ZMIERZENIA]` |

**Weryfikacja:** liczba obrazów na sesję, kompletność metadanych, rozłączność sesji między
podziałami.

**DoD fazy:** `test_own` istnieje, jest opisany metadanymi i został na nim wykonany pierwszy
pomiar.

**Kryterium przejścia:** każda z 52 klas ma co najmniej kilka reprezentantów w zbiorze
testowym; metadane kompletne dla wszystkich sesji.

| Możliwy problem | Reakcja |
|---|---|
| Ręczne etykietowanie 400 wycinków jest żmudne | wykorzystać obecny model do wstępnego oznaczenia i jedynie **poprawiać** jego błędy; uwaga: to wprowadza obciążenie na korzyść modelu — sprawdzić ręcznie losową próbkę i odnotować metodę w pracy |
| Wyniki na własnym zbiorze są dużo gorsze | **to nie jest porażka, to jest wynik E11** — udokumentować i zbadać przyczyny |
| Nagrania są rozmyte | użyć krótszego czasu naświetlania i lepszego światła; wykorzystać filtr rozmycia z `quality.py` do odsiania złych klatek |
| Odblaski zasłaniają indeksy | zmienić kąt oświetlenia; zachować część takich ujęć jako świadomie trudny podzbiór testowy |

---

### FAZA 8 — Confidence, kalibracja i obsługa błędów

**Cel:** system, który wie, czego nie wie — z progiem wyznaczonym na podstawie danych.

**Szacunek:** 10–12 h

| ID | Zadanie | h | Wynik |
|---|---|---|---|
| F8-T1 | `classification/calibration.py` — wykres niezawodności i ECE | 2 | pomiar kalibracji |
| F8-T2 | Implementacja temperature scaling; dopasowanie `T` na zbiorze walidacyjnym | 2 | kalibracja |
| F8-T3 | Testy jednostkowe kalibracji (niezmienność kolejności klas, ECE = 0 dla przypadku idealnego) | 1 | testy zielone |
| F8-T4 | Krzywa ryzyko–pokrycie; procedura wyboru progu z sekcji 9.4 | 2 | wyznaczony próg `[DO ZMIERZENIA]` |
| F8-T5 | Porównanie miary „maksimum softmax" z miarą „margines dwóch najlepszych" | 1 | wynik do E9 |
| F8-T6 | Integracja progu z pipeline'em: status `PEWNA` / `NIEPEWNA` | 1 | decyzja w systemie |
| F8-T7 | Implementacja katalogu sytuacji błędnych z sekcji 9.6 | 2 | odporność systemu |
| F8-T8 | Scenariusze P5–P8 z sekcji 15.4 | 1 | protokół testów |

**DoD fazy:** system zwraca „Nie udało się jednoznacznie rozpoznać karty" w sytuacjach
niepewnych zamiast zgadywać; próg pochodzi z udokumentowanej procedury, nie z wyboru
arbitralnego.

**Kryterium przejścia:** scenariusze P7 (rewers) i P8 (obiekt niebędący kartą) nie
generują fałszywych rozpoznań z wysoką pewnością.

---

### FAZA 9 — Framework eksperymentalny i eksperymenty klasyfikacji

**Cel:** przeprowadzenie eksperymentów E1, E2, E3, E4, E8 wraz z gotowymi materiałami
do pracy.

**Dlaczego framework najpierw:** ręczne uruchamianie kilkudziesięciu przebiegów i ręczne
zbieranie wyników jest źródłem błędów i pochłania czas. Kilka godzin na automatyzację
zwraca się wielokrotnie.

**Szacunek:** 20–26 h (z czego znaczna część to czas oczekiwania na treningi)

| ID | Zadanie | h | Wynik |
|---|---|---|---|
| F9-T1 | `scripts/run_experiment.py` — uruchamianie siatki wariantów z jednego pliku YAML, wiele ziaren | 3 | automatyzacja |
| F9-T2 | `scripts/make_report.py` — agregacja wyników do tabel i wykresów publikacyjnych | 3 | generator materiałów |
| F9-T3 | **E1** — porównanie architektur (3 modele × 3 ziarna) | 4 | tabela + wykres |
| F9-T4 | **E2** — transfer learning (3 warianty + wariant na 25% danych) | 3 | krzywe uczenia |
| F9-T5 | **E3** — dekompozycja 52 / 13+4 / multi-head (3 × 3 ziarna) | 4 | tabela + macierze pomyłek |
| F9-T6 | **E4** — augmentacja (5 wariantów × 3 ziarna), w tym wariant z błędnym odbiciem | 4 | wykres słupkowy |
| F9-T7 | **E8** — rozdzielczość wejścia (6 wariantów) | 2 | wykres dwuosiowy |
| F9-T8 | Uzupełnienie `docs/EXPERIMENTS.md` o wszystkie przebiegi i wnioski | 2 | szkielet rozdziału pracy |
| F9-T9 | Analiza błędów na podstawie macierzy pomyłek — które karty i dlaczego | 2 | materiał do rozdziału o analizie wyników |

**DoD fazy:** pięć eksperymentów przeprowadzonych, wyniki zapisane, wykresy wygenerowane,
wnioski spisane.

**Kryterium przejścia:** każdy eksperyment ma zapisaną konfigurację, metryki i wykres
w katalogu `experiments/`.

| Możliwy problem | Reakcja |
|---|---|
| Różnice między wariantami mieszczą się w rozrzucie ziaren | **to też jest wynik** — raportować z odchyleniem standardowym i wnioskiem „różnica nieistotna"; nie naciągać interpretacji |
| Treningi trwają zbyt długo | zmniejszyć rozdzielczość i liczbę epok dla eksperymentów porównawczych; ważna jest porównywalność wariantów, nie maksymalna dokładność |
| Wyniki niepowtarzalne mimo ustawionego ziarna | sprawdzić determinizm cuDNN i kolejność ładowania danych |

---

### FAZA 10 — Eksperymenty systemowe

**Cel:** przeprowadzenie eksperymentów dotyczących całego systemu: E5, E6, E7, E9, E10,
E11, E12.

**Szacunek:** 20–26 h

| ID | Zadanie | h | Wynik |
|---|---|---|---|
| F10-T1 | `evaluation/metrics_e2e.py` — metryka end-to-end z sekcji 10.3 + testy jednostkowe | 3 | metryka główna |
| F10-T2 | **E5** — korekcja perspektywy, z rozbiciem na przedziały kąta | 3 | wykres + decyzja projektowa |
| F10-T3 | **E7** — detektor konturowy kontra YOLO, z rozbiciem na warunki akwizycji | 3 | tabela + przykłady błędów |
| F10-T4 | Trening YOLO z 52 klasami na potrzeby E6 | 2 | model jednoetapowy |
| F10-T5 | **E6** — architektura jednoetapowa kontra dwuetapowa | 3 | uzasadnienie decyzji architektonicznej |
| F10-T6 | **E9** — kalibracja i próg, pełne opracowanie wyników z Fazy 8 | 2 | wykresy do pracy |
| F10-T7 | **E10a/b/c** — odporność na oświetlenie, kąt, liczbę kart i przesłonięcia | 4 | rozdział o odporności |
| F10-T8 | **E11** — domain gap: dane publiczne kontra własne | 3 | kluczowy wynik pracy |
| F10-T9 | **E12** — dane syntetyczne kontra rzeczywiste | 2 | wynik praktyczny |
| F10-T10 | Uzupełnienie `docs/EXPERIMENTS.md`; wybór konfiguracji produkcyjnej na podstawie wyników | 2 | `configs/pipeline/default.yaml` uzasadniony pomiarami |

**DoD fazy:** wszystkie zaplanowane eksperymenty wykonane; konfiguracja produkcyjna systemu
wybrana na podstawie zmierzonych wyników, a nie założeń.

**Kryterium przejścia:** dla każdej decyzji projektowej z sekcji 3 i 4 istnieje pomiar,
który ją uzasadnia lub podważa.

---

### FAZA 11 — Aplikacja GUI

**Cel:** desktopowa aplikacja prezentująca działanie systemu.

**Szacunek:** 16–20 h

| ID | Zadanie | h | Wynik |
|---|---|---|---|
| F11-T1 | Szkielet okna PySide6, układ z sekcji 12.2 | 3 | okno aplikacji |
| F11-T2 | Wczytywanie obrazu i wyświetlanie z naniesionymi ramkami | 2,5 | tryb obrazu |
| F11-T3 | Panel listy wykrytych kart z pewnością i statusem | 2 | panel wyników |
| F11-T4 | Panel statystyk: liczba kart, średnia pewność, czasy etapów | 2 | statystyki |
| F11-T5 | Panel ustawień z sekcji 12.3 (wybór modelu, przełączniki, suwaki progów) | 3 | konfiguracja z GUI |
| F11-T6 | Historia analiz z zapisem do `results/history.jsonl` i eksportem do CSV | 2,5 | historia |
| F11-T7 | Obsługa błędów w warstwie GUI: brak modelu, brak pliku, uszkodzony obraz | 1,5 | odporność |
| F11-T8 | Dopracowanie wyglądu: arkusz stylów, ikony, czytelna typografia | 2 | wygląd profesjonalny |
| F11-T9 | Zrzuty ekranu do dokumentacji i pracy | 1 | materiał ilustracyjny |

**DoD fazy:** aplikacja uruchamia się jedną komendą, wczytuje obraz, pokazuje wyniki
i statystyki, zapisuje historię.

| Możliwy problem | Reakcja |
|---|---|
| Faza pochłania nieproporcjonalnie dużo czasu | GUI jest priorytetem 7 z 7 — ograniczyć do trybu obrazu i podstawowych statystyk; CLI pozostaje pełnoprawnym interfejsem |
| Wyniki w GUI różnią się od CLI | oznacza to logikę w warstwie GUI — przenieść do `pipeline`, to naruszenie zasady 4 z sekcji 13.3 |

---

### FAZA 12 — Tryb kamery i praca w czasie rzeczywistym

**Cel:** rozpoznawanie kart na żywo ze stabilnym, zmierzonym FPS.

**Szacunek:** 10–14 h

| ID | Zadanie | h | Wynik |
|---|---|---|---|
| F12-T1 | `app/camera_worker.py` — wątek akwizycji z kolejką o rozmiarze 1 | 2,5 | akwizycja bez blokowania GUI |
| F12-T2 | `app/inference_worker.py` — wątek inferencji z sygnałami Qt | 2,5 | inferencja w tle |
| F12-T3 | Integracja z GUI: podgląd na żywo z nałożonymi wynikami | 2 | tryb kamery |
| F12-T4 | Licznik FPS i rozbicie czasów etapów w czasie rzeczywistym | 1,5 | pomiary |
| F12-T5 | Pomijanie klatek nieostrych (filtr z `quality.py`) | 1 | stabilniejszy wynik |
| F12-T6 | Wybór kamery i rozdzielczości w ustawieniach | 1 | konfigurowalność |
| F12-T7 | Scenariusze P9 i P10 — 60 s ciągłej pracy, brak kamery | 1,5 | protokół testów |
| F12-T8 | Pomiar FPS w scenariuszach 1, 5 i 10 kart | 1 | dane do C7 `[DO ZMIERZENIA]` |

**DoD fazy:** tryb kamery działa przez 60 s bez wzrostu zużycia pamięci i bez zamrażania
interfejsu; FPS zmierzony i zapisany.

**Kryterium przejścia:** brak wycieku pamięci; FPS wystarczający do płynnego użycia
(cel ≥ 15 FPS — jeśli nieosiągnięty, udokumentować i przejść do rozszerzenia X4).

| Możliwy problem | Reakcja |
|---|---|
| FPS zbyt niski | zmniejszyć rozdzielczość detektora; klasyfikować karty w jednej wsadowej partii zamiast pojedynczo; ograniczyć klasyfikację do kart nowych (wymaga X2) |
| Wyniki „migoczą" między klatkami | głosowanie po ostatnich N klatkach (rozszerzenie X2) — wyraźnie poprawia odbiór wizualny |
| Zamrożenia interfejsu | oznaczają wykonywanie inferencji w wątku GUI — naruszenie architektury z sekcji 12.4 |

---

### FAZA 13 — Testy i jakość kodu

**Cel:** domknięcie zestawu testów i uporządkowanie kodu przed finalizacją.

**Uwaga:** testy krytyczne (`labels`, geometria, metryki) powstają wcześniej — w fazach,
w których tworzone są odpowiednie moduły. Ta faza je uzupełnia i domyka.

**Szacunek:** 8–12 h

| ID | Zadanie | h | Wynik |
|---|---|---|---|
| F13-T1 | Uzupełnienie testów jednostkowych wg tabeli z sekcji 15.2 | 3 | pokrycie obszarów krytycznych |
| F13-T2 | Testy integracyjne wg tabeli z sekcji 15.3 | 2,5 | testy pipeline'u |
| F13-T3 | Pomiar pokrycia testami; uzupełnienie luk w modułach krytycznych | 1,5 | raport pokrycia |
| F13-T4 | Uruchomienie lintera i formatera na całym kodzie; usunięcie martwego kodu | 2 | kod uporządkowany |
| F13-T5 | Pełny przebieg scenariuszy P1–P10 z protokołem | 1,5 | protokół testów manualnych |
| F13-T6 | Weryfikacja odtwarzalności: powtórzenie wybranego eksperymentu z zapisanej konfiguracji | 1,5 | dowód reprodukowalności |

**DoD fazy:** `pytest` przechodzi w całości; scenariusze P1–P10 udokumentowane; wybrany
eksperyment odtworzony z zapisanej konfiguracji z tym samym wynikiem.

---

### FAZA 14 — Rozszerzenia `[OPCJONALNA]`

**Cel:** funkcje podnoszące wartość projektu, o ile pozostanie czas. **Żadna z nich nie
jest warunkiem ukończenia pracy.**

Kolejność wg stosunku wartości do nakładu — patrz sekcja 21.

**Zasada bezwzględna:** rozszerzenia realizuje się **wyłącznie po** ukończeniu Faz 1–13.
Priorytet 7 z 7 oznacza, że dodatkowa funkcja nie może zdestabilizować działającego systemu.
Każde rozszerzenie realizować na osobnej gałęzi i scalać dopiero po potwierdzeniu, że
scenariusze P1–P10 nadal przechodzą.

---

### FAZA 15 — Finalizacja i materiały do pracy

**Cel:** doprowadzenie repozytorium i materiałów do stanu gotowego do obrony.

**Szacunek:** 12–16 h

| ID | Zadanie | h | Wynik |
|---|---|---|---|
| F15-T1 | Wygenerowanie wszystkich finalnych wykresów i tabel jednym przebiegiem `make_report.py` | 2 | spójne materiały |
| F15-T2 | Weryfikacja: **każda liczba przeznaczona do pracy ma pokrycie w katalogu eksperymentu** | 2 | brak liczb bez źródła |
| F15-T3 | Rysunek architektury systemu do pracy (schemat pipeline'u) | 2 | ilustracja |
| F15-T4 | Uzupełnienie `README.md` — instrukcja instalacji i uruchomienia od zera | 1,5 | dokumentacja |
| F15-T5 | Weryfikacja instrukcji: instalacja na czystym środowisku wg własnego README | 2 | dowód kompletności |
| F15-T6 | `docs/THESIS_OUTLINE.md` — mapowanie wyników projektu na rozdziały pracy | 1,5 | plan pisania |
| F15-T7 | Uporządkowanie repozytorium: usunięcie plików roboczych, sprawdzenie `.gitignore` | 1 | czyste repozytorium |
| F15-T8 | Zestawienie ograniczeń systemu na podstawie wyników E10 — do rozdziału o ograniczeniach | 1,5 | uczciwa ocena rozwiązania |
| F15-T9 | Zestawienie kierunków rozwoju na podstawie napotkanych problemów | 1 | rozdział o rozwoju |

**DoD projektu:** działający system, komplet wykonanych eksperymentów, wszystkie materiały
do pracy wygenerowane i mające pokrycie w zapisanych przebiegach, repozytorium
zainstalowane od zera według własnej instrukcji.

---

## 18. Ryzyka

| # | Ryzyko | Prawdop. | Skutek | Zapobieganie i reakcja |
|---|---|---|---|---|
| R1 | Brak zgodności Pythona 3.14 ze stosem PyTorch | **wysokie** | blokada startu | osobne środowisko na 3.12 (`F1-T1`); w razie problemów 3.11 |
| R2 | Przeciek danych w publicznym zbiorze (duplikaty, augmentowane kopie) | **wysokie** | **wyniki w pracy zawyżone i nieprawdziwe** | obowiązkowy audyt pHash (`F2-T4`, `F2-T8`) — kryterium przejścia fazy |
| R3 | Duży domain gap między danymi publicznymi a webcamem | **wysokie** | słabe wyniki w rzeczywistym użyciu | zaplanowane jako eksperyment E11; dostrojenie na własnych danych; opisać uczciwie w ograniczeniach |
| R4 | Nierówne tempo pracy — utrata kontekstu po przerwie | **wysokie** | powolny restart, błędy | `docs/JOURNAL.md` z polem „następny krok"; zadania po 1–4 h; fazy jako niezależne bloki |
| R5 | Zbiór publiczny okazuje się nieprzydatny lub o niejasnej licencji | średnie | opóźnienie 1–2 tygodni | kilku kandydatów w `F2-T1`; awaryjnie budowa wycinków ze zbioru detekcyjnego lub przyspieszenie Fazy 7 |
| R6 | Nieosiągnięcie celu 15 FPS | średnie | cel C7 niespełniony | model MobileNet, niższa rozdzielczość, klasyfikacja wsadowa; ostatecznie eksport do ONNX (rozszerzenie X4); udokumentować rzeczywisty FPS zamiast deklarować cel |
| R7 | Rozrost zakresu — GUI i rozszerzenia kosztem eksperymentów | średnie | słaba część badawcza pracy | sztywny priorytet: F1–F10 przed F11; rozszerzenia dopiero po F13 |
| R8 | Karty mylone systematycznie (♠↔♣, 6↔9) | średnie | obniżona dokładność | wyższa rozdzielczość (E8), analiza macierzy pomyłek, ewentualnie ważenie klas; **wynik wart opisania niezależnie od skuteczności naprawy** |
| R9 | Błąd w mapowaniu etykiet | niskie | **unieważnienie wszystkich wyników** | testy jednostkowe `labels.py` jako pierwsze zadanie kodowe (`F1-T8`) |
| R10 | Awaria sprzętu | niskie | utrata pracy | repozytorium zdalne, regularne commity, kopia zapasowa `data/own/` |
| R11 | Trening YOLO nie daje zadowalających wyników | niskie | brak detektora uczonego | detektor konturowy z Fazy 3 pozostaje sprawnym rozwiązaniem awaryjnym; różnica staje się treścią E7 |
| R12 | Problem licencyjny AGPL (Ultralytics) | niskie | konieczność zmiany detektora | licencja odnotowana w pracy; moduł detekcji wymienny; awaryjnie detektor konturowy lub własna implementacja |
| R13 | Przeuczenie na jednej talii | średnie | model nie generalizuje | jeśli w posiadaniu jest druga talia o odmiennym kroju (`F1-T6`) — użyć jej wyłącznie w zbiorze testowym; w przeciwnym razie ograniczenie opisane wprost w pracy |
| R14 | Wyniki eksperymentów nieistotne statystycznie | średnie | brak mocnych wniosków | 3 ziarna i odchylenie standardowe; „różnica nieistotna" jest poprawnym wnioskiem |

**Trzy ryzyka wymagające szczególnej uwagi:** R2 (przeciek danych) i R9 (mapowanie etykiet)
są groźne, bo **nie powodują żadnego widocznego błędu** — dają po prostu nieprawdziwe
liczby w pracy. R4 (nierówne tempo) jest największym realnym zagrożeniem dla ukończenia
projektu w terminie.

---

## 19. Powiązanie z pracą inżynierską

### 19.1 Uwagi do wstępnego spisu treści pracy

Wstępny spis treści pracy ma 16 rozdziałów, co przy 40–60 stronach daje średnio 3 strony na
rozdział — zbyt rozdrobniona. Ponadto rozdziały 2–7 (sztuczna inteligencja, computer vision,
klasyfikacja, detekcja, YOLO, CNN) to sześć osobnych rozdziałów teoretycznych, które
w praktyce tworzą jeden blok. Poniżej propozycja scalona, z zachowaniem całej treści.

### 19.2 Proponowana struktura pracy

| Rozdział | Treść | Strony | Źródło materiału |
|---|---|---|---|
| **1. Wstęp** | problem, motywacja, cel, zakres, struktura pracy | 3–4 | sekcje 1, 2 |
| **2. Podstawy teoretyczne** | 2.1 uczenie maszynowe i głębokie; 2.2 widzenie komputerowe; 2.3 sieci konwolucyjne; 2.4 klasyfikacja obrazów; 2.5 detekcja obiektów i rodzina YOLO; 2.6 transfer learning; 2.7 augmentacja danych; 2.8 metryki | 12–15 | sekcje 6, 8, 10 + literatura |
| **3. Analiza problemu i przegląd rozwiązań** | specyfika rozpoznawania kart, istniejące podejścia, klasyczne CV kontra uczenie maszynowe | 5–6 | sekcje 1.2, 1.3 |
| **4. Projekt systemu** | wymagania, **porównanie trzech architektur i uzasadnienie wyboru**, pipeline, dobór technologii, struktura kodu | 8–10 | sekcje 3, 4, 12, 13 |
| **5. Dane** | strategia trójźródłowa, **audyt zbioru**, dane syntetyczne, podziały i zapobieganie przeciekowi, polityka augmentacji | 7–9 | sekcje 5, 6, `docs/DATASET.md` |
| **6. Implementacja** | moduły systemu, korekcja perspektywy, klasyfikatory, kalibracja, aplikacja i wielowątkowość | 8–10 | sekcje 7–9, 12 |
| **7. Eksperymenty i wyniki** | metodyka, opis i wyniki E1–E12 | 12–15 | sekcja 11, `docs/EXPERIMENTS.md` |
| **8. Analiza wyników** | macierz pomyłek, analiza błędów, odporność, **domain gap** | 5–7 | sekcje 10.4, E10, E11 |
| **9. Ograniczenia i kierunki rozwoju** | granice stosowalności wynikające z pomiarów, propozycje rozwinięcia | 3–4 | E10b, sekcja 21 |
| **10. Podsumowanie** | realizacja celów, wnioski, wkład własny | 2–3 | sekcja 2 |
| Bibliografia, załączniki | pełna macierz 52×52, listingi, zrzuty ekranu | — | `experiments/` |
| **Razem** | | **65–83** | |

Zakres wychodzi powyżej dolnej granicy 40–60 stron, co daje swobodę skracania —
lepsza sytuacja niż konieczność sztucznego rozciągania tekstu.

**Uzasadnienie zmian względem wstępnego spisu treści:**
- rozdziały 2–7 ze wstępnego spisu treści scalone w jeden rozdział teoretyczny z podrozdziałami: unika
  powtórzeń (transfer learning trudno opisać w oderwaniu od CNN),
- dodany rozdział **„Analiza problemu i przegląd rozwiązań"**: wymagany element pracy
  inżynierskiej, nieobecny we wstępnym spisie treści,
- rozdział **„Dane"** wyodrębniony i rozbudowany — audyt zbioru i zapobieganie przeciekowi
  to najmocniejszy metodologicznie fragment projektu i zasługuje na osobny rozdział,
- „Wyniki" i „Analiza wyników" rozdzielone: pierwszy przedstawia liczby, drugi je interpretuje.

### 19.3 Mapowanie: co z projektu trafia do której części pracy

| Element projektu | Gdzie w pracy |
|---|---|
| Porównanie trzech architektur (sekcja 3) + E6 | rozdz. 4 — **kluczowe uzasadnienie decyzji projektowej** |
| Porównanie technologii (sekcja 4) | rozdz. 4 |
| Audyt zbioru, `docs/DATASET.md` | rozdz. 5 — pokazuje warsztat metodologiczny |
| Analiza augmentacji, w szczególności odbicia lustrzanego (sekcja 6.2) + E4 | rozdz. 5 i 7 — **treść oryginalna, nie odtwórcza** |
| Korekcja perspektywy (sekcja 7) + E5 | rozdz. 6 i 7 |
| Architektury klasyfikatorów + E1, E2 | rozdz. 6 i 7 |
| Dekompozycja 52 / 13+4 / multi-head + E3 | rozdz. 7 — **jeden z ciekawszych wyników** |
| Kalibracja i wyznaczenie progu + E9 | rozdz. 6 i 7 — pokazuje dojrzałość metodologiczną |
| Detektor konturowy kontra YOLO + E7 | rozdz. 3 i 7 — uzasadnia sens stosowania AI |
| Metryka end-to-end (sekcja 10.3) | rozdz. 7 — **główna liczba w podsumowaniu** |
| Macierze pomyłek | rozdz. 8 + załącznik |
| E10a/b/c | rozdz. 8 i 9 |
| E11 domain gap | rozdz. 8 — **najbardziej wartościowy wynik przy przyjętej strategii danych** |
| Aplikacja i wielowątkowość (sekcja 12.4) | rozdz. 6 |
| Testy (sekcja 15) | rozdz. 6 |
| Wymagania sprzętowe i FPS | rozdz. 7 i 9 |

### 19.4 Zasada dotycząca liczb w pracy

Każda liczba w tekście pracy musi mieć pokrycie w katalogu `experiments/<ID>/`. Zadanie
`F15-T2` weryfikuje to jawnie. Żadna wartość z tego dokumentu nie może trafić do pracy —
wszystkie oznaczone jako `[DO ZMIERZENIA]` są w nim wyłącznie miejscami do wypełnienia.

---

## 20. Zakres MVP

**MVP to zakres, poniżej którego praca nie jest obronna.** Odpowiada Fazom 1–6 plus
minimalny zestaw eksperymentów.

### 20.1 Co MUSI działać

| # | Wymaganie | Faza |
|---|---|---|
| 1 | Wczytanie obrazu z pliku | F3 |
| 2 | Wykrycie kart na obrazie (dowolną z dwóch metod) | F3 / F5 |
| 3 | Wycięcie i normalizacja geometryczna kart | F3 |
| 4 | Klasyfikacja do 52 klas z podaniem pewności | F4 |
| 5 | Odrzucenie klasyfikacji poniżej progu | F8 |
| 6 | Wynik jako obraz z ramkami oraz plik JSON/CSV | F6 |
| 7 | Obsługa wielu kart w jednym kadrze | F6 |
| 8 | Zmierzone metryki: dokładność klasyfikacji, mAP detekcji, metryka end-to-end | F4, F5, F10 |
| 9 | Macierz pomyłek | F4 |
| 10 | Reprodukowalność: zapisana konfiguracja i ziarno dla każdego przebiegu | F1 |
| 11 | Minimum sześć eksperymentów: E1, E2, E3, E4, E5, E9 | F9, F10 |
| 12 | Testy jednostkowe modułów krytycznych (`labels`, geometria, metryki) | F1, F3, F4 |

### 20.2 Co jest ważne, ale nie krytyczne

| # | Element | Uzasadnienie |
|---|---|---|
| 13 | GUI (F11) | podnosi wartość prezentacyjną; CLI wystarcza do obrony |
| 14 | Tryb kamery (F12) | wymieniony w celach, ale nie warunkuje wartości naukowej |
| 15 | Własny zbiór testowy (F7) | **mocno rekomendowany** — bez niego praca jest znacznie słabsza, ale formalnie da się ją obronić na danych publicznych |
| 16 | Eksperymenty E6, E7, E8, E10, E11, E12 | podnoszą jakość części badawczej |

### 20.3 Ścieżka minimalna przy braku czasu

Gdyby okazało się, że czasu jest istotnie mniej niż zakładano, kolejność rezygnacji:

1. najpierw odpada Faza 14 (rozszerzenia),
2. następnie Faza 12 (tryb kamery),
3. następnie Faza 11 ograniczona do samego trybu obrazu,
4. następnie eksperymenty E12, E10c, E8,
5. **nigdy nie rezygnować z:** audytu danych (F2), poprawnych podziałów, testów modułów
   krytycznych i eksperymentów E1–E5 oraz E9.

Punkt 5 wynika wprost z listy priorytetów: poprawność działania i możliwość przeprowadzenia
eksperymentów stoją wyżej niż dodatkowe funkcje.

---

## 21. Rozszerzenia

Uporządkowane według stosunku wartości do nakładu. Identyfikatory `X…` odróżniają
rozszerzenia od ryzyk `R…` z sekcji 18.

| ID | Rozszerzenie | Nakład | Wartość | Opis |
|---|---|---|---|---|
| **X1** | **Analiza pokrycia talii** | 2–3 h | **wysoka** | zliczanie rozpoznanych kart, prezentacja „4 / 52", lista kart brakujących, wykrywanie duplikatów (ta sama karta rozpoznana dwukrotnie sygnalizuje błąd). Najlepszy stosunek wartości do nakładu — mała funkcja, efektowna prezentacja, dodatkowe kryterium poprawności systemu |
| **X2** | **Głosowanie po klatkach w trybie wideo** | 4–6 h | **wysoka** | proste śledzenie kart między klatkami (dopasowanie po IoU) i agregacja predykcji z ostatnich N klatek. Znacząco stabilizuje wynik w trybie kamery, eliminuje „migotanie" etykiet i pozwala klasyfikować tylko nowe obiekty — czyli podnosi też FPS. **Sam w sobie nadaje się na eksperyment:** wpływ agregacji czasowej na dokładność |
| X3 | Eksperyment z talią o odmiennym kroju | 3–4 h | wysoka | trening na talii A, test na talii B — pomiar generalizacji międzytaliowej; wykonalny, jeśli weryfikacja `F1-T6` wykaże drugą talię o odmiennym kroju |
| X4 | Eksport do ONNX i pomiar przyspieszenia | 4–6 h | średnia | konieczne tylko wtedy, gdy nie uda się osiągnąć celu FPS; daje dodatkowy wynik wydajnościowy |
| X5 | Wizualizacja map istotności (Grad-CAM) | 3–4 h | średnia | pokazuje, na które fragmenty karty patrzy sieć. Weryfikuje hipotezę z sekcji 1.2a, że decyduje indeks narożny — bardzo dobra ilustracja do rozdziału o analizie wyników |
| X6 | Wykrywanie rewersów jako osobnej klasy | 2–3 h | średnia | rozszerza zakres systemu o sytuację obecnie odrzucaną |
| X7 | Tryb wsadowy z raportem PDF | 3–4 h | niska | wygodne, ale bez wartości naukowej |
| X8 | Rozpoznawanie układów pokerowych | 4–6 h | niska | efektowne w prezentacji, ale to logika gry, a nie widzenie komputerowe — poza tematem pracy |

**Rekomendacja:** jeśli czas pozwoli na dwa rozszerzenia, wybrać **X1 i X2** — oba
bezpośrednio poprawiają działanie systemu, a X2 dodatkowo generuje wynik eksperymentalny.
Jako trzeci wybrać **X5**, jeśli rozdział o analizie wyników wymaga wzmocnienia.

---

## 22. Rekomendowany plan i pierwszy krok

### 22.1 REKOMENDOWANY PLAN — podsumowanie

**Architektura:** dwuetapowa — YOLO jednoklasowy jako detektor, osobny klasyfikator CNN
dla 52 klas, z opcjonalną korekcją perspektywy między etapami. Architektura jednoetapowa
(YOLO 52-klasowy) i detektor konturowy pozostają w projekcie jako punkty odniesienia
mierzone w E6 i E7 — dzięki temu wybór architektury zostaje uzasadniony pomiarem.

**Technologie:** Python 3.12 (nie 3.14), PyTorch, Ultralytics YOLO wariant `n`, OpenCV,
PySide6, Albumentations, scikit-learn, Matplotlib, pytest. Każda pozycja uzasadniona
w sekcji 4; świadomie odrzucone: Lightning, W&B, Hydra, DVC, Docker.

**Dane:** strategia trójźródłowa — publiczne zbiory do treningu (~80%), dane syntetyczne
do rozszerzenia zbioru detekcyjnego (~15%), **własne zdjęcia wyłącznie jako zbiór testowy**
(~5%). Obowiązkowy audyt danych przed treningiem. Podział rozłączny na poziomie sesji.

**Modele:** SimpleCNN, ResNet18, MobileNetV3-Small; trzy warianty struktury wyjścia
(52 klasy, dwa modele 13+4, multi-head).

**Eksperymenty:** dwanaście (E1–E12), z czego sześć krytycznych (E1–E5, E9) wykonalnych
bez własnego zbioru danych.

**Fazy:** 15 faz, około 200–250 h nakładu, zadania po 1–4 h. **Kamień milowy w Fazie 6** —
po niej istnieje działający system end-to-end.

**Trzy elementy, które najbardziej odróżniają ten plan od typowego projektu studenckiego:**
1. audyt danych z wykrywaniem przecieku (sekcja 5.3) — chroni przed nieprawdziwymi wynikami,
2. próg pewności wyznaczony z krzywej ryzyko–pokrycie (sekcja 9.4) zamiast wartości przyjętej arbitralnie,
3. pomiar domain gap (E11) — zamienia ograniczenie strategii danych w wynik naukowy.

### 22.2 PIERWSZY KROK DO WYKONANIA

**Faza 1, zadania F1-T1 do F1-T3 — przygotowanie środowiska.**

Zanim powstanie pierwsza linia kodu, trzeba rozwiązać problem wykryty na maszynie
deweloperskiej: zainstalowany jest Python 3.14.7, dla którego stos PyTorch/Ultralytics
najprawdopodobniej nie ma gotowych pakietów binarnych.

**Do wykonania:**

1. **Instalacja Pythona 3.12** ze strony python.org (wariant Windows installer 64-bit).
   W instalatorze:
   - opcja „Add python.exe to PATH" pozostaje **odznaczona**, żeby nie nadpisać
     istniejącej instalacji 3.14,
   - należy zapisać ścieżkę instalacji.

2. **Weryfikacja instalacji** — w terminalu w katalogu projektu:
   ```
   py -0
   ```
   Na liście powinny pojawić się obie wersje: 3.14 i 3.12.

3. **Środowisko wirtualne i PyTorch** — utworzenie `.venv` na Pythonie 3.12, instalacja
   PyTorcha z wersją CUDA odpowiednią dla RTX 3060 Ti oraz sprawdzenie, że GPU jest
   widoczne (`torch.cuda.is_available()`).

Równolegle, niezależnie od powyższego — **weryfikacja posiadanych talii** (`F1-T6`) według
listy kontrolnej z sekcji 5.7: rozmiar, krój indeksów, liczba kolorów indeksów i symetria
względem obrotu o 180°. Same talie będą potrzebne dopiero w Fazie 7, ale ich opis jest
potrzebny wcześniej — przesądza o polityce augmentacji przyjętej w Fazie 4.

### 22.3 Zasady dalszej pracy

- Fazy realizowane są po kolei — przejście do kolejnej następuje dopiero wtedy, gdy
  kryterium przejścia poprzedniej jest spełnione.
- Każde zadanie ma określony rezultat: ścieżkę pliku, komendę uruchomieniową, oczekiwany
  wynik i sposób sprawdzenia.
- Po każdej sesji pracy uzupełniany jest `docs/JOURNAL.md` — przy nierównym tempie pracy
  to właśnie on pozwala wrócić do projektu po przerwie bez odtwarzania kontekstu.
- Zmiany w architekturze, doborze technologii lub strategii danych są odnotowywane
  w tym dokumencie wraz z uzasadnieniem.

### 22.4 Kwestie otwarte

Nie blokują startu Fazy 1, ale wymagają rozstrzygnięcia:

1. **Termin obrony pracy.** Plan zakłada około pół roku. Znając datę, można przypisać
   fazom konkretne okna czasowe i wcześniej wykryć opóźnienie.
2. **Wymagania promotora** co do struktury pracy lub liczby stron. Struktura z sekcji 19.2
   jest propozycją i można ją dostosować.
3. **Kamera internetowa**, na której docelowo ma działać system. Wpływa to na Fazę 12
   i na wybór sprzętu do własnych nagrań w Fazie 7.
4. **Repozytorium zdalne** (GitHub) — rekomendowane jako kopia zapasowa (ryzyko R10).
   Wymaga decyzji, czy repozytorium ma być publiczne czy prywatne.

---

*Dokument przygotowany w Fazie 0. Wszystkie wartości liczbowe oznaczone jako
`[DO ZMIERZENIA]` zostaną uzupełnione wynikami rzeczywistych eksperymentów.
Żadna liczba z tego dokumentu nie może zostać użyta w pracy inżynierskiej jako wynik.*
