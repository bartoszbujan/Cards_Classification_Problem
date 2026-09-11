# Rozpoznawanie kart do gry — system Computer Vision / AI

Projekt inżynierski realizujący temat **„Zastosowanie narzędzi sztucznej inteligencji do
klasyfikacji kart do gry"** (Polsko-Japońska Akademia Technik Komputerowych).

> **Status: Faza 0 — plan zaakceptowany do realizacji.**
> Kod źródłowy jeszcze nie istnieje. Sekcja „Uruchomienie" opisuje stan docelowy i będzie
> uzupełniana wraz z postępem prac.

---

## 1. O czym jest ten projekt

System wykrywa karty do gry na obrazie z pliku lub ze strumienia kamery, wycina je,
prostuje perspektywicznie i klasyfikuje do jednej z 52 klas, podając **skalibrowaną miarę
pewności**. Rozpoznania niepewne są jawnie oznaczane jako odrzucone, a nie zgadywane.

Zadanie jest nietypowo trudne w jednym konkretnym miejscu: informacja odróżniająca dwie
karty (indeks narożny, np. `Q♠` kontra `Q♣`) zajmuje kilka procent powierzchni obrazu karty,
a pary takie jak `6`/`9` czy `♠`/`♣` różnią się drobnym szczegółem kształtu. To wyznacza
wymagania co do rozdzielczości wejścia, polityki augmentacji i korekcji perspektywy — i te
wymagania są w projekcie **mierzone, a nie zakładane**.

**Praca jest zbudowana wokół dwunastu eksperymentów porównawczych, a nie wokół jednego
wytrenowanego modelu.** Każda decyzja projektowa — architektura, rozdzielczość, augmentacja,
próg pewności — ma być poparta pomiarem na wspólnym zbiorze testowym.

