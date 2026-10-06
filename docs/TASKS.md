# TASKS.md — rozpiska zadań projektu

**Projekt:** „Zastosowanie narzędzi sztucznej inteligencji do klasyfikacji kart do gry"
**Dokument nadrzędny:** [`PROJECT_PLAN.md`](PROJECT_PLAN.md)
**Wersja:** 1.0 (2026-10-06)

---

## Po co ten dokument

`PROJECT_PLAN.md` odpowiada na pytanie **dlaczego** — uzasadnia architekturę, dobór danych,
modeli i eksperymentów. Ten dokument odpowiada na pytanie **co dokładnie zrobić i kiedy
uznać to za zrobione**. Jest punktem odniesienia przy każdej sesji pracy:

1. otworzyć ten plik i znaleźć pierwsze nieodhaczone zadanie w bieżącej fazie,
2. wykonać kroki zadania,
3. sprawdzić warunek „Gotowe, gdy",
4. odhaczyć zadanie na liście postępu fazy,
5. dopisać wpis w `docs/JOURNAL.md` (co zrobiono, co nie działa, następny krok),
6. zrobić commit.

Identyfikatory zadań (`Fx-Ty`) są **identyczne z identyfikatorami w `PROJECT_PLAN.md`** —
odwołania między dokumentami pozostają ważne. Numery sekcji w nawiasach (np. „sekcja 5.3")
odnoszą się do `PROJECT_PLAN.md`.

### Format opisu zadania

| Pole | Znaczenie |
|---|---|
| **Cel** | po co to zadanie istnieje — jedno zdanie |
| **Kroki** | konkretne czynności w kolejności wykonania |
| **Rezultat** | pliki lub artefakty, które mają powstać |
| **Gotowe, gdy** | sprawdzalny warunek ukończenia (Definition of Done) |
| **Zależy od** | zadania, które muszą być ukończone wcześniej |

Szacunek czasu podany jest przy nazwie zadania. Są to **szacunki nakładu pracy, nie
pomiary** — do planowania sesji, nie do raportowania.

### Oznaczenia statusu

- `[ ]` — nierozpoczęte
- `[~]` — w toku (warto dopisać w `JOURNAL.md`, na czym stanęło)
- `[x]` — ukończone i zweryfikowane

---

## Przegląd faz

| Faza | Temat | Szacunek | Status | Uwagi |
|---|---|---|---|---|
| F0 | Analiza wymagań i plan | — | **ukończona** | `PROJECT_PLAN.md`, `README.md` |
| F1 | Środowisko i szkielet projektu | 8–10 h | nierozpoczęta | **następna do realizacji** |
| F2 | Dane: pozyskanie i audyt | 10–14 h | nierozpoczęta | kryterium: zero kolizji pHash |
| F3 | Preprocessing, geometria, detektor klasyczny | 12–16 h | nierozpoczęta | |
| F4 | Klasyfikator kart | 14–18 h | nierozpoczęta | rdzeń tematu pracy |
| F5 | Detektor YOLO | 12–16 h | nierozpoczęta | |
| **F6** | **Integracja end-to-end** | 10–12 h | nierozpoczęta | **kamień milowy: MVP** |
| F7 | Własny zbiór testowy | 10–14 h | nierozpoczęta | wymaga talii z `F1-T6` |
| F8 | Confidence, kalibracja, obsługa błędów | 10–12 h | nierozpoczęta | |
| F9 | Eksperymenty klasyfikacji (E1–E4, E8) | 20–26 h | nierozpoczęta | |
| F10 | Eksperymenty systemowe (E5–E7, E9–E12) | 20–26 h | nierozpoczęta | |
| F11 | Aplikacja GUI | 16–20 h | nierozpoczęta | |
| F12 | Tryb kamery i czas rzeczywisty | 10–14 h | nierozpoczęta | |
| F13 | Testy i jakość kodu | 8–12 h | nierozpoczęta | testy krytyczne powstają wcześniej |
| F14 | Rozszerzenia | 0–20 h | nierozpoczęta | **opcjonalna** |
| F15 | Finalizacja i materiały do pracy | 12–16 h | nierozpoczęta | |

### Kolejność i zależności

```
F0 ─> F1 ─> F2 ─> F3 ─┬─> F4 ──┐
                      └─> F5 ──┴─> F6 (MVP) ─┬─> F7 ──────────────┐
                                             ├─> F8 ──> F9 ──> F10 <┘
                                             └─> F11 ──> F12
F13 — testy krytyczne powstają w F1, F3, F4, F5, F8; F13 je domyka po F12
F14 — tylko po F13
F15 — na końcu
```

**Zasada:** do Fazy 6 dochodzi się bez zbaczania na boki. Po F6 projekt jest obronny;
wszystko dalsze podnosi jego wartość.

### Mapowanie eksperymentów na zadania

| Eksperyment | Temat | Zadanie | Wymaga własnych danych (F7) |
|---|---|---|---|
| E1 | architektury klasyfikatora | `F9-T3` | nie |
| E2 | transfer learning | `F9-T4` | nie |
| E3 | 52 klasy / 13+4 / multi-head | `F9-T5` | nie |
| E4 | augmentacja | `F9-T6` | częściowo (seria `test_own`) |
| E5 | korekcja perspektywy | `F10-T2` | tak (rozbicie na kąty) |
| E6 | jedno- vs dwuetapowa | `F10-T4`, `F10-T5` | tak |
| E7 | detektor konturowy vs YOLO | `F10-T3` | tak |
| E8 | rozdzielczość wejścia | `F9-T7` | nie |
| E9 | kalibracja i próg | `F8-T1`…`F8-T5`, `F10-T6` | nie |
| E10a/b/c | odporność | `F10-T7` | tak |
| E11 | domain gap | `F10-T8` | tak |
| E12 | dane syntetyczne vs rzeczywiste | `F10-T9` | tak |

Eksperymenty krytyczne (minimum do obrony): **E1, E2, E3, E4, E5, E9**.

---

## FAZA 0 — Analiza wymagań i plan `[UKOŃCZONA]`

**Cel fazy:** ustalić zakres, architekturę i metodologię przed napisaniem pierwszej linii
kodu, tak aby każda późniejsza decyzja miała punkt odniesienia.

**Rezultat fazy:** `docs/PROJECT_PLAN.md`, `README.md`, repozytorium git z pierwszymi
commitami.

### Postęp

- [x] F0-T1 — Analiza problemu i jego trudności
- [x] F0-T2 — Sformułowanie celów i granic projektu
- [x] F0-T3 — Analiza i wybór architektury systemu
- [x] F0-T4 — Dobór technologii
- [x] F0-T5 — Rozpoznanie sprzętu i środowiska
- [x] F0-T6 — Strategia danych
- [x] F0-T7 — Polityka augmentacji i koncepcja korekcji perspektywy
- [x] F0-T8 — Projekt modeli, metryk i eksperymentów
- [x] F0-T9 — Projekt aplikacji i struktury repozytorium
- [x] F0-T10 — Podział na fazy, analiza ryzyk, zakres MVP
- [x] F0-T11 — README i założenie repozytorium

### Zadania

#### F0-T1 — Analiza problemu i jego trudności
- **Cel:** zrozumieć, co czyni rozpoznawanie kart nietrywialnym.
- **Kroki:** opis zadania; identyfikacja trudności (mały indeks narożny, pary podobne
  `6`/`9`, `♠`/`♣`, perspektywa, oświetlenie, zasłonięcia); uzasadnienie, dlaczego jest to
  problem dla uczenia maszynowego, a nie tylko dla klasycznego CV; zastosowania praktyczne.
- **Rezultat:** sekcja 1 planu.
- **Gotowe, gdy:** każda trudność ma wskazane miejsce w projekcie, które ją adresuje.

#### F0-T2 — Sformułowanie celów i granic projektu
- **Cel:** jednoznacznie określić, co system ma robić i czego nie obejmuje.
- **Kroki:** cel główny; cele inżynierskie C1–C9, badawcze B1–B8, metodologiczne M1–M4;
  lista wyłączeń (rewersy, jokery, logika gry, urządzenia mobilne).
- **Rezultat:** sekcja 2 planu.
- **Gotowe, gdy:** każdy cel ma wskazaną fazę lub eksperyment, w którym jest weryfikowany.

#### F0-T3 — Analiza i wybór architektury systemu
- **Cel:** wybrać architekturę w sposób uzasadniony, z możliwością weryfikacji pomiarem.
- **Kroki:** opis trzech wariantów (A — YOLO 52 klasy, B — detektor + klasyfikator,
  C — narożniki/punkty kluczowe); tabela porównawcza; decyzja: B jako główna, A jako
  baseline (E6), C jako opisana alternatywa; docelowy pipeline z interfejsami wymiennymi.
- **Rezultat:** sekcja 3 planu.
- **Gotowe, gdy:** decyzja ma uzasadnienie i zaplanowany eksperyment ją weryfikujący (E6).

#### F0-T4 — Dobór technologii
- **Cel:** wybrać minimalny stos, w którym każda pozycja ma uzasadnienie.
- **Kroki:** Python 3.12, PyTorch, Ultralytics YOLO (wariant `n`), OpenCV, PySide6,
  Albumentations, scikit-learn, Matplotlib, pytest; lista technologii świadomie odrzuconych
  (Lightning, W&B, Hydra, DVC, Docker) z powodem.
- **Rezultat:** sekcja 4 planu.
- **Gotowe, gdy:** każda biblioteka ma przypisaną rolę w systemie.

#### F0-T5 — Rozpoznanie sprzętu i środowiska
- **Cel:** potwierdzić, że projekt da się zrealizować lokalnie.
- **Kroki:** spis sprzętu (i5-12600KF, RTX 3060 Ti 8 GB, 16 GB RAM, Windows 11); wykrycie
  problemu z Pythonem 3.14; szacunek zapotrzebowania na VRAM i dysk; wariant awaryjny bez GPU.
- **Rezultat:** sekcja 16 planu, ryzyko R1.
- **Gotowe, gdy:** wiadomo, co trzeba przygotować w F1 (Python 3.12 obok 3.14).

#### F0-T6 — Strategia danych
- **Cel:** zaplanować dane tak, by wyniki były wiarygodne, a nakład realny.
- **Kroki:** strategia trójźródłowa (publiczne ~80%, syntetyczne ~15%, własne ~5% tylko do
  testu); kryteria wyboru zbiorów; zakres audytu; zasady podziału rozłącznego na poziomie
  sesji; metoda zbierania danych własnych z nagrań wideo; lista kontrolna talii.
- **Rezultat:** sekcja 5 planu.
- **Gotowe, gdy:** każde źródło danych ma przypisaną rolę i zabezpieczenie przed przeciekiem.

#### F0-T7 — Polityka augmentacji i koncepcja korekcji perspektywy
- **Cel:** z góry ustalić, które transformacje są poprawne dla kart, a które szkodliwe.
- **Kroki:** analiza każdej transformacji (obrót 180° — tak, odbicie lustrzane — nie,
  hue — nie lub minimalnie itd.); opis prostowania homografią ze ścieżką awaryjną.
- **Rezultat:** sekcje 6 i 7 planu.
- **Gotowe, gdy:** każda decyzja ma uzasadnienie i jest weryfikowana eksperymentem (E4, E5).

#### F0-T8 — Projekt modeli, metryk i eksperymentów
- **Cel:** zaprojektować część badawczą pracy przed rozpoczęciem implementacji.
- **Kroki:** modele (SimpleCNN, ResNet18, MobileNetV3-Small, MultiHead); procedura
  kalibracji i wyboru progu; metryki detekcji, klasyfikacji i end-to-end; katalog
  eksperymentów E1–E12 z hipotezami i zmiennymi; priorytetyzacja.
- **Rezultat:** sekcje 8–11 planu.
- **Gotowe, gdy:** każdy eksperyment ma hipotezę, zmienną niezależną, zmienne zależne
  i sposób prezentacji.

#### F0-T9 — Projekt aplikacji i struktury repozytorium
- **Cel:** wiedzieć z góry, gdzie trafi każdy fragment kodu.
- **Kroki:** scenariusze użycia; układ GUI; architektura wątków; struktura katalogów
  (`src-layout`); zasady architektoniczne; reprodukowalność; strategia testów.
- **Rezultat:** sekcje 12–15 planu.
- **Gotowe, gdy:** struktura z sekcji 13.2 obejmuje wszystkie moduły pipeline'u.

#### F0-T10 — Podział na fazy, analiza ryzyk, zakres MVP
- **Cel:** zamienić projekt w sekwencję wykonalnych kroków odpornych na nierówne tempo pracy.
- **Kroki:** 15 faz z zadaniami po 1–4 h; mapa zależności; budżet czasu; ryzyka R1–R14;
  zakres MVP i kolejność rezygnacji; mapowanie projektu na rozdziały pracy.
- **Rezultat:** sekcje 17–22 planu.
- **Gotowe, gdy:** po Fazie 6 istnieje obronny system, a każda faza kończy się artefaktem.

#### F0-T11 — README i założenie repozytorium
- **Cel:** mieć publiczną „wizytówkę" projektu i historię zmian od pierwszego dnia.
- **Kroki:** `git init`; `README.md` z opisem projektu, celów i stanu; commit planu.
- **Rezultat:** commity `71e880a` i `2c84d45`.
- **Gotowe, gdy:** plan i README są w repozytorium.

**Kryterium przejścia:** architektura, technologie i strategia danych ustalone — spełnione.

---

## FAZA 1 — Środowisko i szkielet projektu

**Cel fazy:** działające, odtwarzalne środowisko na GPU oraz szkielet repozytorium, w którym
można pisać kod bez późniejszej reorganizacji.

**Wejście:** ukończona F0. Zainstalowany sterownik NVIDIA.

**Szacunek:** 8–10 h

### Postęp

- [ ] F1-T1 — Python 3.12 i środowisko wirtualne
- [ ] F1-T2 — PyTorch z obsługą CUDA
- [ ] F1-T3 — Pozostałe zależności i `requirements.txt`
- [ ] F1-T4 — Struktura katalogów i `.gitignore`
- [ ] F1-T5 — Pakiet instalowalny (`pyproject.toml`)
- [ ] F1-T6 — Weryfikacja posiadanych talii
- [ ] F1-T7 — Moduł etykiet `labels.py`
- [ ] F1-T8 — Testy jednostkowe `labels.py`
- [ ] F1-T9 — Ziarno losowe i katalog przebiegu
- [ ] F1-T10 — Konfiguracja `pytest` i dziennik prac
- [ ] F1-T11 — Pierwszy commit kodu i repozytorium zdalne

### Zadania

#### F1-T1 — Python 3.12 i środowisko wirtualne · 1 h
- **Cel:** usunąć blokadę R1 — stos PyTorch nie wspiera Pythona 3.14.
- **Kroki:**
  1. Pobrać instalator Python 3.12 (Windows, 64-bit) z python.org.
  2. Zainstalować **bez** opcji „Add python.exe to PATH", żeby nie naruszyć instalacji 3.14.
  3. Sprawdzić `py -0` — lista musi zawierać 3.12 i 3.14.
  4. W katalogu projektu: `py -3.12 -m venv .venv`.
  5. Aktywować: `.venv\Scripts\Activate.ps1`; zaktualizować `pip`.
- **Rezultat:** katalog `.venv/` z interpreterem 3.12.
- **Gotowe, gdy:** `python --version` w aktywnym środowisku zwraca 3.12.x.
- **Zależy od:** —

#### F1-T2 — PyTorch z obsługą CUDA · 1 h
- **Cel:** trening na GPU od pierwszego eksperymentu.
- **Kroki:**
  1. `nvidia-smi` — odczytać wersję sterownika i maksymalną obsługiwaną wersję CUDA.
  2. Dobrać wariant PyTorcha (CUDA) zgodny ze sterownikiem na pytorch.org.
  3. Zainstalować `torch` i `torchvision` z odpowiedniego indeksu.
  4. Sprawdzić `torch.cuda.is_available()` i `torch.cuda.get_device_name(0)`.
- **Rezultat:** PyTorch widzi RTX 3060 Ti.
- **Gotowe, gdy:** `python -c "import torch; print(torch.cuda.is_available())"` → `True`.
- **Zależy od:** F1-T1
- **Jeśli się nie uda:** Python 3.11; w ostateczności wariant CPU do czasu naprawy.

#### F1-T3 — Pozostałe zależności i `requirements.txt` · 1 h
- **Cel:** odtwarzalne środowisko na dowolnej maszynie.
- **Kroki:**
  1. Zainstalować: `ultralytics`, `opencv-python`, `numpy`, `pandas`, `scikit-learn`,
     `matplotlib`, `albumentations`, `pyyaml`, `imagehash`, `pyside6`, `pytest`.
  2. Sprawdzić, że `ultralytics` nie nadpisało `torch` wariantem CPU.
  3. Zapisać `requirements.txt` z **przypiętymi wersjami** (`==`), z komentarzem, z jakiego
     indeksu instalować PyTorch.
- **Rezultat:** `requirements.txt`.
- **Gotowe, gdy:** wszystkie pakiety importują się bez błędów; `torch.cuda.is_available()`
  nadal zwraca `True`.
- **Zależy od:** F1-T2

#### F1-T4 — Struktura katalogów i `.gitignore` · 1 h
- **Cel:** szkielet zgodny z sekcją 13.2, żeby każdy plik od razu trafiał na swoje miejsce.
- **Kroki:**
  1. Utworzyć katalogi: `configs/` (z podkatalogami), `data/` (z podkatalogami), `experiments/`,
     `models/`, `notebooks/`, `results/`, `scripts/`, `src/cardvision/` (z podpakietami
     i plikami `__init__.py`), `tests/{unit,integration,fixtures}/`.
  2. Puste katalogi oznaczyć plikiem `.gitkeep`.
  3. Odtworzyć `.gitignore` (usunięty w commicie `2c84d45`): `.venv/`, `__pycache__/`,
     `data/*` z wyjątkiem `data/splits/` i `data/metadata/`, `models/`, `experiments/*/weights/`,
     `results/`, `configs/paths.yaml`, pliki tymczasowe Office (`~$*.docx`).
  4. Dodać `configs/paths.example.yaml` jako wzór lokalnych ścieżek.
- **Rezultat:** kompletny szkielet repozytorium.
- **Gotowe, gdy:** `git status` nie pokazuje `.venv/` ani danych; struktura zgadza się
  z sekcją 13.2.
- **Zależy od:** —

#### F1-T5 — Pakiet instalowalny (`pyproject.toml`) · 1 h
- **Cel:** importy działają jednakowo w skryptach, testach i notatnikach.
- **Kroki:**
  1. `pyproject.toml`: nazwa `cardvision`, wersja `0.1.0`, `src-layout`, wymagany Python
     `>=3.11,<3.13`, sekcja `[tool.pytest.ini_options]`.
  2. `src/cardvision/__init__.py` z `__version__`.
  3. `pip install -e .`
- **Rezultat:** pakiet `cardvision` zainstalowany w trybie edycji.
- **Gotowe, gdy:** `python -c "import cardvision; print(cardvision.__version__)"` działa
  z dowolnego katalogu.
- **Zależy od:** F1-T3, F1-T4

#### F1-T6 — Weryfikacja posiadanych talii · 0,5 h
- **Cel:** potwierdzić, że talie nie unieważniają założeń planu (augmentacja obrotem,
  porównywalność z danymi publicznymi).
- **Kroki:** dla każdej talii sprawdzić według listy z sekcji 5.7: rozmiar (poker/bridge),
  krój indeksów, liczbę kolorów indeksów (2 czy 4), symetrię obrotu o 180°, powierzchnię
  (połysk); wybrać talię główną i ewentualnie drugą (tylko do testu); zrobić po jednym
  zdjęciu referencyjnym każdej talii.
- **Rezultat:** sekcja „Talie" w `docs/DATASET.md`.
- **Gotowe, gdy:** wiadomo, czy augmentacja obrotem 180° jest dozwolona i czy rozszerzenie
  X3 (druga talia) jest wykonalne.
- **Zależy od:** —

#### F1-T7 — Moduł etykiet `labels.py` · 1,5 h
- **Cel:** jedno źródło prawdy o 52 klasach — błąd tutaj po cichu unieważnia każdy wynik (R9).
- **Kroki:**
  1. `src/cardvision/labels.py`: stałe `RANKS` (13), `SUITS` (4), `CARDS` (52) w ustalonej,
     udokumentowanej kolejności.
  2. Funkcje: `card_to_index`, `index_to_card`, `card_to_rank_suit`, `rank_suit_to_card`,
     indeksy figury i koloru dla wariantu multi-head.
  3. Funkcja pomocnicza `suit_color` (czerwony/czarny) — do metryk z sekcji 10.2.
  4. Funkcja mapująca etykiety zbioru publicznego na nazwy kanoniczne (uzupełniana w F2).
- **Rezultat:** `src/cardvision/labels.py`.
- **Gotowe, gdy:** moduł importuje się, ma docstringi i nie zawiera zależności od innych
  modułów projektu.
- **Zależy od:** F1-T5

#### F1-T8 — Testy jednostkowe `labels.py` · 1 h
- **Cel:** zabezpieczyć mapowanie etykiet przed cichymi błędami.
- **Kroki:** testy: dokładnie 52 unikalne karty; odwracalność karta ↔ indeks ↔ (figura,
  kolor) dla wszystkich 52; stabilność kolejności (porównanie z zapisaną listą); błąd dla
  nieznanej karty; brak jokera.
- **Rezultat:** `tests/unit/test_labels.py`.
- **Gotowe, gdy:** `pytest tests/unit/test_labels.py` przechodzi.
- **Zależy od:** F1-T7

#### F1-T9 — Ziarno losowe i katalog przebiegu · 1,5 h
- **Cel:** fundament reprodukowalności (cel M1) — każde uruchomienie zostawia pełny ślad.
- **Kroki:**
  1. `utils/seed.py`: `set_seed(seed, deterministic)` dla `random`, NumPy, PyTorch (CPU i CUDA),
     cuDNN.
  2. `utils/experiment.py`: tworzenie katalogu `experiments/<data>_<eksperyment>_<wariant>_s<ziarno>/`,
     zapis `config.yaml` (pełna, rozwinięta konfiguracja) i `environment.json` (Python,
     wersje bibliotek, GPU, sterownik, hash commita, flaga niezacommitowanych zmian).
  3. Test: dwa wywołania `set_seed(42)` dają identyczne liczby losowe.
- **Rezultat:** `src/cardvision/utils/{seed,experiment}.py`, `tests/unit/test_seed.py`.
- **Gotowe, gdy:** testowe utworzenie katalogu przebiegu daje kompletny `environment.json`.
- **Zależy od:** F1-T5

#### F1-T10 — Konfiguracja `pytest` i dziennik prac · 0,5 h
- **Cel:** jedno polecenie uruchamiające wszystkie testy; miejsce na notatki między sesjami.
- **Kroki:** `tests/conftest.py`; uruchomienie pełnego `pytest`; utworzenie `docs/JOURNAL.md`
  z formatem wpisu z sekcji 14.5 i pierwszym wpisem.
- **Rezultat:** `tests/conftest.py`, `docs/JOURNAL.md`.
- **Gotowe, gdy:** `pytest` w katalogu głównym przechodzi w całości.
- **Zależy od:** F1-T8, F1-T9

#### F1-T11 — Pierwszy commit kodu i repozytorium zdalne · 0,5 h
- **Cel:** historia zmian i kopia zapasowa (R10).
- **Kroki:** ustalić konwencję komunikatów commitów (np. `F1-T7: moduł etykiet`); commit
  szkieletu; założenie prywatnego repozytorium zdalnego i `git push`.
- **Rezultat:** commit z kodem Fazy 1 w repozytorium lokalnym i zdalnym.
- **Gotowe, gdy:** `git log` pokazuje commit, a repozytorium zdalne zawiera ten sam stan.
- **Zależy od:** F1-T10

**Weryfikacja fazy:**
```
python -c "import torch; print(torch.__version__, torch.cuda.is_available())"   → True
python -c "import cardvision; print(cardvision.__version__)"                    → 0.1.0
pytest                                                                          → wszystkie zielone
```

**Kryterium przejścia:** wszystkie trzy komendy kończą się powodzeniem; talie opisane.

---

## FAZA 2 — Dane: pozyskanie i audyt

**Cel fazy:** zweryfikowany, opisany i podzielony zbiór danych, o którym wiadomo, co zawiera,
skąd pochodzi i że nie ma w nim przecieku między podziałami.

**Wejście:** ukończona F1.

**Szacunek:** 10–14 h

### Postęp

- [ ] F2-T1 — Przegląd i wybór zbiorów publicznych
- [ ] F2-T2 — Pobranie zbioru klasyfikacyjnego
- [ ] F2-T3 — Pobranie zbioru detekcyjnego
- [ ] F2-T4 — Skrypt audytu danych
- [ ] F2-T5 — Uruchomienie audytu i ręczny przegląd próbki
- [ ] F2-T6 — Czyszczenie zbioru
- [ ] F2-T7 — Budowa podziałów train / val / test
- [ ] F2-T8 — Weryfikacja rozłączności podziałów
- [ ] F2-T9 — Raport `docs/DATASET.md`

### Zadania

#### F2-T1 — Przegląd i wybór zbiorów publicznych · 2 h
- **Cel:** wybrać zbiory spełniające kryteria z sekcji 5.2, zanim cokolwiek zostanie pobrane.
- **Kroki:** przejrzeć Kaggle i Roboflow Universe; dla każdego kandydata odnotować: licencję,
  liczbę klas (czy jest joker), liczbę obrazów, rozdzielczość, czy zbiór jest już
  zaugmentowany, format anotacji; ocenić wg kryteriów w kolejności ważności (licencja →
  brak augmentacji → standardowa talia → rozdzielczość → zrównoważenie).
- **Rezultat:** tabela kandydatów w `docs/DATASET.md` z decyzją i uzasadnieniem.
- **Gotowe, gdy:** wybrany jest jeden zbiór klasyfikacyjny i jeden detekcyjny, każdy
  z kandydatem zapasowym.
- **Zależy od:** —

#### F2-T2 — Pobranie zbioru klasyfikacyjnego · 1 h
- **Cel:** dane w nienaruszonej postaci, z pełną informacją o pochodzeniu.
- **Kroki:** `scripts/download_data.py` (lub instrukcja ręczna, jeśli pobranie wymaga
  logowania); rozpakowanie do `data/raw/public_classification/`; obliczenie SHA-256 archiwum;
  wpis w `data/metadata/sources.md` (nazwa, adres, autor, licencja, data, liczba plików,
  suma kontrolna).
- **Rezultat:** dane w `data/raw/`, wpis w `sources.md`.
- **Gotowe, gdy:** liczba plików zgadza się z deklaracją źródła.
- **Zależy od:** F2-T1

#### F2-T3 — Pobranie zbioru detekcyjnego · 1 h
- **Cel:** jak w F2-T2, dla danych z bounding boxami.
- **Kroki:** jak w F2-T2, do `data/raw/public_detection/`; odnotować format anotacji
  (YOLO/COCO/VOC).
- **Rezultat:** dane w `data/raw/`, wpis w `sources.md`.
- **Gotowe, gdy:** liczba obrazów i plików anotacji się zgadza.
- **Zależy od:** F2-T1

#### F2-T4 — Skrypt audytu danych · 3 h
- **Cel:** narzędzie wykrywające wady, które zawyżają wyniki (R2).
- **Kroki:** `scripts/audit_dataset.py` realizujący kontrole z sekcji 5.3:
  1. duplikaty i near-duplicates — pHash, próg odległości Hamminga w konfiguracji,
  2. augmentowane kopie — analiza nazw plików + pHash między istniejącymi podziałami,
  3. histogram liczności klas,
  4. statystyki rozdzielczości i proporcji,
  5. lista klas spoza standardu (joker),
  6. walidacja bboxów: poza kadrem, zerowe pole, nieznana klasa,
  7. eksport losowej próbki 100 obrazów do przeglądu (siatka z etykietami).
  Wynik: raport tekstowy/JSON + wykresy w `data/interim/audit/`.
- **Rezultat:** `scripts/audit_dataset.py`.
- **Gotowe, gdy:** skrypt działa na obu zbiorach i zwraca liczby dla każdej kontroli.
- **Zależy od:** F2-T2, F2-T3

#### F2-T5 — Uruchomienie audytu i ręczny przegląd próbki · 2 h
- **Cel:** poznać rzeczywisty stan danych.
- **Kroki:** uruchomić audyt; obejrzeć siatkę 100 losowych obrazów i policzyć błędne etykiety;
  obejrzeć znalezione pary duplikatów (czy to naprawdę duplikaty); spisać liczby.
- **Rezultat:** wyniki audytu (liczby do `DATASET.md`).
- **Gotowe, gdy:** dla każdej kontroli jest konkretna liczba i decyzja (akceptacja /
  czyszczenie / odrzucenie zbioru).
- **Zależy od:** F2-T4

#### F2-T6 — Czyszczenie zbioru · 1,5 h
- **Cel:** usunąć wady wykryte w audycie bez modyfikowania `data/raw/`.
- **Kroki:** lista wykluczeń (duplikaty, jokery, uszkodzone, zbyt małe) zapisana jako plik
  w `data/metadata/`; zbiór oczyszczony = surowy minus lista wykluczeń; ponowne uruchomienie
  audytu na zbiorze oczyszczonym.
- **Rezultat:** `data/metadata/exclusions.csv` (z powodem wykluczenia dla każdego pliku).
- **Gotowe, gdy:** ponowny audyt nie zgłasza duplikatów ani klas spoza standardu.
- **Zależy od:** F2-T5

#### F2-T7 — Budowa podziałów train / val / test · 2 h
- **Cel:** podział zgodny z sekcją 5.5 — rozłączny na poziomie źródła, nie pojedynczego pliku.
- **Kroki:** `scripts/build_splits.py`: grupowanie po źródle/oryginale (pHash), podział
  ~70/15/15 stratyfikowany po klasach, ziarno z konfiguracji; zapis list plików do
  `data/splits/classification_v1.json` i `data/splits/detection_v1.json`.
- **Rezultat:** pliki podziałów w gicie.
- **Gotowe, gdy:** każda z 52 klas jest obecna w każdym podziale.
- **Zależy od:** F2-T6

#### F2-T8 — Weryfikacja rozłączności podziałów · 1 h
- **Cel:** dowód braku przecieku — kryterium przejścia fazy.
- **Kroki:** porównanie pHash każdej pary (train × val, train × test, val × test); raport
  liczby kolizji.
- **Rezultat:** wynik weryfikacji zapisany w `DATASET.md`.
- **Gotowe, gdy:** **zero kolizji**. W przeciwnym razie poprawić F2-T7 i powtórzyć.
- **Zależy od:** F2-T7

#### F2-T9 — Raport `docs/DATASET.md` · 1,5 h
- **Cel:** dokument, który trafi niemal bezpośrednio do rozdziału 5 pracy.
- **Kroki:** źródła i licencje; wyniki wszystkich kontrol audytu z liczbami; decyzje
  czyszczenia; histogram klas; opis podziałów i dowód rozłączności; opis talii (z F1-T6).
- **Rezultat:** `docs/DATASET.md`.
- **Gotowe, gdy:** dokument pozwala odtworzyć zbiór od zera.
- **Zależy od:** F2-T8

**Kryterium przejścia:** zero kolizji pHash między podziałami; `DATASET.md` kompletny.

---

## FAZA 3 — Preprocessing, geometria i detektor klasyczny

**Cel fazy:** wycinanie i prostowanie kart bez żadnego modelu uczonego. Detektor konturowy
jest jednocześnie baseline'em dla E7 i zabezpieczeniem na wypadek problemów z YOLO (R11).

**Wejście:** ukończona F2.

**Szacunek:** 12–16 h

### Postęp

- [ ] F3-T1 — Jednolite źródło obrazów
- [ ] F3-T2 — Preprocessing podstawowy
- [ ] F3-T3 — Testy preprocessingu
- [ ] F3-T4 — Interfejs `CardDetector`
- [ ] F3-T5 — Detektor konturowy
- [ ] F3-T6 — Wykrywanie i porządkowanie narożników
- [ ] F3-T7 — Prostowanie perspektywy
- [ ] F3-T8 — Testy geometrii
- [ ] F3-T9 — Filtry jakości obrazu
- [ ] F3-T10 — Skrypt poglądowy i kontrola wizualna

### Zadania

#### F3-T1 — Jednolite źródło obrazów · 2 h
- **Cel:** reszta systemu nie wie, czy klatka pochodzi z pliku, katalogu, wideo czy kamery.
- **Kroki:** `io/image_source.py`: wspólny interfejs iteratora zwracającego `(klatka BGR,
  metadane)`; implementacje: plik, katalog, wideo, kamera; czytelne błędy (brak pliku,
  brak kamery).
- **Rezultat:** `src/cardvision/io/image_source.py`.
- **Gotowe, gdy:** każde z czterech źródeł zwraca klatki tym samym interfejsem.
- **Zależy od:** F1

#### F3-T2 — Preprocessing podstawowy · 2 h
- **Cel:** wspólne przekształcenia obrazu z możliwością cofnięcia współrzędnych.
- **Kroki:** `preprocessing/basic.py`: letterbox z zachowaniem proporcji, zmiana rozmiaru,
  BGR→RGB, normalizacja; funkcje przeliczające współrzędne między obrazem oryginalnym
  a przeskalowanym (w obie strony). Parametry z konfiguracji.
- **Rezultat:** `src/cardvision/preprocessing/basic.py`.
- **Gotowe, gdy:** funkcje działają na obrazach o różnych proporcjach.
- **Zależy od:** F1

#### F3-T3 — Testy preprocessingu · 1 h
- **Cel:** błąd przeliczenia współrzędnych przesuwa bboxy — musi być wykryty testem.
- **Kroki:** testy: letterbox zachowuje proporcje; współrzędne → przeskalowane → oryginalne
  dają punkt wyjściowy (z tolerancją); obrazy skrajne (bardzo wąskie, 1×1).
- **Rezultat:** `tests/unit/test_preprocessing.py`.
- **Gotowe, gdy:** testy przechodzą.
- **Zależy od:** F3-T2

#### F3-T4 — Interfejs `CardDetector` · 1 h
- **Cel:** punkt wymienności — kontur i YOLO podłączane bez zmian w pipelinie.
- **Kroki:** `detection/base.py`: klasa abstrakcyjna z metodą `detect(image) → list[Detection]`;
  typ `Detection` (bbox w pikselach, `det_confidence`, opcjonalnie kontur/narożniki).
- **Rezultat:** `src/cardvision/detection/base.py`.
- **Gotowe, gdy:** kontrakt wejścia i wyjścia opisany w docstringu.
- **Zależy od:** F1

#### F3-T5 — Detektor konturowy · 3 h
- **Cel:** działający baseline detekcji z klasycznego CV.
- **Kroki:** `detection/contour.py`: skala szarości → rozmycie → progowanie (Otsu i adaptacyjne,
  wybór w konfiguracji) → morfologia → kontury; `detection/postprocess.py`: filtr pola
  i proporcji boków (~1:1,4 z tolerancją), NMS; wszystkie progi w `configs/detector/contour.yaml`.
- **Rezultat:** `src/cardvision/detection/{contour,postprocess}.py`, `configs/detector/contour.yaml`.
- **Gotowe, gdy:** na zdjęciu kilku kart na kontrastowym tle wszystkie są wykryte.
- **Zależy od:** F3-T4

#### F3-T6 — Wykrywanie i porządkowanie narożników · 2 h
- **Cel:** cztery narożniki w ustalonej kolejności, z kartą zawsze w pionie.
- **Kroki:** `geometry/corners.py`: `approxPolyDP` z tolerancją proporcjonalną do obwodu;
  fallback `minAreaRect` przy 5–6 wierzchołkach; porządkowanie (lewy górny → prawy górny →
  prawy dolny → lewy dolny); orientacja wg dłuższego boku.
- **Rezultat:** `src/cardvision/geometry/corners.py`.
- **Gotowe, gdy:** narożniki są poprawnie uporządkowane niezależnie od kolejności wejścia.
- **Zależy od:** F3-T5

#### F3-T7 — Prostowanie perspektywy · 2 h
- **Cel:** kanoniczny obraz karty niezależny od kąta obserwacji.
- **Kroki:** `geometry/rectify.py`: `getPerspectiveTransform` + `warpPerspective` do rozmiaru
  docelowego (proporcja ~1:1,4, rozmiar z konfiguracji); ścieżka awaryjna: prosty crop z
  marginesem, gdy brak czworokąta; flaga `rectified` w wyniku.
- **Rezultat:** `src/cardvision/geometry/rectify.py`.
- **Gotowe, gdy:** karta sfotografowana pod kątem daje prostokątny wycinek w pionie.
- **Zależy od:** F3-T6

#### F3-T8 — Testy geometrii · 1,5 h
- **Cel:** ciche pomylenie narożników daje obraz obrócony lub odbity — musi być wykryte.
- **Kroki:** testy: porządkowanie dla wszystkich permutacji 4 punktów; syntetyczny prostokąt
  zniekształcony znaną homografią jest odtwarzany (z tolerancją); przypadki zdegenerowane
  (punkty współliniowe) uruchamiają ścieżkę awaryjną.
- **Rezultat:** `tests/unit/test_geometry.py`.
- **Gotowe, gdy:** testy przechodzą.
- **Zależy od:** F3-T7

#### F3-T9 — Filtry jakości obrazu · 1 h
- **Cel:** wykrywanie obrazów nieostrych i zbyt ciemnych (sekcja 9.6).
- **Kroki:** `preprocessing/quality.py`: wariancja laplasjanu (ostrość), średnia jasność;
  progi w konfiguracji.
- **Rezultat:** `src/cardvision/preprocessing/quality.py`.
- **Gotowe, gdy:** obraz celowo rozmyty ma wyraźnie niższą miarę ostrości niż oryginał.
- **Zależy od:** F1

#### F3-T10 — Skrypt poglądowy i kontrola wizualna · 1,5 h
- **Cel:** zobaczyć na własne oczy, że wycinki są poprawne.
- **Kroki:** skrypt: obraz → detekcja → narożniki → prostowanie → zapis wycinków i obrazu
  z naniesionymi konturami; uruchomić na 20 obrazach ze zbioru publicznego; policzyć
  poprawnie wyprostowane karty.
- **Rezultat:** skrypt w `scripts/`, wynik kontroli w `JOURNAL.md`.
- **Gotowe, gdy:** wycinki są kartami w pionie z czytelnym indeksem narożnym.
- **Zależy od:** F3-T1…T9

**Kryterium przejścia:** wizualna kontrola potwierdza poprawne wycinki; testy geometrii
i preprocessingu przechodzą.

---

## FAZA 4 — Klasyfikator kart

**Cel fazy:** wytrenowany klasyfikator 52 klas z udokumentowanym wynikiem walidacyjnym
oraz kod treningu gotowy do wszystkich eksperymentów klasyfikacji.

**Wejście:** ukończone F2 (dane i podziały) i F3 (wycinki).

**Szacunek:** 14–18 h

### Postęp

- [ ] F4-T1 — Dataset klasyfikacyjny
- [ ] F4-T2 — Polityki augmentacji
- [ ] F4-T3 — Wizualna kontrola augmentacji
- [ ] F4-T4 — Model SimpleCNN
- [ ] F4-T5 — Modele pretrenowane (ResNet18, MobileNetV3-Small)
- [ ] F4-T6 — Model MultiHead (13 + 4)
- [ ] F4-T7 — Pętla treningowa
- [ ] F4-T8 — Skrypt `train_classifier.py`
- [ ] F4-T9 — Pierwszy pełny trening
- [ ] F4-T10 — Metryki klasyfikacji i macierz pomyłek
- [ ] F4-T11 — Analiza pierwszej macierzy pomyłek

### Zadania

#### F4-T1 — Dataset klasyfikacyjny · 2 h
- **Cel:** ładowanie danych wyłącznie przez pliki podziałów.
- **Kroki:** `classification/dataset.py`: `Dataset` czytający listę plików z
  `data/splits/*.json`; etykiety przez `labels.py`; tryb zwracający etykietę 52-klasową
  lub parę (figura, kolor); normalizacja ImageNet dla modeli pretrenowanych.
- **Rezultat:** `src/cardvision/classification/dataset.py`.
- **Gotowe, gdy:** `DataLoader` zwraca batch o poprawnych wymiarach i etykietach.
- **Zależy od:** F2-T7, F1-T7

#### F4-T2 — Polityki augmentacji · 2 h
- **Cel:** augmentacja zgodna z analizą z sekcji 6, przełączana konfiguracją.
- **Kroki:** `classification/augment.py` + `configs/augmentation/{none,geometric,photometric,full}.yaml`
  oraz wariant celowo błędny `full_with_flip.yaml` (dla E4); obrót 180° tylko jeśli F1-T6
  potwierdził symetrię; **brak odbić lustrzanych** w wariantach poprawnych.
- **Rezultat:** moduł augmentacji i 5 plików konfiguracji.
- **Gotowe, gdy:** każda polityka ładuje się z YAML bez zmian w kodzie.
- **Zależy od:** F4-T1

#### F4-T3 — Wizualna kontrola augmentacji · 1 h
- **Cel:** pewność, że augmentacja nie niszczy informacji (np. nie ucina indeksu narożnego).
- **Kroki:** dla każdej polityki zapisać siatkę 64 zaugmentowanych obrazów z etykietami;
  obejrzeć je.
- **Rezultat:** siatki w `docs/img/` lub `data/interim/`, wnioski w `JOURNAL.md`.
- **Gotowe, gdy:** na żadnym obrazie indeks narożny nie jest systematycznie niewidoczny.
- **Zależy od:** F4-T2

#### F4-T4 — Model SimpleCNN · 2 h
- **Cel:** własna sieć jako baseline dla E1 i E2.
- **Kroki:** architektura z sekcji 8.2 w `classification/models.py`; konfiguracja
  `configs/classifier/simple_cnn.yaml`; sprawdzenie liczby parametrów.
- **Rezultat:** klasa `SimpleCNN`.
- **Gotowe, gdy:** przejście w przód dla losowego batcha zwraca tensor `[B, 52]`.
- **Zależy od:** F1

#### F4-T5 — Modele pretrenowane (ResNet18, MobileNetV3-Small) · 1,5 h
- **Cel:** modele do transfer learningu.
- **Kroki:** wczytanie wag ImageNet z `torchvision`; podmiana głowy na 52 wyjścia; opcja
  zamrożenia backbone'u (dla E2); konfiguracje `resnet18.yaml`, `mobilenetv3.yaml`.
- **Rezultat:** fabryka modeli wybierająca architekturę po nazwie z konfiguracji.
- **Gotowe, gdy:** oba modele zwracają `[B, 52]`; zamrożenie wyłącza gradienty backbone'u.
- **Zależy od:** F4-T4

#### F4-T6 — Model MultiHead (13 + 4) · 1,5 h
- **Cel:** wariant C dekompozycji problemu dla E3.
- **Kroki:** wspólny backbone, dwie głowy (13 figur, 4 kolory); funkcja straty jako suma
  ważona dwóch entropii krzyżowych; predykcja karty złożona z obu głów przez `labels.py`;
  `multihead.yaml`.
- **Rezultat:** klasa `MultiHeadClassifier`.
- **Gotowe, gdy:** model zwraca dwa tensory `[B, 13]` i `[B, 4]`, a predykcja karty jest
  poprawną kartą z 52.
- **Zależy od:** F4-T5

#### F4-T7 — Pętla treningowa · 3 h
- **Cel:** jeden trener dla wszystkich modeli i eksperymentów.
- **Kroki:** `classification/train.py`: AMP, optymalizator i harmonogram LR z konfiguracji,
  early stopping po `val`, zapis najlepszych wag, log po każdej epoce do
  `metrics/training_log.csv`, obsługa trybu 52-klasowego i multi-head; test „przeuczenia
  10 obrazów" (model musi osiągnąć ~100% na 10 przykładach).
- **Rezultat:** `src/cardvision/classification/train.py`.
- **Gotowe, gdy:** test przeuczenia przechodzi — to dowód, że kod uczący działa.
- **Zależy od:** F4-T1, F4-T6, F1-T9

#### F4-T8 — Skrypt `train_classifier.py` · 1,5 h
- **Cel:** trening uruchamiany jedną komendą z pliku konfiguracji.
- **Kroki:** `scripts/train_classifier.py --config ...`: ustawienie ziarna, utworzenie
  katalogu przebiegu, trening, zapis metryk i wag.
- **Rezultat:** punkt wejścia CLI.
- **Gotowe, gdy:** krótki trening (2 epoki) tworzy kompletny katalog w `experiments/`.
- **Zależy od:** F4-T7

#### F4-T9 — Pierwszy pełny trening · 1,5 h
- **Cel:** pierwszy rzeczywisty wynik i pomiar czasu treningu.
- **Kroki:** ResNet18 + pełna augmentacja; zapis dokładności walidacyjnej i czasu treningu;
  utworzenie `docs/EXPERIMENTS.md` i pierwszy wpis.
- **Rezultat:** katalog przebiegu, wagi, wpis w rejestrze.
- **Gotowe, gdy:** dokładność walidacyjna wyraźnie powyżej losowej (1/52 ≈ 1,9%).
- **Zależy od:** F4-T8

#### F4-T10 — Metryki klasyfikacji i macierz pomyłek · 2 h
- **Cel:** poprawnie policzone metryki — błąd tutaj daje fałszywe liczby w pracy.
- **Kroki:** `evaluation/metrics_cls.py`: accuracy, macro-F1, P/R/F1 per klasa, top-3,
  dokładność figury, koloru i barwy (sekcja 10.2); `evaluation/confusion.py`: macierze
  52×52, 13×13, 4×4; testy na ręcznie policzonych przykładach.
- **Rezultat:** moduły metryk + `tests/unit/test_metrics_cls.py`.
- **Gotowe, gdy:** testy przechodzą; metryki zgadzają się z wyliczeniem ręcznym.
- **Zależy od:** F1-T7

#### F4-T11 — Analiza pierwszej macierzy pomyłek · 1 h
- **Cel:** wykryć błędy danych lub kodu, zanim zostaną powielone w eksperymentach.
- **Kroki:** wygenerować macierze dla modelu z F4-T9; sprawdzić, czy pomyłki są „sensowne"
  (`6`↔`9`, `♠`↔`♣`) czy systematyczne (przesunięcie klas → błąd etykiet); obejrzeć
  przykłady najczęstszych pomyłek.
- **Rezultat:** wnioski w `JOURNAL.md`.
- **Gotowe, gdy:** brak wzorców wskazujących na błąd w etykietach.
- **Zależy od:** F4-T9, F4-T10

**Kryterium przejścia:** dokładność walidacyjna na poziomie użytecznym dla pipeline'u.
Przy bardzo niskim wyniku — diagnoza (mapowanie etykiet, normalizacja, wycinki), nie
przechodzenie dalej.

---

## FAZA 5 — Detektor YOLO

**Cel fazy:** wytrenowany detektor jednoklasowy, wymienialny z detektorem konturowym bez
zmian w pozostałym kodzie.

**Wejście:** ukończone F2 i F3 (interfejs `CardDetector`). Może być realizowana równolegle z F4.

**Szacunek:** 12–16 h

### Postęp

- [ ] F5-T1 — Konwersja zbioru do formatu YOLO (jedna klasa)
- [ ] F5-T2 — Generator scen syntetycznych
- [ ] F5-T3 — Wygenerowanie i kontrola scen syntetycznych
- [ ] F5-T4 — Konfiguracja treningu YOLO
- [ ] F5-T5 — Trening wariantu `n`
- [ ] F5-T6 — Implementacja `YoloDetector`
- [ ] F5-T7 — Metryki detekcji
- [ ] F5-T8 — Analiza błędów detekcji
- [ ] F5-T9 — Trening wariantu `s`

### Zadania

#### F5-T1 — Konwersja zbioru do formatu YOLO (jedna klasa) · 2 h
- **Cel:** zbiór detekcyjny z jedną klasą „karta" — klasyfikacja należy do drugiego etapu.
- **Kroki:** konwersja anotacji do formatu YOLO (`klasa cx cy w h`, znormalizowane);
  wszystkie klasy → `0`; zachowanie podziału z `detection_v1.json`; plik `dataset.yaml`;
  wizualna kontrola 20 obrazów z naniesionymi boxami.
- **Rezultat:** zbiór w `data/processed/detection/`.
- **Gotowe, gdy:** boxy na obrazach kontrolnych pokrywają karty.
- **Zależy od:** F2-T8

#### F5-T2 — Generator scen syntetycznych · 4 h
- **Cel:** dowolnie wiele scen wielokartowych z automatyczną, dokładną anotacją.
- **Kroki:** `synth/compose.py` + `synth/backgrounds.py`: wycinek karty + losowe tło +
  losowa homografia (0–60°) + jasność/kontrast/temperatura barwowa + cień + nakładanie
  1–10 kart z kontrolą zasłonięcia; bbox = obraz narożników przez homografię; zapis
  parametrów każdej sceny (do E10b/E10c); wycinki **tylko ze zbioru treningowego**.
- **Rezultat:** moduł `synth`, `scripts/generate_synthetic.py`.
- **Gotowe, gdy:** wygenerowana scena ma boxy idealnie dopasowane do kart.
- **Zależy od:** F2-T7, F3-T7

#### F5-T3 — Wygenerowanie i kontrola scen syntetycznych · 1,5 h
- **Cel:** zbiór syntetyczny gotowy do treningu.
- **Kroki:** wygenerować ~5000 scen do `data/synthetic/`; obejrzeć losowe 50; sprawdzić
  rozkład liczby kart na scenę.
- **Rezultat:** `data/synthetic/`.
- **Gotowe, gdy:** brak scen z błędnymi boxami w próbce kontrolnej.
- **Zależy od:** F5-T2

#### F5-T4 — Konfiguracja treningu YOLO · 1 h
- **Cel:** augmentacje YOLO zgodne z polityką z sekcji 6.
- **Kroki:** `configs/detector/yolo_n.yaml`: **`fliplr: 0`, `flipud: 0`**, obniżone `hsv_h`,
  mozaika włączona, rozmiar obrazu, batch, liczba epok, ziarno.
- **Rezultat:** plik konfiguracji.
- **Gotowe, gdy:** konfiguracja jawnie wyłącza odbicia.
- **Zależy od:** F5-T1

#### F5-T5 — Trening wariantu `n` · 2 h
- **Cel:** detektor produkcyjny.
- **Kroki:** `scripts/train_detector.py --config ...` (opakowanie Ultralytics zapisujące
  katalog przebiegu jak w F1-T9); trening na danych rzeczywistych + syntetycznych; zapis
  mAP@50, recall, precision, czasu treningu.
- **Rezultat:** wagi w `models/detector/`, katalog przebiegu.
- **Gotowe, gdy:** metryki zapisane, krzywe uczenia bez oznak rozbieżności.
- **Zależy od:** F5-T3, F5-T4

#### F5-T6 — Implementacja `YoloDetector` · 1,5 h
- **Cel:** YOLO za tym samym interfejsem co detektor konturowy.
- **Kroki:** `detection/yolo.py`: wczytanie wag, inferencja, konwersja wyników do
  `Detection`, filtracja przez `postprocess.py`; próg pewności z konfiguracji.
- **Rezultat:** `src/cardvision/detection/yolo.py`.
- **Gotowe, gdy:** ten sam obraz przez `ContourDetector` i `YoloDetector` daje wyniki tego
  samego typu.
- **Zależy od:** F5-T5, F3-T4

#### F5-T7 — Metryki detekcji · 2 h
- **Cel:** własna, przetestowana implementacja metryk do E7 i metryki end-to-end.
- **Kroki:** `evaluation/metrics_det.py`: IoU, dopasowanie predykcji do prawdy, precision,
  recall, mAP@50; testy na ręcznie policzonych przykładach.
- **Rezultat:** moduł + `tests/unit/test_metrics_det.py`.
- **Gotowe, gdy:** testy przechodzą; wynik na zbiorze testowym zgadza się z raportem
  Ultralytics (z dokładnością do różnic definicji).
- **Zależy od:** —

#### F5-T8 — Analiza błędów detekcji · 1,5 h
- **Cel:** zrozumieć, gdzie detektor zawodzi.
- **Kroki:** zapisać obrazy z fałszywymi detekcjami i pominiętymi kartami; pogrupować błędy
  (karty stykające się, krawędź kadru, tło).
- **Rezultat:** wnioski w `JOURNAL.md`, przykładowe obrazy.
- **Gotowe, gdy:** każda kategoria błędów ma przykład i hipotezę przyczyny.
- **Zależy od:** F5-T6, F5-T7

#### F5-T9 — Trening wariantu `s` · 1,5 h
- **Cel:** zmierzyć, czy większy model jest wart kosztu.
- **Kroki:** trening `yolo_s.yaml` w identycznych warunkach; porównanie mAP i czasu inferencji.
- **Rezultat:** katalog przebiegu, porównanie w `EXPERIMENTS.md`.
- **Gotowe, gdy:** decyzja `n` vs `s` poparta pomiarem.
- **Zależy od:** F5-T5

**Kryterium przejścia:** recall wystarczająco wysoki, by pipeline miał sens (pominięta
karta jest nie do odzyskania); `YoloDetector` spełnia kontrakt.

---

## FAZA 6 — Integracja end-to-end `[KAMIEŃ MILOWY: MVP]`

**Cel fazy:** jeden spójny system — obraz na wejściu, lista rozpoznanych kart na wyjściu.
Po tej fazie projekt jest obronny.

**Wejście:** ukończone F3, F4 i F5.

**Szacunek:** 10–12 h

### Postęp

- [ ] F6-T1 — Typy wyników i pomiar czasu
- [ ] F6-T2 — Pipeline `card_pipeline.py`
- [ ] F6-T3 — Budowa komponentów z konfiguracji
- [ ] F6-T4 — Zapis wyników
- [ ] F6-T5 — CLI `predict.py`
- [ ] F6-T6 — Test integracyjny pipeline'u
- [ ] F6-T7 — Testy przypadków brzegowych
- [ ] F6-T8 — Scenariusze praktyczne P1–P4

### Zadania

#### F6-T1 — Typy wyników i pomiar czasu · 1,5 h
- **Cel:** jednoznaczny kontrakt wyniku dla CLI, GUI i ewaluacji.
- **Kroki:** `pipeline/result.py`: `CardResult` (bbox, karta, pewność, top-k, flaga
  `rectified`, status), `FrameResult` (lista kart, czasy etapów, ostrzeżenia);
  `utils/timing.py` — pomiar czasu etapów.
- **Rezultat:** `src/cardvision/pipeline/result.py`, `src/cardvision/utils/timing.py`.
- **Gotowe, gdy:** wynik serializuje się do JSON.
- **Zależy od:** F1

#### F6-T2 — Pipeline `card_pipeline.py` · 3 h
- **Cel:** serce systemu — spięcie wszystkich etapów.
- **Kroki:**
  1. `classification/base.py`: interfejs `CardClassifier` i implementacja opakowująca
     wytrenowany model (wczytanie wag, **preprocessing identyczny jak w treningu**,
     `predict` zwracający rozkład prawdopodobieństw).
  2. `pipeline/card_pipeline.py`: preprocessing → detekcja → crop z marginesem →
     prostowanie (przełącznik) → klasyfikacja wsadowa wszystkich kart → `FrameResult`.
  3. Pipeline nie zna konkretnych implementacji detektora i klasyfikatora.
- **Rezultat:** `src/cardvision/{classification/base.py,pipeline/card_pipeline.py}`.
- **Gotowe, gdy:** obraz testowy przechodzi przez pipeline i daje poprawne karty.
- **Zależy od:** F6-T1, F4-T9, F5-T6

#### F6-T3 — Budowa komponentów z konfiguracji · 2 h
- **Cel:** eksperyment = zmiana konfiguracji, nie kodu.
- **Kroki:** `config.py`: wczytanie YAML, walidacja (błąd przy nieznanym kluczu), rejestr
  implementacji po nazwie (`contour`, `yolo`, `resnet18`…); `configs/pipeline/default.yaml`;
  test jednostkowy walidacji.
- **Rezultat:** `src/cardvision/config.py`, `configs/pipeline/default.yaml`,
  `tests/unit/test_config.py`.
- **Gotowe, gdy:** zmiana detektora z `yolo` na `contour` w YAML działa bez zmian w kodzie.
- **Zależy od:** F6-T2

#### F6-T4 — Zapis wyników · 1,5 h
- **Cel:** wynik czytelny dla człowieka i dla skryptów.
- **Kroki:** `io/writers.py`: obraz z ramkami, etykietami i pewnością; JSON i CSV z listą kart.
- **Rezultat:** `src/cardvision/io/writers.py`.
- **Gotowe, gdy:** wszystkie trzy formaty powstają dla obrazu testowego.
- **Zależy od:** F6-T1

#### F6-T5 — CLI `predict.py` · 1,5 h
- **Cel:** jedna komenda do uruchomienia systemu.
- **Kroki:** `scripts/predict.py --input <plik|katalog|wideo> --config ... --output ...`;
  wypisanie listy kart z pewnościami; `scripts/evaluate.py` — ewaluacja pipeline'u na
  zbiorze z anotacjami.
- **Rezultat:** `scripts/{predict,evaluate}.py`.
- **Gotowe, gdy:** komenda z sekcji „Weryfikacja" działa dla pliku, katalogu i wideo.
- **Zależy od:** F6-T3, F6-T4

#### F6-T6 — Test integracyjny pipeline'u · 1 h
- **Cel:** automatyczne sprawdzenie całej ścieżki.
- **Kroki:** mały obraz w `tests/fixtures/`; test: plik → wynik o oczekiwanej strukturze;
  test wymienności: oba detektory zwracają wynik zgodny z kontraktem.
- **Rezultat:** `tests/integration/test_pipeline.py`.
- **Gotowe, gdy:** test przechodzi.
- **Zależy od:** F6-T5

#### F6-T7 — Testy przypadków brzegowych · 1 h
- **Cel:** system nie wywraca się na nietypowym wejściu.
- **Kroki:** testy: obraz bez kart, obraz czarny, plik uszkodzony, obraz 1×1 px.
- **Rezultat:** testy w `tests/integration/`.
- **Gotowe, gdy:** każdy przypadek kończy się czytelnym wynikiem lub komunikatem, nie wyjątkiem.
- **Zależy od:** F6-T6

#### F6-T8 — Scenariusze praktyczne P1–P4 · 1 h
- **Cel:** sprawdzenie na prawdziwych zdjęciach z posiadanej talii.
- **Kroki:** wykonać zdjęcia i scenariusze P1–P4 z sekcji 15.4; zapisać wyniki w `JOURNAL.md`.
- **Rezultat:** protokół testów praktycznych.
- **Gotowe, gdy:** P1 i P2 przechodzą.
- **Zależy od:** F6-T5

**Weryfikacja fazy:**
```
python scripts/predict.py --input przyklad.jpg --config configs/pipeline/default.yaml
```
wypisuje listę kart z pewnościami i zapisuje obraz z ramkami.

**Kryterium przejścia:** P1 i P2 przechodzą; działa z oboma detektorami. Commit oznaczony
tagiem `mvp`.

---

## FAZA 7 — Własny zbiór testowy

**Cel fazy:** zbiór testowy zebrany samodzielnie i opisany metadanymi warunków — podstawa
głównego wyniku pracy oraz eksperymentów E5–E7, E10–E12.

**Wejście:** ukończona F6, talie opisane w F1-T6.

**Szacunek:** 10–14 h (w tym ~2 h nagrywania)

### Postęp

- [ ] F7-T1 — Plan sesji nagraniowych
- [ ] F7-T2 — Sesje 1–4: warunki oświetleniowe
- [ ] F7-T3 — Sesje 5–8: tła i kąty obserwacji
- [ ] F7-T4 — Sesje 9–12: układy wielokartowe
- [ ] F7-T5 — Wyodrębnianie klatek
- [ ] F7-T6 — Metadane sesji
- [ ] F7-T7 — Anotacja scen detekcyjnych
- [ ] F7-T8 — Etykietowanie wycinków klasyfikacyjnych
- [ ] F7-T9 — Podział własny rozłączny na poziomie sesji
- [ ] F7-T10 — Pierwsza ewaluacja na zbiorze własnym

### Zadania

#### F7-T1 — Plan sesji nagraniowych · 1 h
- **Cel:** nagrania pokrywające wszystkie warunki potrzebne do E10.
- **Kroki:** macierz: oświetlenie (dzienne, żarowe ciepłe, LED zimne, przyciemnione) ×
  tło (jasne, ciemne, wzorzyste) × układ (siatka 52 kart, 2/5/10 kart, wachlarz, nachodzące);
  wybór ~12 sesji; przygotowanie `data/metadata/sessions.csv` z kolumnami: `session_id`,
  oświetlenie, tło, talia, kąt, liczba kart, układ, urządzenie, data.
- **Rezultat:** plan i szablon `sessions.csv`.
- **Gotowe, gdy:** każdy poziom każdej zmiennej z E10 jest pokryty co najmniej jedną sesją.
- **Zależy od:** F6

#### F7-T2 — Sesje 1–4: warunki oświetleniowe · 1,5 h
- **Cel:** materiał do E10a.
- **Kroki:** 52 karty w siatce na tym samym tle; po jednym nagraniu 30–60 s w każdym z
  czterech oświetleń; powolny ruch kamery, zmiana wysokości i kąta.
- **Rezultat:** 4 nagrania w `data/own/videos/`.
- **Gotowe, gdy:** nagrania są ostre, a wszystkie karty widoczne.
- **Zależy od:** F7-T1

#### F7-T3 — Sesje 5–8: tła i kąty obserwacji · 1,5 h
- **Cel:** materiał do E10b, E5 i E7.
- **Kroki:** nagrania na różnych tłach; osobne ujęcia w przedziałach kąta 0–15°, 15–30°,
  30–45°, > 45° (kąt odnotowany przy każdym ujęciu).
- **Rezultat:** 4 nagrania.
- **Gotowe, gdy:** każdy przedział kąta ma osobne ujęcia.
- **Zależy od:** F7-T1

#### F7-T4 — Sesje 9–12: układy wielokartowe · 1,5 h
- **Cel:** materiał do E10c i E6.
- **Kroki:** układy 1/2/5/10 kart, wachlarz w ręce, karty nachodzące; ewentualnie druga
  talia (tylko do testu, X3).
- **Rezultat:** 4 nagrania.
- **Gotowe, gdy:** każdy układ z planu ma nagranie.
- **Zależy od:** F7-T1

#### F7-T5 — Wyodrębnianie klatek · 1,5 h
- **Cel:** obrazy z nagrań z zachowaniem informacji o sesji.
- **Kroki:** `scripts/extract_frames.py`: co N-ta klatka, nazwa pliku z `session_id`,
  odrzucanie klatek nieostrych filtrem z `quality.py`.
- **Rezultat:** `data/own/frames/`.
- **Gotowe, gdy:** każda klatka jest jednoznacznie przypisana do sesji.
- **Zależy od:** F7-T2…T4, F3-T9

#### F7-T6 — Metadane sesji · 1 h
- **Cel:** warunki akwizycji dostępne do grupowania wyników w E10.
- **Kroki:** uzupełnić `sessions.csv` dla wszystkich sesji.
- **Rezultat:** kompletny `data/metadata/sessions.csv`.
- **Gotowe, gdy:** brak pustych pól.
- **Zależy od:** F7-T5

#### F7-T7 — Anotacja scen detekcyjnych · 2,5 h
- **Cel:** zbiór testowy detekcji z prawdziwych zdjęć.
- **Kroki:** wybrać ~100 scen (różne sesje); oznaczyć boxy jednej klasy „karta" w LabelImg
  lub CVAT; eksport do formatu YOLO; dla E6 i metryki end-to-end — dodatkowo przypisać
  każdemu boxowi kartę (52 klasy).
- **Rezultat:** anotacje w `data/own/`.
- **Gotowe, gdy:** każda widoczna karta na scenie ma box i etykietę.
- **Zależy od:** F7-T5

#### F7-T8 — Etykietowanie wycinków klasyfikacyjnych · 2 h
- **Cel:** ~300–400 wycinków z poprawnymi etykietami.
- **Kroki:** wycinki z pipeline'u; wstępne etykiety z modelu, **ręczna weryfikacja każdej**;
  dodatkowo ręczna kontrola losowej próbki; metodę odnotować (obciążenie na korzyść modelu).
- **Rezultat:** zbiór klasyfikacyjny własny.
- **Gotowe, gdy:** każda z 52 klas ma kilka reprezentantów.
- **Zależy od:** F7-T5

#### F7-T9 — Podział własny rozłączny na poziomie sesji · 1 h
- **Cel:** `test_own` i mała porcja do dostrajania (E11) bez przecieku.
- **Kroki:** `data/splits/own_test_v1.json`; wydzielenie sesji przeznaczonych do dostrajania
  w E11 — rozłącznych z `test_own`.
- **Rezultat:** plik podziału własnego.
- **Gotowe, gdy:** żadna sesja nie występuje w dwóch podziałach.
- **Zależy od:** F7-T6…T8

#### F7-T10 — Pierwsza ewaluacja na zbiorze własnym · 1 h
- **Cel:** punkt odniesienia dla E11.
- **Kroki:** `evaluate.py` na `test_own` i na `test_public` tym samym modelem; zapis obu wyników.
- **Rezultat:** wpis w `EXPERIMENTS.md`.
- **Gotowe, gdy:** oba wyniki zapisane w katalogu przebiegu.
- **Zależy od:** F7-T9

**Kryterium przejścia:** każda z 52 klas ma reprezentantów w `test_own`; metadane kompletne.

---

## FAZA 8 — Confidence, kalibracja i obsługa błędów

**Cel fazy:** system, który odmawia odpowiedzi zamiast zgadywać, z progiem wyznaczonym
procedurą opartą na danych (cele C5, C9, B6).

**Wejście:** ukończona F6.

**Szacunek:** 10–12 h

### Postęp

- [ ] F8-T1 — Pomiar kalibracji (ECE, wykres niezawodności)
- [ ] F8-T2 — Temperature scaling
- [ ] F8-T3 — Testy kalibracji
- [ ] F8-T4 — Krzywa ryzyko–pokrycie i wybór progu
- [ ] F8-T5 — Porównanie miar niepewności
- [ ] F8-T6 — Próg w pipelinie
- [ ] F8-T7 — Obsługa sytuacji błędnych
- [ ] F8-T8 — Scenariusze praktyczne P5–P8

### Zadania

#### F8-T1 — Pomiar kalibracji · 2 h
- **Cel:** zmierzyć, na ile surowy softmax jest przesadnie pewny.
- **Kroki:** `classification/calibration.py`: ECE z koszykami (liczba w konfiguracji),
  wykres niezawodności; pomiar na zbiorze walidacyjnym.
- **Rezultat:** ECE przed kalibracją, wykres.
- **Gotowe, gdy:** wykres i ECE zapisane w katalogu przebiegu.
- **Zależy od:** F4-T9

#### F8-T2 — Temperature scaling · 2 h
- **Cel:** poprawić kalibrację bez zmiany dokładności.
- **Kroki:** dopasowanie `T` na **walidacji** (minimalizacja NLL); zapis `T` obok wag
  modelu; ponowny pomiar ECE.
- **Rezultat:** wartość `T`, ECE po kalibracji.
- **Gotowe, gdy:** dokładność przed i po jest identyczna.
- **Zależy od:** F8-T1

#### F8-T3 — Testy kalibracji · 1 h
- **Cel:** zabezpieczyć poprawność implementacji.
- **Kroki:** testy: skalowanie nie zmienia argmax; ECE = 0 dla przypadku idealnego.
- **Rezultat:** `tests/unit/test_calibration.py`.
- **Gotowe, gdy:** testy przechodzą.
- **Zależy od:** F8-T2

#### F8-T4 — Krzywa ryzyko–pokrycie i wybór progu · 2 h
- **Cel:** próg wynikający z danych, nie przyjęty arbitralnie.
- **Kroki:** procedura z sekcji 9.4: progi 0,50–0,99, pokrycie i dokładność warunkowa na
  walidacji; krzywa; wybór najniższego progu spełniającego cel jakościowy; jednorazowa
  weryfikacja na teście.
- **Rezultat:** wartość progu z uzasadnieniem, wykres.
- **Gotowe, gdy:** próg zapisany w `configs/pipeline/default.yaml` z odwołaniem do przebiegu.
- **Zależy od:** F8-T2

#### F8-T5 — Porównanie miar niepewności · 1 h
- **Cel:** sprawdzić, czy margines dwóch najlepszych klas jest lepszym sygnałem niż maksimum.
- **Kroki:** krzywe ryzyko–pokrycie dla obu miar na jednym wykresie.
- **Rezultat:** wynik do E9.
- **Gotowe, gdy:** wybrana miara uzasadniona porównaniem.
- **Zależy od:** F8-T4

#### F8-T6 — Próg w pipelinie · 1 h
- **Cel:** decyzja `PEWNA` / `NIEPEWNA` w wyniku systemu.
- **Kroki:** moduł decyzyjny w pipelinie; przy niepewności zwracane dwie najlepsze hipotezy.
- **Rezultat:** status w `CardResult`.
- **Gotowe, gdy:** karta poniżej progu ma status `NIEPEWNA` i nie jest raportowana jako rozpoznana.
- **Zależy od:** F8-T4, F6-T2

#### F8-T7 — Obsługa sytuacji błędnych · 2 h
- **Cel:** reakcje systemu z katalogu w sekcji 9.6.
- **Kroki:** brak kart, karta na krawędzi kadru, nachodzenie, słabe światło, rozmycie,
  obiekt niebędący kartą, brak wag modelu — każda sytuacja z jednoznacznym komunikatem
  w `FrameResult.warnings`.
- **Rezultat:** obsługa w pipelinie + testy.
- **Gotowe, gdy:** każda sytuacja z tabeli ma test lub scenariusz praktyczny.
- **Zależy od:** F8-T6, F3-T9

#### F8-T8 — Scenariusze praktyczne P5–P8 · 1 h
- **Cel:** sprawdzić zachowanie na prawdziwych trudnych przypadkach.
- **Kroki:** P5 (słabe światło), P6 (kąt > 45°), P7 (rewers), P8 (telefon, kubek);
  protokół w `JOURNAL.md`.
- **Rezultat:** protokół.
- **Gotowe, gdy:** P7 i P8 nie dają fałszywych rozpoznań z wysoką pewnością.
- **Zależy od:** F8-T7

**Kryterium przejścia:** próg pochodzi z udokumentowanej procedury; P7 i P8 przechodzą.

---

## FAZA 9 — Framework eksperymentalny i eksperymenty klasyfikacji

**Cel fazy:** eksperymenty E1, E2, E3, E4, E8 z kompletem materiałów do rozdziału 7 pracy.

**Wejście:** ukończone F4 i F8 (dla E4 w wariancie `test_own` — także F7).

**Szacunek:** 20–26 h (znaczna część to oczekiwanie na treningi)

**Zasady wspólne (sekcja 11.1):** jedna zmienna na eksperyment; wspólny zbiór testowy;
3 ziarna dla E1, E3, E4; zapis konfiguracji i środowiska; wynik negatywny to też wynik.

### Postęp

- [ ] F9-T1 — Skrypt `run_experiment.py`
- [ ] F9-T2 — Skrypt `make_report.py`
- [ ] F9-T3 — E1: architektury klasyfikatora
- [ ] F9-T4 — E2: transfer learning
- [ ] F9-T5 — E3: dekompozycja problemu
- [ ] F9-T6 — E4: augmentacja
- [ ] F9-T7 — E8: rozdzielczość wejścia
- [ ] F9-T8 — Rejestr `EXPERIMENTS.md`
- [ ] F9-T9 — Analiza błędów klasyfikacji

### Zadania

#### F9-T1 — Skrypt `run_experiment.py` · 3 h
- **Cel:** dziesiątki przebiegów uruchamiane jednym poleceniem, bez ręcznych pomyłek.
- **Kroki:** definicja eksperymentu w `configs/experiments/eNN_*.yaml` (konfiguracja bazowa
  + lista wariantów + lista ziaren); uruchomienie siatki; pomijanie przebiegów już
  ukończonych (wznawianie po przerwie).
- **Rezultat:** `scripts/run_experiment.py`.
- **Gotowe, gdy:** mini-eksperyment (2 warianty × 2 ziarna × 1 epoka) tworzy 4 katalogi.
- **Zależy od:** F4-T8

#### F9-T2 — Skrypt `make_report.py` · 3 h
- **Cel:** tabele i wykresy publikacyjne generowane z zapisanych wyników, nie ręcznie.
- **Kroki:** agregacja `summary.json` z przebiegów eksperymentu; średnia ± odchylenie;
  `evaluation/plots.py` — wykresy w PDF/PNG o czcionce dopasowanej do pracy; tabele
  w Markdown i CSV.
- **Rezultat:** `scripts/make_report.py`, `src/cardvision/evaluation/plots.py`.
- **Gotowe, gdy:** raport mini-eksperymentu z F9-T1 generuje tabelę i wykres.
- **Zależy od:** F9-T1

#### F9-T3 — E1: architektury klasyfikatora · 4 h
- **Cel:** wybrać model o najlepszym stosunku jakości do kosztu (B1).
- **Kroki:** SimpleCNN / ResNet18 / MobileNetV3-Small × 3 ziarna; accuracy, macro-F1, czas
  inferencji, czas treningu, liczba parametrów.
- **Rezultat:** tabela średnia ± odchylenie; wykres dokładność vs czas inferencji.
- **Gotowe, gdy:** 9 przebiegów zakończonych, raport wygenerowany, wniosek spisany.
- **Zależy od:** F9-T2

#### F9-T4 — E2: transfer learning · 3 h
- **Cel:** ilościowo uzasadnić użycie transfer learningu (B5).
- **Kroki:** ResNet18: od zera / zamrożony backbone / pełny fine-tuning; powtórzenie na 25%
  zbioru treningowego.
- **Rezultat:** krzywe uczenia na wspólnym wykresie; tabela wyników końcowych.
- **Gotowe, gdy:** 6 wariantów zakończonych, wniosek o zależności od rozmiaru danych.
- **Zależy od:** F9-T2

#### F9-T5 — E3: dekompozycja problemu · 4 h
- **Cel:** rozstrzygnąć, czy dekompozycja na figurę i kolor pomaga (B2).
- **Kroki:** A (52 klasy) / B (dwa modele 13 + 4) / C (multi-head) × 3 ziarna; dokładność
  karty, figury, koloru; czas inferencji; sprawdzenie, czy w B dokładność karty ≈ iloczyn
  dokładności składowych.
- **Rezultat:** tabela; macierze pomyłek 13×13 i 4×4.
- **Gotowe, gdy:** 9 przebiegów zakończonych, wniosek spisany.
- **Zależy od:** F9-T2

#### F9-T6 — E4: augmentacja · 4 h
- **Cel:** uzasadnić politykę augmentacji pomiarem, w tym szkodliwość odbicia lustrzanego.
- **Kroki:** brak / geometryczna / fotometryczna / pełna / pełna z odbiciem × 3 ziarna;
  ewaluacja osobno na `test_public` i `test_own`.
- **Rezultat:** wykres słupkowy z dwiema seriami.
- **Gotowe, gdy:** 15 przebiegów zakończonych, wniosek spisany.
- **Zależy od:** F9-T2 (seria `test_own` — F7)

#### F9-T7 — E8: rozdzielczość wejścia · 2 h
- **Cel:** wybrać rozdzielczość produkcyjną z jawnym kompromisem jakość–szybkość.
- **Kroki:** ResNet18 przy 64×46, 96×69, 128×91, 160×114, 224×160, 320×229; accuracy, czas
  inferencji, pamięć.
- **Rezultat:** wykres dwuosiowy.
- **Gotowe, gdy:** 6 przebiegów zakończonych, wybrana rozdzielczość uzasadniona.
- **Zależy od:** F9-T2

#### F9-T8 — Rejestr `EXPERIMENTS.md` · 2 h
- **Cel:** szkielet rozdziału eksperymentalnego pracy.
- **Kroki:** dla każdego eksperymentu: hipoteza, warianty, kluczowy wynik, wniosek
  (potwierdzenie / odrzucenie hipotezy), ścieżki do katalogów.
- **Rezultat:** uzupełniony `docs/EXPERIMENTS.md`.
- **Gotowe, gdy:** każda liczba w rejestrze wskazuje katalog źródłowy.
- **Zależy od:** F9-T3…T7

#### F9-T9 — Analiza błędów klasyfikacji · 2 h
- **Cel:** materiał do rozdziału 8 (B8).
- **Kroki:** najczęściej mylone pary i ich wyjaśnienie wizualne; pomyłki figury vs koloru;
  pomyłki w obrębie barwy vs przez barwę; klasy systematycznie słabe vs ich liczność.
- **Rezultat:** opis z przykładami w `EXPERIMENTS.md`.
- **Gotowe, gdy:** każde z pytań z sekcji 10.4 ma odpowiedź popartą danymi.
- **Zależy od:** F9-T3, F9-T5

**Kryterium przejścia:** każdy eksperyment ma w `experiments/` konfigurację, metryki i wykres.

---

## FAZA 10 — Eksperymenty systemowe

**Cel fazy:** eksperymenty dotyczące całego systemu (E5, E6, E7, E9, E10, E11, E12)
i wybór konfiguracji produkcyjnej na podstawie pomiarów.

**Wejście:** ukończone F7, F8, F9.

**Szacunek:** 20–26 h

### Postęp

- [ ] F10-T1 — Metryka end-to-end
- [ ] F10-T2 — E5: korekcja perspektywy
- [ ] F10-T3 — E7: detektor konturowy vs YOLO
- [ ] F10-T4 — Trening YOLO z 52 klasami
- [ ] F10-T5 — E6: architektura jedno- vs dwuetapowa
- [ ] F10-T6 — E9: opracowanie kalibracji i progu
- [ ] F10-T7 — E10a/b/c: odporność systemu
- [ ] F10-T8 — E11: domain gap
- [ ] F10-T9 — E12: dane syntetyczne vs rzeczywiste
- [ ] F10-T10 — Wybór konfiguracji produkcyjnej

### Zadania

#### F10-T1 — Metryka end-to-end · 3 h
- **Cel:** główna liczba pracy — skuteczność całego systemu (sekcja 10.3).
- **Kroki:** `evaluation/metrics_e2e.py`: karta poprawna = wykryta (IoU ≥ 0,5) + poprawnie
  sklasyfikowana + pewność powyżej progu; dodatkowo odsetki pominiętych, fałszywych
  i odrzuconych; latencja z rozbiciem na etapy; testy na ręcznie policzonych przykładach.
- **Rezultat:** moduł + `tests/unit/test_metrics_e2e.py`.
- **Gotowe, gdy:** testy przechodzą.
- **Zależy od:** F5-T7, F4-T10

#### F10-T2 — E5: korekcja perspektywy · 3 h
- **Cel:** zdecydować, czy prostowanie trafia do wersji produkcyjnej (B3).
- **Kroki:** ten sam model z prostowaniem i bez (trening i test w obu wariantach);
  rozbicie wyników na przedziały kąta z `sessions.csv`; czas przetwarzania; odsetek udanych
  prostowań.
- **Rezultat:** wykres dokładności w funkcji kąta (dwie serie); tabela kosztu czasowego.
- **Gotowe, gdy:** decyzja produkcyjna poparta wynikiem.
- **Zależy od:** F10-T1, F7

#### F10-T3 — E7: detektor konturowy vs YOLO · 3 h
- **Cel:** empiryczne uzasadnienie użycia uczenia maszynowego do detekcji.
- **Kroki:** oba detektory na tym samym zbiorze; precision, recall, mAP@50, FPS z rozbiciem
  na warunki akwizycji; przykłady błędów obu metod.
- **Rezultat:** tabela z podziałem na warunki; ilustracje.
- **Gotowe, gdy:** wniosek o warunkach, w których klasyczne CV wystarcza, a w których nie.
- **Zależy od:** F10-T1, F7

#### F10-T4 — Trening YOLO z 52 klasami · 2 h
- **Cel:** model jednoetapowy do porównania w E6.
- **Kroki:** konwersja zbioru detekcyjnego z zachowaniem 52 klas; trening w warunkach
  porównywalnych z F5-T5.
- **Rezultat:** wagi i katalog przebiegu.
- **Gotowe, gdy:** model zwraca karty z klasą.
- **Zależy od:** F5-T1

#### F10-T5 — E6: architektura jedno- vs dwuetapowa · 3 h
- **Cel:** uzasadnić kluczową decyzję architektoniczną pomiarem (B4).
- **Kroki:** oba systemy na tym samym zbiorze testowym; metryka end-to-end, FPS, łączny
  czas treningu.
- **Rezultat:** tabela; wykres dokładność end-to-end vs FPS.
- **Gotowe, gdy:** wniosek spisany niezależnie od tego, który wariant wygrał.
- **Zależy od:** F10-T1, F10-T4

#### F10-T6 — E9: opracowanie kalibracji i progu · 2 h
- **Cel:** domknięcie E9 materiałami do pracy.
- **Kroki:** wykresy niezawodności przed i po; krzywa ryzyko–pokrycie z zaznaczonym progiem;
  porównanie miar niepewności; liczba błędów zaakceptowanych.
- **Rezultat:** wykresy i tabela z `make_report.py`.
- **Gotowe, gdy:** wybrana wartość progu ma pełne uzasadnienie.
- **Zależy od:** F8-T5

#### F10-T7 — E10a/b/c: odporność systemu · 4 h
- **Cel:** granice stosowalności systemu — materiał do rozdziału o ograniczeniach.
- **Kroki:**
  - **E10a:** wyniki wg oświetlenia; osobno dokładność figury i koloru; wariant z CLAHE /
    balansem bieli (`preprocessing/enhance.py`).
  - **E10b:** wyniki wg przedziału kąta, z korekcją i bez.
  - **E10c:** 1/2/5/10 kart i układy nachodzące: recall, dokładność end-to-end, czas klatki.
- **Rezultat:** wykresy i tabele dla trzech podeksperymentów.
- **Gotowe, gdy:** dla każdej zmiennej wyznaczona jest granica, przy której system przestaje
  działać poprawnie.
- **Zależy od:** F10-T1, F7-T6

#### F10-T8 — E11: domain gap · 3 h
- **Cel:** ilościowa ocena przenoszalności modelu z danych publicznych na rzeczywiste (B7).
- **Kroki:** (a) model tylko na danych publicznych; (b) dostrojony na małej porcji danych
  własnych (rozłącznej z `test_own` na poziomie sesji); ewaluacja obu na `test_public`
  i `test_own`; przykłady obrazów błędnych tylko na zbiorze własnym.
- **Rezultat:** wykres słupkowy; ilustracje.
- **Gotowe, gdy:** różnica (domain gap) i jej zmiana po dostrojeniu zmierzone.
- **Zależy od:** F7-T9

#### F10-T9 — E12: dane syntetyczne vs rzeczywiste · 2 h
- **Cel:** ocenić opłacalność generatora scen.
- **Kroki:** trzy treningi YOLO o zrównanej liczbie obrazów: tylko rzeczywiste / tylko
  syntetyczne / mieszane; ewaluacja na rzeczywistym zbiorze testowym.
- **Rezultat:** tabela; przykłady błędów modelu uczonego wyłącznie na syntetykach.
- **Gotowe, gdy:** wniosek praktyczny spisany.
- **Zależy od:** F5-T3, F7-T7

#### F10-T10 — Wybór konfiguracji produkcyjnej · 2 h
- **Cel:** konfiguracja systemu wynikająca z pomiarów, nie z założeń.
- **Kroki:** uzupełnienie `EXPERIMENTS.md`; aktualizacja `configs/pipeline/default.yaml`
  (model, rozdzielczość, korekcja, detektor, próg) z komentarzem wskazującym eksperyment
  uzasadniający każdą wartość.
- **Rezultat:** finalna konfiguracja produkcyjna.
- **Gotowe, gdy:** każda decyzja z sekcji 3 i 4 planu ma pomiar, który ją potwierdza lub podważa.
- **Zależy od:** F10-T2…T9

**Kryterium przejścia:** wszystkie zaplanowane eksperymenty wykonane i opisane.

---

## FAZA 11 — Aplikacja GUI

**Cel fazy:** desktopowa aplikacja prezentująca działanie systemu (C8). GUI nie zawiera
logiki — wywołuje wyłącznie pipeline.

**Wejście:** ukończona F6 (najlepiej także F8 i F10 — wtedy GUI pokazuje wersję produkcyjną).

**Szacunek:** 16–20 h

### Postęp

- [ ] F11-T1 — Szkielet okna
- [ ] F11-T2 — Tryb obrazu
- [ ] F11-T3 — Panel wykrytych kart
- [ ] F11-T4 — Panel statystyk
- [ ] F11-T5 — Panel ustawień
- [ ] F11-T6 — Historia i eksport
- [ ] F11-T7 — Obsługa błędów w GUI
- [ ] F11-T8 — Dopracowanie wyglądu
- [ ] F11-T9 — Zrzuty ekranu

### Zadania

#### F11-T1 — Szkielet okna · 3 h
- **Cel:** układ z sekcji 12.2.
- **Kroki:** `app/main_window.py`: pasek przycisków, podgląd, panel kart, panel statystyk,
  pasek historii; `scripts/run_app.py`.
- **Rezultat:** puste okno o docelowym układzie.
- **Gotowe, gdy:** `python scripts/run_app.py` otwiera okno.
- **Zależy od:** F6

#### F11-T2 — Tryb obrazu · 2,5 h
- **Cel:** analiza wybranego pliku.
- **Kroki:** okno wyboru pliku; wywołanie pipeline'u; wyświetlenie obrazu z ramkami
  i etykietami (rysowanie przez `io/writers.py`).
- **Rezultat:** działający tryb obrazu.
- **Gotowe, gdy:** wynik w GUI jest identyczny z wynikiem `predict.py` dla tego samego pliku.
- **Zależy od:** F11-T1

#### F11-T3 — Panel wykrytych kart · 2 h
- **Cel:** lista kart z pewnością i statusem.
- **Kroki:** widżet listy: symbol karty, pewność, status `PEWNA`/`NIEPEWNA`, dla niepewnych
  druga hipoteza; zaznaczenie karty podświetla ramkę na podglądzie.
- **Rezultat:** `app/widgets/`.
- **Gotowe, gdy:** lista odpowiada ramkom na obrazie.
- **Zależy od:** F11-T2

#### F11-T4 — Panel statystyk · 2 h
- **Cel:** liczby z działającego systemu, nie z dokumentu.
- **Kroki:** liczba kart, pewne, średnia pewność, czasy etapów, łączny czas, FPS (w trybie kamery).
- **Rezultat:** widżet statystyk.
- **Gotowe, gdy:** wartości pochodzą z `FrameResult`.
- **Zależy od:** F11-T2

#### F11-T5 — Panel ustawień · 3 h
- **Cel:** parametry eksperymentów sterowane z GUI (sekcja 12.3) — narzędzie demonstracyjne
  na obronę.
- **Kroki:** wybór klasyfikatora i detektora, przełącznik korekcji perspektywy, suwak progu
  pewności (z zaznaczoną wartością z E9), suwak progu detekcji; przebudowa pipeline'u po zmianie.
- **Rezultat:** okno ustawień.
- **Gotowe, gdy:** zmiana detektora w GUI daje ten sam wynik co zmiana w YAML.
- **Zależy od:** F11-T2, F6-T3

#### F11-T6 — Historia i eksport · 2,5 h
- **Cel:** zapis każdej analizy (sekcja 12.5).
- **Kroki:** dopisywanie rekordów do `results/history.jsonl`; pasek historii z możliwością
  powrotu do analizy; eksport do CSV.
- **Rezultat:** historia w GUI.
- **Gotowe, gdy:** historia przetrwa ponowne uruchomienie aplikacji.
- **Zależy od:** F11-T2

#### F11-T7 — Obsługa błędów w GUI · 1,5 h
- **Cel:** aplikacja nie zamyka się przy błędzie.
- **Kroki:** czytelne komunikaty: brak wag modelu, brak pliku, uszkodzony obraz.
- **Rezultat:** okna dialogowe błędów.
- **Gotowe, gdy:** każdy z trzech przypadków daje komunikat zamiast awarii.
- **Zależy od:** F11-T2

#### F11-T8 — Dopracowanie wyglądu · 2 h
- **Cel:** profesjonalna prezentacja.
- **Kroki:** arkusz stylów, ikony, czytelna typografia, spójne kolory statusów.
- **Rezultat:** finalny wygląd aplikacji.
- **Gotowe, gdy:** interfejs jest czytelny w rozdzielczości projektora.
- **Zależy od:** F11-T1…T7

#### F11-T9 — Zrzuty ekranu · 1 h
- **Cel:** ilustracje do README i pracy.
- **Kroki:** zrzuty dla typowych przypadków (pewne, niepewne, wiele kart) do `docs/img/`.
- **Rezultat:** zrzuty ekranu.
- **Gotowe, gdy:** zrzuty pokazują wszystkie panele.
- **Zależy od:** F11-T8

**Kryterium przejścia:** aplikacja uruchamia się jedną komendą, analizuje obraz, pokazuje
wyniki i statystyki, zapisuje historię.

---

## FAZA 12 — Tryb kamery i praca w czasie rzeczywistym

**Cel fazy:** rozpoznawanie kart na żywo ze zmierzonym FPS (C1, C7), bez zamrażania
interfejsu.

**Wejście:** ukończona F11.

**Szacunek:** 10–14 h

### Postęp

- [ ] F12-T1 — Wątek akwizycji
- [ ] F12-T2 — Wątek inferencji
- [ ] F12-T3 — Podgląd na żywo w GUI
- [ ] F12-T4 — Licznik FPS i czasy etapów
- [ ] F12-T5 — Pomijanie klatek nieostrych
- [ ] F12-T6 — Wybór kamery i rozdzielczości
- [ ] F12-T7 — Scenariusze P9 i P10
- [ ] F12-T8 — Pomiar FPS

### Zadania

#### F12-T1 — Wątek akwizycji · 2,5 h
- **Cel:** odczyt klatek bez blokowania GUI.
- **Kroki:** `app/camera_worker.py`: `QThread` czytający z `image_source`; kolejka
  o rozmiarze 1 z nadpisywaniem (pomijanie zaległych klatek).
- **Rezultat:** wątek akwizycji.
- **Gotowe, gdy:** GUI reaguje płynnie podczas odczytu z kamery.
- **Zależy od:** F11, F3-T1

#### F12-T2 — Wątek inferencji · 2,5 h
- **Cel:** pipeline w tle, wyniki przez sygnały Qt.
- **Kroki:** `app/inference_worker.py`: pobieranie najnowszej klatki, pipeline, sygnał
  z `FrameResult`.
- **Rezultat:** wątek inferencji.
- **Gotowe, gdy:** żadne wywołanie pipeline'u nie odbywa się w wątku GUI.
- **Zależy od:** F12-T1

#### F12-T3 — Podgląd na żywo w GUI · 2 h
- **Cel:** tryb kamery dla użytkownika.
- **Kroki:** przycisk „Kamera"; wyświetlanie klatek z nałożonymi wynikami; start/stop.
- **Rezultat:** tryb kamery.
- **Gotowe, gdy:** karty są rozpoznawane na żywo.
- **Zależy od:** F12-T2

#### F12-T4 — Licznik FPS i czasy etapów · 1,5 h
- **Cel:** pomiar wydajności w czasie rzeczywistym.
- **Kroki:** FPS (średnia krocząca), czasy etapów w panelu statystyk.
- **Rezultat:** statystyki na żywo.
- **Gotowe, gdy:** wartości aktualizują się w trakcie działania.
- **Zależy od:** F12-T3

#### F12-T5 — Pomijanie klatek nieostrych · 1 h
- **Cel:** stabilniejszy wynik bez zbędnej inferencji.
- **Kroki:** filtr z `quality.py` przed pipeline'em; próg w konfiguracji.
- **Rezultat:** filtr w wątku inferencji.
- **Gotowe, gdy:** przy szybkim ruchu kamery klatki rozmyte nie generują wyników.
- **Zależy od:** F12-T2

#### F12-T6 — Wybór kamery i rozdzielczości · 1 h
- **Cel:** konfigurowalność sprzętowa.
- **Kroki:** lista dostępnych kamer i rozdzielczości w ustawieniach.
- **Rezultat:** opcje w panelu ustawień.
- **Gotowe, gdy:** zmiana kamery działa bez restartu aplikacji.
- **Zależy od:** F12-T3

#### F12-T7 — Scenariusze P9 i P10 · 1,5 h
- **Cel:** stabilność i odporność trybu kamery.
- **Kroki:** P9 — 60 s ciągłej pracy z obserwacją zużycia pamięci; P10 — uruchomienie bez
  podłączonej kamery.
- **Rezultat:** protokół w `JOURNAL.md`.
- **Gotowe, gdy:** brak wzrostu pamięci; brak kamery daje komunikat, nie awarię.
- **Zależy od:** F12-T3

#### F12-T8 — Pomiar FPS · 1 h
- **Cel:** dane do celu C7.
- **Kroki:** pomiar FPS dla 1, 5 i 10 kart w kadrze; zapis z konfiguracją i sprzętem.
- **Rezultat:** wyniki w `EXPERIMENTS.md`.
- **Gotowe, gdy:** FPS zmierzony i zapisany; jeśli < 15 — udokumentowany i rozważone X4.
- **Zależy od:** F12-T4

**Kryterium przejścia:** 60 s pracy bez wycieku pamięci i bez zamrażania interfejsu;
FPS zmierzony.

---

## FAZA 13 — Testy i jakość kodu

**Cel fazy:** domknięcie testów i uporządkowanie kodu przed finalizacją. Testy krytyczne
(`labels`, geometria, metryki, kalibracja) powstały wcześniej — ta faza je uzupełnia.

**Wejście:** ukończona F12 (lub F10, jeśli F11–F12 zostały ograniczone).

**Szacunek:** 8–12 h

### Postęp

- [ ] F13-T1 — Uzupełnienie testów jednostkowych
- [ ] F13-T2 — Testy integracyjne
- [ ] F13-T3 — Pomiar pokrycia testami
- [ ] F13-T4 — Linter, formater, martwy kod
- [ ] F13-T5 — Pełny przebieg scenariuszy P1–P10
- [ ] F13-T6 — Weryfikacja odtwarzalności

### Zadania

#### F13-T1 — Uzupełnienie testów jednostkowych · 3 h
- **Cel:** pokrycie wszystkich modułów z tabeli w sekcji 15.2.
- **Kroki:** porównać istniejące testy z tabelą; dopisać brakujące (np. `postprocess.py`, NMS).
- **Rezultat:** kompletne `tests/unit/`.
- **Gotowe, gdy:** każdy moduł z tabeli 15.2 ma testy.
- **Zależy od:** —

#### F13-T2 — Testy integracyjne · 2,5 h
- **Cel:** pokrycie tabeli z sekcji 15.3.
- **Kroki:** wymienność implementacji; każda konfiguracja z `configs/pipeline/` uruchamia
  się poprawnie; determinizm (dwa uruchomienia z tym samym ziarnem → ten sam wynik).
- **Rezultat:** kompletne `tests/integration/`.
- **Gotowe, gdy:** każdy test z tabeli 15.3 istnieje i przechodzi.
- **Zależy od:** —

#### F13-T3 — Pomiar pokrycia testami · 1,5 h
- **Cel:** wykryć luki w modułach krytycznych.
- **Kroki:** `pytest --cov`; uzupełnienie testów tam, gdzie moduły krytyczne mają luki.
- **Rezultat:** raport pokrycia.
- **Gotowe, gdy:** moduły krytyczne (`labels`, `geometry`, `evaluation`, `calibration`) mają
  wysokie pokrycie.
- **Zależy od:** F13-T1, F13-T2

#### F13-T4 — Linter, formater, martwy kod · 2 h
- **Cel:** czytelny, spójny kod.
- **Kroki:** konfiguracja lintera i formatera w `pyproject.toml`; uruchomienie na całym kodzie;
  usunięcie nieużywanego kodu i zakomentowanych fragmentów.
- **Rezultat:** kod bez ostrzeżeń lintera.
- **Gotowe, gdy:** linter nie zgłasza błędów; `pytest` przechodzi.
- **Zależy od:** —

#### F13-T5 — Pełny przebieg scenariuszy P1–P10 · 1,5 h
- **Cel:** końcowy protokół testów praktycznych.
- **Kroki:** wykonać wszystkie scenariusze z sekcji 15.4 na wersji finalnej.
- **Rezultat:** protokół (do załącznika pracy).
- **Gotowe, gdy:** wszystkie scenariusze udokumentowane z wynikiem.
- **Zależy od:** F13-T4

#### F13-T6 — Weryfikacja odtwarzalności · 1,5 h
- **Cel:** dowód spełnienia celu M1.
- **Kroki:** powtórzyć wybrany przebieg (np. jeden z E1) z zapisanego `config.yaml`;
  porównać metryki.
- **Rezultat:** wynik porównania w `JOURNAL.md`.
- **Gotowe, gdy:** wynik jest identyczny (lub różnica mieści się w udokumentowanym
  niedeterminizmie GPU).
- **Zależy od:** —

**Kryterium przejścia:** `pytest` przechodzi w całości; P1–P10 udokumentowane; eksperyment
odtworzony.

---

## FAZA 14 — Rozszerzenia `[OPCJONALNA]`

**Cel fazy:** funkcje podnoszące wartość projektu, **wyłącznie po ukończeniu F1–F13**.
Każde rozszerzenie na osobnej gałęzi; scalenie dopiero po potwierdzeniu, że P1–P10 nadal
przechodzą.

**Kolejność rekomendowana:** X1 → X2 → X5, pozostałe w miarę czasu.

### Postęp

- [ ] F14-T1 — X1: analiza pokrycia talii
- [ ] F14-T2 — X2: głosowanie po klatkach w trybie wideo
- [ ] F14-T3 — X5: mapy istotności (Grad-CAM)
- [ ] F14-T4 — X3: generalizacja na talię o odmiennym kroju
- [ ] F14-T5 — X4: eksport do ONNX
- [ ] F14-T6 — X6: wykrywanie rewersów
- [ ] F14-T7 — X7: tryb wsadowy z raportem PDF
- [ ] F14-T8 — X8: rozpoznawanie układów pokerowych

### Zadania

#### F14-T1 — X1: analiza pokrycia talii · 2–3 h
- **Cel:** prezentacja „rozpoznano N / 52" i dodatkowa kontrola poprawności.
- **Kroki:** zliczanie unikalnych kart w sesji; lista kart brakujących; ostrzeżenie
  o duplikacie (ta sama karta dwukrotnie = sygnał błędu); panel w GUI.
- **Gotowe, gdy:** rozłożenie pełnej talii daje 52 / 52 lub listę brakujących.

#### F14-T2 — X2: głosowanie po klatkach w trybie wideo · 4–6 h
- **Cel:** stabilny wynik w trybie kamery i wyższy FPS.
- **Kroki:** śledzenie kart między klatkami (dopasowanie po IoU); agregacja predykcji
  z ostatnich N klatek; klasyfikacja tylko nowych obiektów; pomiar wpływu na dokładność
  i FPS jako dodatkowy eksperyment.
- **Gotowe, gdy:** etykiety nie „migoczą", a wpływ jest zmierzony.

#### F14-T3 — X5: mapy istotności (Grad-CAM) · 3–4 h
- **Cel:** sprawdzić, czy sieć patrzy na indeks narożny (hipoteza z sekcji 1.2).
- **Kroki:** Grad-CAM dla modelu produkcyjnego; wizualizacje dla poprawnych i błędnych
  klasyfikacji.
- **Gotowe, gdy:** zestaw ilustracji z wnioskiem do rozdziału 8.

#### F14-T4 — X3: generalizacja na talię o odmiennym kroju · 3–4 h
- **Cel:** pomiar generalizacji międzytaliowej.
- **Warunek:** F1-T6 wykazał drugą talię o odmiennym kroju.
- **Kroki:** ewaluacja na nagraniach drugiej talii; porównanie z talią główną.
- **Gotowe, gdy:** różnica zmierzona i opisana.

#### F14-T5 — X4: eksport do ONNX · 4–6 h
- **Cel:** przyspieszenie inferencji, jeśli F12-T8 nie osiągnął 15 FPS.
- **Kroki:** eksport klasyfikatora i detektora; inferencja przez ONNX Runtime; pomiar.
- **Gotowe, gdy:** zmierzone przyspieszenie i zgodność wyników z PyTorch.

#### F14-T6 — X6: wykrywanie rewersów · 2–3 h
- **Cel:** jawna klasa „rewers" zamiast odrzucenia.
- **Kroki:** zebranie przykładów rewersów; klasa dodatkowa lub osobny klasyfikator binarny.
- **Gotowe, gdy:** P7 zwraca „rewers" zamiast „niepewna".

#### F14-T7 — X7: tryb wsadowy z raportem PDF · 3–4 h
- **Cel:** wygoda użytkowa.
- **Kroki:** analiza katalogu → raport PDF z miniaturami i listą kart.
- **Gotowe, gdy:** raport generuje się jedną komendą.

#### F14-T8 — X8: rozpoznawanie układów pokerowych · 4–6 h
- **Cel:** efekt prezentacyjny (poza tematem widzenia komputerowego — najniższy priorytet).
- **Kroki:** reguły układów na liście rozpoznanych kart; wyświetlenie w GUI.
- **Gotowe, gdy:** 5 kart w kadrze daje nazwę układu.

---

## FAZA 15 — Finalizacja i materiały do pracy

**Cel fazy:** repozytorium i materiały gotowe do obrony; każda liczba w pracy ma źródło.

**Wejście:** ukończona F13 (opcjonalnie F14).

**Szacunek:** 12–16 h

### Postęp

- [ ] F15-T1 — Finalne wykresy i tabele
- [ ] F15-T2 — Weryfikacja źródeł liczb
- [ ] F15-T3 — Rysunek architektury systemu
- [ ] F15-T4 — Uzupełnienie `README.md`
- [ ] F15-T5 — Instalacja od zera według README
- [ ] F15-T6 — Aktualizacja `THESIS_OUTLINE.md`
- [ ] F15-T7 — Uporządkowanie repozytorium
- [ ] F15-T8 — Zestawienie ograniczeń systemu
- [ ] F15-T9 — Kierunki rozwoju

### Zadania

#### F15-T1 — Finalne wykresy i tabele · 2 h
- **Cel:** spójne materiały wygenerowane jednym przebiegiem.
- **Kroki:** `make_report.py` dla wszystkich eksperymentów; jednolity styl wykresów.
- **Gotowe, gdy:** wszystkie wykresy do pracy powstają jednym poleceniem.

#### F15-T2 — Weryfikacja źródeł liczb · 2 h
- **Cel:** zasada z sekcji 19.4 — żadnej liczby bez pokrycia.
- **Kroki:** dla każdej liczby w tekście pracy wskazać katalog w `experiments/`; usunąć
  lub poprawić liczby bez źródła.
- **Gotowe, gdy:** lista kontrolna liczb jest kompletna.

#### F15-T3 — Rysunek architektury systemu · 2 h
- **Cel:** schemat pipeline'u do rozdziału 4.
- **Kroki:** diagram etapów i interfejsów wymiennych w formacie wektorowym w `docs/img/`.
- **Gotowe, gdy:** rysunek zgodny z faktycznym kodem.

#### F15-T4 — Uzupełnienie `README.md` · 1,5 h
- **Cel:** instrukcja instalacji i uruchomienia od zera.
- **Kroki:** wymagania, instalacja, pobranie danych, trening, uruchomienie CLI i GUI,
  odtworzenie eksperymentów; aktualizacja statusu projektu.
- **Gotowe, gdy:** README opisuje stan faktyczny, a nie docelowy.

#### F15-T5 — Instalacja od zera według README · 2 h
- **Cel:** dowód kompletności dokumentacji.
- **Kroki:** świeży klon repozytorium w nowym katalogu, nowe środowisko, wykonanie instrukcji
  krok po kroku; poprawienie każdej luki.
- **Gotowe, gdy:** system działa po wykonaniu wyłącznie kroków z README.

#### F15-T6 — Aktualizacja `THESIS_OUTLINE.md` · 1,5 h
- **Cel:** plan pisania pracy oparty na wynikach.
- **Kroki:** aktualizacja wersji roboczej struktury (utworzonej przed implementacją) na
  podstawie wyników; dla każdego rozdziału lista materiałów z repozytorium (wykresy,
  tabele, dokumenty).

#### F15-T7 — Uporządkowanie repozytorium · 1 h
- **Cel:** czyste repozytorium do oddania.
- **Kroki:** usunięcie plików roboczych; przegląd `.gitignore`; sprawdzenie, że wagi i dane
  nie trafiły do gita; tag wersji finalnej.
- **Gotowe, gdy:** `git status` czysty, repozytorium zdalne aktualne.

#### F15-T8 — Zestawienie ograniczeń systemu · 1,5 h
- **Cel:** uczciwa ocena rozwiązania do rozdziału 9.
- **Kroki:** granice z E10 (oświetlenie, kąt, liczba kart), domain gap z E11, jedna talia
  (jeśli X3 niewykonane), FPS.
- **Gotowe, gdy:** każde ograniczenie poparte pomiarem.

#### F15-T9 — Kierunki rozwoju · 1 h
- **Cel:** zakończenie rozdziału 9.
- **Kroki:** zestawienie na podstawie napotkanych problemów i niezrealizowanych rozszerzeń
  (architektura C, X-y niewykonane).
- **Gotowe, gdy:** każdy kierunek wynika z konkretnej obserwacji w projekcie.

**DoD projektu:** działający system, komplet eksperymentów, materiały do pracy z pokryciem
w zapisanych przebiegach, repozytorium instalowalne od zera według własnej instrukcji.

---

## Ścieżka minimalna przy braku czasu

Kolejność rezygnacji (sekcja 20.3): F14 → F12 → F11 ograniczona do trybu obrazu →
E12, E10c, E8.

**Nigdy nie rezygnować z:** F2 (audyt i poprawne podziały), testów modułów krytycznych,
eksperymentów E1–E5 i E9.