Klasyczne CV nie jest w tym projekcie odrzucone, lecz **świadomie użyte jako punkt
odniesienia**: eksperyment E7 porównuje detektor konturowy z siecią neuronową na tym samym
zbiorze testowym, dzięki czemu sięgnięcie po narzędzia AI jest uzasadnione pomiarem, a nie
założeniem — zob. [PROJECT_PLAN.md, sekcja 1.3](docs/PROJECT_PLAN.md#13-dlaczego-to-jest-problem-ai-a-nie-tylko-cv).

### Co wyróżnia ten plan

1. **Audyt danych z wykrywaniem przecieku** przed pierwszym treningiem — chroni przed
   wynikami zawyżonymi przez duplikaty i przez klatki z tej samej sesji po obu stronach podziału.
2. **Próg pewności wyznaczony z krzywej ryzyko–pokrycie**, a nie przyjęty arbitralnie
   (typowe „0,5, bo tak").
3. **Pomiar domain gap (E11)** — zamienia główne ograniczenie strategii danych (trening na
   danych publicznych, użycie na własnej kamerze) w wynik naukowy.

### Czego projekt świadomie nie obejmuje

Rozpoznawania rewersów (mają być wykrywane i odrzucane jako nierozpoznawalne), jokerów
i talii nietypowych, śledzenia kart w czasie, logiki reguł gry, wdrożenia mobilnego.

---

## 2. Cele

**Inżynierskie (C1–C9):** wczytywanie obrazu i strumienia z kamery, detekcja wszystkich kart
w kadrze, normalizacja geometryczna wycinków, klasyfikacja do 52 klas, skalibrowana pewność
z odrzucaniem, obsługa wielu kart i częściowych zasłonięć, czas rzeczywisty
(cel ≥ 15 FPS `[DO ZMIERZENIA]`), GUI z historią, **odmowa odpowiedzi zamiast zgadywania**.

**Badawcze (B1–B8):** porównanie co najmniej trzech architektur klasyfikatora, wpływ
dekompozycji problemu, wpływ korekcji perspektywy, porównanie architektury jedno-
i dwuetapowej, transfer learning kontra uczenie od zera, próg pewności wyznaczony z danych,
pomiar domain gap, analiza błędów na macierzy pomyłek.

**Metodologiczne (M1–M4):** odtwarzalność każdego przebiegu z zapisanej konfiguracji
i ziarna, zbiór testowy nietykany podczas strojenia, podział wykluczający przeciek
informacji, każda liczba w pracy pochodząca z zapisanego uruchomienia.

Pełne tabele celów: [PROJECT_PLAN.md, sekcja 2](docs/PROJECT_PLAN.md#2-cele-projektu).

---

## 3. Architektura

Pipeline **dwuetapowy**: detektor odpowiada wyłącznie na pytanie *gdzie jest karta*,
klasyfikator na pytanie *jaka to karta*.

```
obraz / kamera
   → preprocessing
   → detekcja kart (YOLO, 1 klasa | detektor konturowy)
   → wycięcie
   → korekcja perspektywy (opcjonalna)
   → klasyfikacja (52 klasy)
   → kalibracja pewności i decyzja accept / reject
   → agregacja wyników
   → GUI / CLI / JSON
```

Dlaczego dwuetapowo: detektor uczy się jednej klasy o bardzo wyrazistym kształcie, więc jest
tani w trenowaniu i odporny; klasyfikator dostaje wycinek o znormalizowanej geometrii, więc
pracuje w wyższej efektywnej rozdzielczości niż ta, którą karta zajmowała w oryginalnym
kadrze. Alternatywy — YOLO z 52 klasami oraz czysto klasyczny detektor konturowy — **nie są
odrzucone deklaratywnie, lecz pozostają w projekcie jako punkty odniesienia mierzone
w E6 i E7**.

Detektor, klasyfikator i korekcja perspektywy są wymienne z poziomu pliku konfiguracyjnego,
dzięki czemu przeprowadzenie eksperymentu porównawczego nie wymaga zmian w kodzie.

Uzasadnienie wyboru wraz z porównaniem odrzuconych alternatyw:
[PROJECT_PLAN.md, sekcja 3](docs/PROJECT_PLAN.md#3-analiza-i-wybór-architektury).

---

## 4. Strategia danych

Trzy źródła o **rozdzielnych rolach** — to najważniejsza decyzja metodologiczna projektu:

| Źródło | Udział | Rola | Uzasadnienie |
|---|---|---|---|
| Zbiory publiczne | ~80% | trening i walidacja | gotowe i liczne, pozwalają zacząć bez czekania na własne nagrania |
| Dane syntetyczne (kompozycja scen) | ~15% | **wyłącznie trening detektora** | dowolnie wiele scen wielokartowych z darmową i idealnie dokładną anotacją |
| Własne zdjęcia i nagrania | ~5% | **wyłącznie zbiór testowy** | mierzą to, co naprawdę interesujące: skuteczność w warunkach docelowych |

Dane syntetyczne powstają przez złożenie: wycinek karty o znanej etykiecie + losowe tło +
losowa homografia + losowe oświetlenie + losowe przesłonięcie. Bounding box wynika
z przekształcenia narożników tą samą homografią, więc anotacja jest darmowa i dokładna.
Ceną jest brak realizmu (odblaski, szum sensora, cienie kontaktowe) oraz ryzyko, że model
nauczy się artefaktów generatora — dlatego dane syntetyczne **nigdy nie trafiają do zbioru
testowego**, a ich rzeczywista wartość jest przedmiotem osobnego eksperymentu (E12).

**Dwie zasady, bez których wyniki byłyby nieprawdziwe:**
- podział train/val/test jest rozłączny **na poziomie sesji nagraniowej**, nie pojedynczej
  klatki — sąsiednie klatki tego samego nagrania są niemal identyczne, a rozdzielenie ich
  między trening i test zawyża wynik,
- przed pierwszym treningiem wykonywany jest **audyt zbioru** (duplikaty perceptualne,
  rozkład klas, poprawność etykiet), którego raport trafia do `docs/DATASET.md`.

Szczegóły: [PROJECT_PLAN.md, sekcja 5](docs/PROJECT_PLAN.md#5-dataset).

---

## 5. Część eksperymentalna

Zasady wspólne: **jedna zmienna niezależna** na eksperyment, **wspólny zbiór testowy**
w obrębie eksperymentu, zapisane ziarno losowe, a tam gdzie spodziewane różnice są małe
(E1, E3, E4) — **trzy powtórzenia z różnymi ziarnami** i raportowanie średniej wraz
z odchyleniem standardowym. Bez tego nie da się odróżnić rzeczywistej przewagi od szumu.

### 5.1 Klasyfikator

**E1 — Porównanie architektur klasyfikatora.**
Trzy modele (własna SimpleCNN, ResNet18, MobileNetV3-Small) na identycznym zbiorze, podziale
i budżecie epok; 3 modele × 3 ziarna = 9 przebiegów. Mierzona jest nie tylko dokładność, lecz
także czas inferencji i liczba parametrów. Wynik jest podstawą wyboru modelu produkcyjnego —
wskazuje najlepszy stosunek jakości do kosztu, a nie po prostu najwyższą liczbę.

**E2 — Transfer learning kontra uczenie od zera.**
ResNet18 w trzech wariantach inicjalizacji: od zera, z zamrożonym backbone'em i z pełnym
fine-tuningiem. Eksperyment powtarzany dodatkowo na 25% zbioru treningowego, żeby zmierzyć,
jak przewaga transfer learningu zależy od ilości danych. Prezentacja obejmuje krzywe uczenia,
bo istotna jest nie tylko wartość końcowa, ale też szybkość zbieżności.

**E3 — Dekompozycja problemu: 52 klasy kontra figura + kolor.**
Porównanie trzech struktur wyjścia: jeden klasyfikator 52-klasowy, dwa niezależne modele
(13 figur + 4 kolory) oraz multi-head ze wspólnym backbone'em i dwiema głowami. Hipoteza
mówi, że wygra multi-head, ponieważ wykorzystuje strukturę problemu bez podwajania kosztu
inferencji. Przy okazji weryfikowane jest przewidywanie, że w wariancie z dwoma modelami
dokładność karty jest w przybliżeniu iloczynem dokładności składowych.

**E4 — Wpływ augmentacji danych.**
Pięć zestawów: brak, tylko geometryczne, tylko fotometryczne, pełny zestaw oraz — celowo —
pełny zestaw **z błędnie włączonym odbiciem lustrzanym**. Wyniki raportowane osobno na
zbiorze publicznym i własnym, bo augmentacja fotometryczna powinna pomagać przede wszystkim
tam, gdzie oświetlenie różni się od treningowego. Wariant z odbiciem jest w planie po to, by
empirycznie potwierdzić decyzję o jego wykluczeniu — lustrzana karta to obiekt, który
w rzeczywistości nie istnieje.

**E5 — Wpływ korekcji perspektywy.**
Ten sam model oceniany z prostowaniem wycinka i bez niego, z rozbiciem wyników według
przedziałów kąta obserwacji (0–15°, 15–30°, 30–45°, powyżej 45°). Oczekiwany jest przyrost
rosnący wraz z kątem. **Wynik negatywny jest tu w pełni akceptowalny** — pod warunkiem
podania kosztu czasowego, który dzięki rezygnacji z korekcji zostaje zaoszczędzony.

**E8 — Wpływ rozdzielczości wejściowej klasyfikatora.**
Sześć rozdzielczości od 64×46 do 320×229 przy niezmienionym protokole treningu. Hipoteza
zakłada istnienie progu, poniżej którego indeks narożny przestaje być czytelny i dokładność
gwałtownie spada, a powyżej którego przyrosty są niewielkie przy kwadratowo rosnącym koszcie
obliczeniowym. Wynik daje jawny kompromis jakość–szybkość i domyka analizę rozmiaru indeksu
z rozdziału wprowadzającego.

### 5.2 Architektura systemu

**E6 — Architektura jednoetapowa kontra dwuetapowa.**
YOLO z 52 klasami trenowany na tym samym zbiorze detekcyjnym i porównany z pełnym
pipeline'em dwuetapowym na tym samym zbiorze testowym, przy pomiarze dokładności end-to-end
i FPS. To **najważniejszy eksperyment z punktu widzenia rozdziału o projekcie systemu**:
uzasadnia kluczową decyzję architektoniczną pomiarem, a nie deklaracją.

**E7 — Detektor klasyczny kontra uczony.**
Detektor konturowy (OpenCV) kontra YOLO, z wynikami rozbitymi według warunków akwizycji.
Hipoteza: metoda klasyczna dorówna sieci na jednolitym tle przy dobrym świetle, ale wyraźnie
ustąpi na tle wzorzystym i przy kartach nachodzących. Jest to bezpośrednia, empiryczna
odpowiedź na pytanie, czy uczenie maszynowe jest w tym zadaniu w ogóle potrzebne.

### 5.3 Niezawodność i odporność

**E9 — Kalibracja i wyznaczenie progu pewności.**
Pomiar błędu kalibracji (ECE) surowego softmaksu, zastosowanie temperature scalingu
i wyznaczenie progu akceptacji z krzywej ryzyko–pokrycie. Porównywane są dwie miary pewności:
maksimum softmaksu oraz margines między dwoma najlepszymi klasami. Wynikiem jest **konkretna,
uzasadniona wartość progu** używana w wersji produkcyjnej — np. najniższy próg dający
dokładność warunkową ≥ 99% przy pokryciu powyżej 90%.

**E10 — Odporność systemu (rodzina trzech eksperymentów).**
Wszystkie oceniane na własnym zbiorze testowym, z grupowaniem wyników według metadanych
sesji. *E10a — oświetlenie*: dzienne, żarowe, LED zimne, przyciemnione; hipoteza mówi, że
spadnie przede wszystkim dokładność **koloru**, a nie figury, bo rozróżnienie
czerwony–czarny zależy od barwy światła (sprawdzane jest też, czy CLAHE i korekta balansu
bieli to niwelują). *E10b — kąt obserwacji*: wyznaczenie granicy stosowalności systemu.
*E10c — liczba kart i przesłonięcia*: układy 1/2/5/10 kart oraz karty nachodzące, z pomiarem
recallu detekcji i czasu przetwarzania klatki — wprost wpływa na budżet czasu rzeczywistego.

### 5.4 Dane

**E11 — Domain gap: dane publiczne kontra własne.**
Model uczony wyłącznie na danych publicznych oceniany jest równolegle na publicznym i własnym
zbiorze testowym; różnica między tymi wynikami **jest** miarą domain gap. Drugi wariant
sprawdza, czy dostrojenie na niewielkiej próbce własnych zdjęć (rozłącznej z testem na
poziomie sesji) tę różnicę likwiduje. Eksperyment zamienia główne ograniczenie przyjętej
strategii danych w wynik ilościowy.

**E12 — Dane syntetyczne kontra rzeczywiste.**
Trzy treningi detektora o **zrównanej liczbie obrazów**: na danych tylko rzeczywistych, tylko
syntetycznych i mieszanych — wszystkie oceniane na rzeczywistym zbiorze testowym. Hipoteza:
mieszanka poprawi recall na scenach wielokartowych, a wariant czysto syntetyczny wyraźnie
odstanie, bo model nauczy się artefaktów generatora zamiast cech kart. Wynikiem jest
praktyczna odpowiedź na pytanie, czy napisanie generatora opłaca się bardziej niż ręczne
fotografowanie układów.

### 5.5 Priorytety

| Priorytet | Eksperymenty | Uzasadnienie |
|---|---|---|
| Krytyczne | E1, E3, E4, E9 | podstawa rozdziału eksperymentalnego |
| Bardzo ważne | E2, E5, E11 | uzasadniają kluczowe decyzje projektowe |
| Ważne | E6, E7, E8 | uzasadniają architekturę i wybór technologii |
| Uzupełniające | E10a–c, E12 | rozdział o odporności i ograniczeniach; wymagają własnego zbioru |

Minimalny zestaw dający wartościową pracę: **E1–E5 oraz E9** — sześć eksperymentów
wykonalnych wyłącznie na danych publicznych.

Pełne karty eksperymentów (hipoteza, zmienne, wykonanie, sposób prezentacji):
[PROJECT_PLAN.md, sekcja 11](docs/PROJECT_PLAN.md#11-eksperymenty).

---

## 6. Stos technologiczny

| Obszar | Wybór |
|---|---|
| Język | Python 3.12 |
| Uczenie głębokie | PyTorch |
| Detekcja | Ultralytics YOLO (wariant `n`) — licencja AGPL-3.0 |
| Klasyczne CV | OpenCV |
| Augmentacja | Albumentations |
| Metryki | scikit-learn |
| Wykresy | Matplotlib |
| GUI | PySide6 (Qt, LGPL) |
| Testy | pytest |

Świadomie odrzucone jako nieuzasadniona złożoność w projekcie tej skali: PyTorch Lightning,
Weights & Biases, Hydra, DVC, Docker. Każdy wybór i każde odrzucenie jest uzasadnione
porównaniem z alternatywami w [PROJECT_PLAN.md, sekcja 4](docs/PROJECT_PLAN.md#4-technologie).

---

## 7. Struktura repozytorium

```
├── README.md            ten plik
├── configs/             wszystkie parametry systemu (YAML)
├── data/                dane — poza gitem, oprócz splits/ i metadata/
├── docs/                dokumentacja i tekst pracy inżynierskiej
├── experiments/         wyniki przebiegów: config + metryki + wykresy
├── models/              wagi modeli (poza gitem)
├── scripts/             punkty wejścia CLI
├── src/cardvision/      kod źródłowy (pakiet instalowalny)
└── tests/               testy jednostkowe i integracyjne
```

Pełny opis wraz z uzasadnieniem:
[PROJECT_PLAN.md, sekcja 13](docs/PROJECT_PLAN.md#13-struktura-repozytorium).

---

## 8. Plan realizacji

15 faz, około 200–250 h nakładu, zadania rozbite na porcje 1–4 h.
**Kamień milowy w Fazie 6** — po jej zakończeniu istnieje działający system end-to-end,
a wszystko dalsze (eksperymenty, kalibracja, GUI, ewaluacja) jest jego ulepszaniem, a nie
warunkiem, żeby cokolwiek w ogóle zadziałało.

| Fazy | Zawartość |
|---|---|
| 1–2 | środowisko, pozyskanie i **audyt danych**, podziały |
| 3–5 | geometria i korekcja perspektywy, klasyfikator, detektor i generator scen |
| 6 | **integracja pipeline'u end-to-end — kamień milowy** |
| 7–9 | własny zbiór testowy, kalibracja i próg pewności, eksperymenty E1–E9 |
| 10–12 | eksperymenty E10–E12, GUI, optymalizacja czasu rzeczywistego |
| 13–15 | ewaluacja końcowa, analiza błędów, redakcja pracy |

Harmonogram zadaniowy: [PROJECT_PLAN.md, sekcje 14–20](docs/PROJECT_PLAN.md).

---

## 9. Uruchomienie

> Sekcja docelowa — komendy zaczną działać po Fazie 6.

```bash
# instalacja
py -3.12 -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
pip install -e .

# weryfikacja środowiska
python -c "import torch; print(torch.__version__, torch.cuda.is_available())"

# rozpoznanie kart na zdjęciu
python scripts/predict.py --input obraz.jpg --config configs/pipeline/default.yaml

# aplikacja GUI
python scripts/run_app.py

# uruchomienie eksperymentu
python scripts/run_experiment.py --config configs/experiments/e01_architectures.yaml

# testy
pytest
```

---

## 10. Dokumentacja

| Plik | Zawartość |
|---|---|
| [PROJECT_PLAN.md](docs/PROJECT_PLAN.md) | pełny plan projektu: architektura, dane, eksperymenty, 15 faz z zadaniami |
| `docs/DATASET.md` | raport z audytu danych — powstaje w Fazie 2 |
| `docs/EXPERIMENTS.md` | rejestr przeprowadzonych eksperymentów i wniosków |
| `docs/JOURNAL.md` | dziennik prac — co zrobiono, co nie działa, następny krok |
| `docs/THESIS_OUTLINE.md` | mapowanie wyników projektu na rozdziały pracy |
| `docs/Zastosowanie narzędzi...docx` | tekst pracy inżynierskiej |

---

## 11. Zasady projektu

1. **Żadnych wymyślonych wyników.** Każda liczba w pracy ma pokrycie w zapisanym katalogu
   eksperymentu. Wartości nieznane są oznaczane jako `[DO ZMIERZENIA]`.
2. **Zbiór testowy używany raz**, na końcu — nigdy do strojenia parametrów.
3. **Podziały danych rozłączne na poziomie sesji nagraniowej**, nie pojedynczej klatki —
   inaczej wyniki są zawyżone przez przeciek informacji.
4. **Zero magicznych liczb w kodzie** — wszystkie parametry pochodzą z plików konfiguracyjnych.
5. **Wyniki negatywne są pełnoprawnymi wynikami** i trafiają do pracy tak samo jak pozytywne.

---

## Autor

Bartosz Bujanowicz — praca inżynierska, PJATK.
