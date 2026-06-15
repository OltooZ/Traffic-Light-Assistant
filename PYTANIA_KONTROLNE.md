# Pytania kontrolne - Traffic Light Assistant
### (wyjaśnienia "dla zielonego" - od zera, krok po kroku)

---

## CZĘŚĆ 1: O co w ogóle chodzi w tym projekcie?

### P1. Czym jest ten projekt, jednym zdaniem?
**O:** To program, który uczy "sztucznego kierowcę-zarządcę sygnalizacji świetlnej"
(agenta), jak najlepiej przełączać światła na skrzyżowaniu (dwie drogi, A i B),
żeby samochody jak najmniej stały w korkach. Agent uczy się sam, metodą
"prób i błędów", a na końcu porównujemy go z prostym, "głupim" sterowaniem
(światła zmieniają się cyklicznie, co X sekund, bez patrzenia na ruch).

### P2. Jak wygląda skrzyżowanie w tej symulacji?
**O:** To bardzo uproszczony model - dwie drogi (A i B), na każdej jest "kolejka"
samochodów (liczba czekających pojazdów). Tylko jedna droga może mieć zielone
światło w danym momencie. Pojazdy przyjeżdżają losowo, a gdy światło jest
zielone, kolejka się zmniejsza (samochody odjeżdżają).

### P3. Dlaczego to jest problem "inteligentny", a nie da się tego po prostu zaprogramować na sztywno?
**O:** Bo ruch jest **losowy** - nie wiemy z góry, kiedy przyjedzie kolejny
samochód. Sztywny harmonogram (np. "30 sekund zielone na A, 30 sekund na B")
nie reaguje na rzeczywistą sytuację - może świecić zielone na drodze, na której
nie ma nikogo, a w tym czasie druga droga się zapycha. Agent uczenia ze
wzmocnieniem **obserwuje** sytuację (jak długie są kolejki) i **decyduje**,
czy zmienić światło, czy nie - dopasowując się do warunków.

---

## CZĘŚĆ 2: Uczenie ze wzmocnieniem (Reinforcement Learning) - podstawy

### P4. Co to jest uczenie ze wzmocnieniem (RL - Reinforcement Learning)?
**O:** To rodzaj uczenia maszynowego, w którym **agent** (program) wykonuje
**akcje** w jakimś **środowisku** (tu: skrzyżowanie), a w zamian dostaje
**nagrodę** (liczbę mówiącą, jak dobra była ta akcja). Agent nie ma z góry
podanych "dobrych odpowiedzi" - musi sam, metodą prób i błędów (miliony razy),
odkryć, jakie akcje prowadzą do najwyższej sumy nagród w długim okresie.

Analogia: uczysz psa sztuczki. Nie mówisz mu "zrób krok 1, krok 2...". Dajesz
mu smakołyk (nagrodę), gdy zrobi coś dobrze. Po wielu powtórzeniach pies
("agent") sam wnioskuje, jakie zachowanie przynosi smakołyki.

### P5. Czym różni się to od "normalnego" uczenia maszynowego (np. klasyfikacji obrazków)?
**O:** W klasycznym uczeniu nadzorowanym mamy gotowe przykłady z odpowiedziami
("to jest kot", "to jest pies") i model uczy się je naśladować. W RL **nikt
nie podaje poprawnej odpowiedzi** - agent sam generuje swoje dane, próbując
różnych akcji, i uczy się na podstawie tego, czy poszło mu dobrze czy źle
(nagroda). To uczenie "przez doświadczenie", a nie "przez przykłady".

### P6. Co to jest "agent", "środowisko", "obserwacja", "akcja" i "nagroda" w TYM projekcie?
**O:**
- **Agent** - dwa "mózgi" decyzyjne: `light_A` (zarządza sygnalizacją na
  drodze A) i `light_B` (na drodze B). W praktyce współdzielą jedną sieć
  neuronową.
- **Środowisko** - symulacja skrzyżowania (`TrafficLightEnv`), która co krok
  generuje nowe samochody, zmienia długości kolejek i mówi agentom, co się
  dzieje.
- **Obserwacja** - to, co agent "widzi": długość własnej kolejki, długość
  kolejki sąsiada, aktualna faza świateł (kto ma zielone) i ile czasu trwa ta
  faza. Wszystko znormalizowane do zakresu [0,1] (czyli np. "50% maksymalnej
  kolejki" zamiast "25 samochodów").
- **Akcja** - decyzja binarna: `0` = "nie zmieniaj świateł", `1` = "chcę
  zmiany świateł".
- **Nagroda** - liczba ujemna, mówiąca jak źle jest (im dłuższe kolejki, tym
  gorsza nagroda, czyli bliższa minus nieskończoności). Agent uczy się
  **maksymalizować** sumę nagród, czyli **minimalizować** długość kolejek.

### P7. Dlaczego nagroda jest ujemna, a nie np. dodatnia za "dobre" zachowanie?
**O:** To po prostu wybór projektowy - "kara za czekanie". Każdy samochód
stojący w kolejce to "koszt" (-1 punkt za każdą jednostkę kolejki, ważoną).
Agent dostaje 0 tylko wtedy, gdy nie ma żadnych kolejek (sytuacja idealna -
ale praktycznie nieosiągalna, bo samochody zawsze jakieś przyjeżdżają).
Maksymalizacja nagrody = minimalizacja sumarycznego czasu czekania.

---

## CZĘŚĆ 3: Algorytm PPO - jak agent się uczy

### P8. Co to jest PPO (Proximal Policy Optimization) i dlaczego właśnie ten algorytm?
**O:** PPO to jeden z najpopularniejszych i najbardziej "bezpiecznych"
algorytmów RL. Działa tak:
1. Agent ma "politykę" (policy) - sieć neuronową, która na podstawie
   obserwacji zwraca prawdopodobieństwa akcji (np. 70% szans na "0", 30% na "1").
2. Agent gra wiele kroków w środowisku, zbierając dane: obserwacje, akcje,
   nagrody.
3. Na podstawie tych danych PPO **lekko** poprawia politykę tak, żeby częściej
   wybierała akcje, które dały dobre nagrody.
4. Słowo "Proximal" (bliski) oznacza, że PPO **nie pozwala** zmienić polityki
   za dużo na raz (jest tzw. "clipping" - przycinanie) - dzięki temu trening
   jest stabilny i agent nie "zapomina" tego, czego się nauczył.

Wybrano PPO, bo: jest stabilny, dobrze działa z akcjami dyskretnymi (0/1),
jest "sample-efficient" (uczy się sensownie bez gigantycznej liczby
przykładów) i dobrze radzi sobie ze współdzieloną polityką wielu agentów.

### P9. Czym jest "polityka" (policy) w prostych słowach?
**O:** To "mózg" agenta - funkcja (sieć neuronowa), która na wejściu dostaje
obserwację (np. "moja kolejka = 0.4, kolejka sąsiada = 0.8, jestem na zielonym
od 3 kroków") i na wyjściu zwraca decyzję: zmienić światło czy nie. Trening
polega na dopasowywaniu wag tej sieci, żeby decyzje były coraz lepsze.

### P10. Co to jest "funkcja wartości" (value function) i czemu jest ważna?
**O:** Oprócz polityki (co robić), PPO uczy też **drugą sieć**, która
przewiduje: "jak dobra jest ta sytuacja? Jaką sumę przyszłych nagród mogę się
stąd spodziewać?". To pomaga ocenić, czy konkretna akcja była lepsza czy gorsza
niż "przeciętnie" - i na tej podstawie korygować politykę. Jeśli ta funkcja
nie działa dobrze (nie umie ocenić sytuacji), trening polityki też idzie źle.
(To był jeden z bugów w projekcie - patrz Część 6).

### P11. Co to jest GAE (Generalized Advantage Estimation)?
**O:** To metoda obliczania "przewagi" (advantage) danej akcji - czyli "o ile
ta akcja była lepsza/gorsza niż przeciętna akcja w tej sytuacji". GAE łączy
informacje z wielu kroków w przyszłość, ważąc je tak, żeby zmniejszyć "szum"
(wariancję) w ocenie, ale nie wprowadzać za dużego błędu (bias). Im lepiej
oceniona "przewaga", tym precyzyjniej PPO poprawia politykę.

### P12. Co oznaczają najważniejsze hiperparametry treningu (z config.py)?
**O:**
- **total_timesteps = 300 000** - łączna liczba "kroków" symulacji, przez
  które przechodzi agent podczas treningu (im więcej, tym dłużej się uczy).
- **learning_rate = 3e-4** - jak duże kroki robi algorytm optymalizacji przy
  poprawianiu sieci neuronowej (zbyt duże = niestabilność, zbyt małe = wolne
  uczenie).
- **n_steps = 2048** - ile kroków symulacji agent zbiera przed każdą
  aktualizacją sieci.
- **batch_size = 64** - na ile małych "kawałków" dzielimy zebrane dane podczas
  jednej aktualizacji.
- **n_epochs = 10** - ile razy te same zebrane dane są używane do poprawiania
  sieci (powtórki).
- **gamma = 0.99** - "współczynnik dyskontowania" - jak bardzo agent ceni
  przyszłe nagrody względem natychmiastowych (0.99 = "przyszłość jest ważna,
  ale trochę mniej niż 'teraz'").
- **gae_lambda = 0.95** - parametr techniczny GAE, balans między
  dokładnością i stabilnością oceny.
- **clip_range = 0.2** - maksymalna dopuszczalna zmiana polityki w jednej
  aktualizacji (mechanizm "Proximal" z PPO).
- **ent_coef = 0.01** - "premia za eksplorację" - zachęca agenta, by czasem
  próbował różnych akcji, a nie zawsze tej samej (żeby nie "zaciął się" na
  jednej strategii za wcześnie).

---

## CZĘŚĆ 4: Wiele agentów i kooperacja

### P13. Co to znaczy, że to system "wieloagentowy" (multi-agent)?
**O:** Mamy dwóch agentów - `light_A` i `light_B` - każdy "odpowiada" za jedną
drogę. Każdy z nich w każdym kroku podejmuje własną decyzję (0 albo 1). To
różni się od typowego RL z jednym agentem.

### P14. Co to jest "parameter sharing" (współdzielenie parametrów) i dlaczego je zastosowano?
**O:** Zamiast trenować DWIE oddzielne sieci neuronowe (jedną dla `light_A`,
drugą dla `light_B`), używamy **jednej** sieci dla obu. Działa to, bo
obserwacje są zdefiniowane **symetrycznie** - każdy agent widzi "moja kolejka,
kolejka sąsiada, ..." (a nie "kolejka A, kolejka B" na sztywno). Dzięki temu
ta sama sieć działa dobrze niezależnie, którego agenta reprezentuje. Zaleta:
prostsze, szybsze uczenie - dane z obu agentów "trenują" tę samą sieć, czyli
mamy efektywnie 2x więcej danych.

### P15. Co to jest współczynnik kooperacji "alpha" i jak wpływa na zachowanie agentów?
**O:** Wzór nagrody to:
```
reward_A = -(queue_A + alpha * queue_B) / scale
```
`alpha` mówi, jak bardzo agent A "martwi się" o kolejkę na drodze B (czyli
o sąsiada):
- **alpha = 0** - agent jest całkowicie "egoistyczny" - liczy się tylko jego
  własna kolejka. Efekt: agent, który ma zielone, nigdy nie ma motywacji, by
  je oddać (jego kolejka jest niska, więc jest "zadowolony").
- **alpha = 1** - agent dba o sąsiada tak samo jak o siebie ("pełny
  altruizm").
- **alpha = 0.5** (wartość użyta w projekcie) - balans: agent czuje "ukłucie"
  złej nagrody, gdy druga droga ma korek, nawet jeśli jego własna droga jest
  pusta - więc oddaje zielone, gdy sąsiad tego potrzebuje.

To jest **kluczowy mechanizm** projektu - bez `alpha > 0` agenci uczą się
egoistycznego "zachłannego trzymania" zielonego światła.

### P16. Jak właściwie działa zmiana fazy świateł (kto, kiedy, na jak długo)?
**O:** Zasady:
1. Każdy agent w każdym kroku może zgłosić akcję `1` ("chcę zmiany").
2. Jeśli **obaj** zgłoszą `1` w tym samym kroku - żądania się "anulują"
   (nic się nie zmienia). To zapobiega chaotycznym, jednoczesnym przełączeniom.
3. Jest **minimalny czas zielonego** (`min_green_steps = 4`) - światło nie
   może się zmienić wcześniej, niż po 4 krokach (zapobiega "miganiu").
4. Jest faza **żółta** (`yellow_steps = 2`) - po decyzji o zmianie, najpierw
   2 kroki żółtego, potem zmiana na drugą drogę (jak w prawdziwym ruchu).

---

## CZĘŚĆ 5: Proces Poissona i statystyka (związek z przedmiotem)

### P17. Co to jest rozkład / proces Poissona i jak go użyto?
**O:** Rozkład Poissona opisuje liczbę "zdarzeń" (tu: przyjazdów samochodów)
w jednostce czasu, gdy zdarzenia są **rzadkie i niezależne od siebie** (np.
"średnio 0.6 samochodu na krok na drodze A"). W każdym kroku symulacji liczba
nowych samochodów na drodze A losowana jest z rozkładu Poissona o parametrze
`lambda_A = 0.6`, a na drodze B z `lambda_B = 0.4`. To realistyczny model
"losowego ruchu" - czasem przyjedzie 0 auto, czasem 1, rzadko 2-3.

### P18. Dlaczego proces Poissona "łączy się" z przedmiotem (Analiza regresji i szeregów czasowych)?
**O:** Bo:
- przyjazdy pojazdów to **proces stochastyczny w czasie** (klasyczny temat
  szeregów czasowych),
- krzywa nagrody podczas treningu to **szereg czasowy** (wartości w czasie,
  niestacjonarny - zmienia się trend),
- porównanie PPO vs baseline opiera się na **statystyce opisowej** (średnia
  ± odchylenie standardowe z wielu epizodów) - czyli wnioskowaniu
  statystycznym o tym, czy różnica jest istotna.

### P19. Jak interpretować wynik "PPO = -30.69 ± 1.71" vs "Baseline = -66.43 ± 6.52"?
**O:**
- Liczba przed `±` to **średnia nagroda** z 20 testowych epizodów (im bliżej
  0, tym lepiej - mniejsze kolejki).
- Liczba po `±` to **odchylenie standardowe** - jak bardzo wyniki różnią się
  między epizodami (mniejsze = bardziej **stabilne, przewidywalne**
  zachowanie).
- PPO ma wynik bliższy 0 (lepszy) ORAZ mniejsze odchylenie (bardziej
  konsekwentny) niż baseline. Czyli agent RL nie tylko **średnio** lepiej
  zarządza ruchem, ale robi to też **bardziej powtarzalnie**.

---

## CZĘŚĆ 6: Najważniejsze problemy napotkane (i czego nas uczą)

### P20. Co poszło źle z metodą `seed()` i czego to nas uczy o "wrapperach"?
**O:** Stable-Baselines3 (biblioteka do PPO) przy starcie wywołuje metodę
`env.seed()`, żeby ustawić generator liczb losowych. Środowisko z PettingZoo
jest "owijane" (wrapped) przez SuperSuit, żeby było kompatybilne z SB3 - ale
ten wrapper nie miał metody `seed()`. Trzeba było ją "dorobić" (monkey-patch)
na wewnętrznym obiekcie `env.venv`. **Nauka:** łączenie kilku bibliotek
("klejenie" frameworków) często wymaga takich małych "łatek" na styku API.

### P21. Co to był problem "greedy hold" (zachłanne trzymanie światła) i jak go naprawiono?
**O:** Pierwsza wersja: tylko agent z zielonym mógł zgłosić zmianę. Ten agent
miał niską własną kolejkę (bo jeździ), więc nigdy nie "chciał" oddać zielonego
- a druga droga rosła do maksimum (50 pojazdów). Naprawa: (1) pozwolono
**obu** agentom zgłaszać zmianę, (2) zwiększono `alpha` z 0.3 do 0.5, żeby
agent z zielonym **czuł** karę za korek na drugiej drodze. **Nauka:** projekt
nagrody i reguł gry bezpośrednio determinuje, JAKĄ strategię agent wynajdzie -
nawet "działający" kod może uczyć złej strategii.

### P22. Co to był problem "skala nagrody za duża" i dlaczego normalizacja pomogła?
**O:** Surowa nagroda (suma kolejek razy -1) po 500 krokach epizodu dawała
sumy rzędu **-14 000**. Funkcja wartości PPO musiała "zgadywać" takie ogromne
liczby - to bardzo trudne zadanie regresji (mała zmiana wagi sieci = ogromna
zmiana przewidywania). Po znormalizowaniu nagrody do zakresu [-1, 0] (dzieląc
przez maksymalną możliwą wartość), funkcja wartości miała dużo łatwiejsze,
"mniejsze" liczby do przewidywania i zaczęła się sensownie uczyć (metryka
`explained_variance` zaczęła rosnąć od ~0). **Nauka:** w uczeniu maszynowym
**skala** danych (i nagród) ma ogromne znaczenie dla stabilności treningu.

### P23. Co to był problem "ta sama akcja dla obu agentów" w ewaluacji?
**O:** Podczas testowania (nie treningu) kod błędnie wołał `model.predict()`
**raz** i tę samą akcję stosował do obu agentów. Skoro obaj dostawali np.
akcję `1`, aktywowała się reguła "anulowania" (oba chcą zmiany na raz = nic
się nie dzieje) - agent wyglądał, jakby był "zablokowany". Naprawa: wywołać
`predict()` **niezależnie** dla obserwacji każdego agenta. **Nauka:** w
systemach wieloagentowych trzeba bardzo uważać, żeby każdy agent dostawał
**swoją** obserwację i **swoją** decyzję.

### P24. Co to był problem z baseline (cyklicznym sterowaniem) i "cancel-out"?
**O:** Baseline (sztywny harmonogram) miał dwie instancje tej samej polityki
cyklicznej, które w tym samym kroku **obie** zgłaszały żądanie zmiany - i
znowu uruchamiała się reguła anulowania, więc światła nigdy się nie
zmieniały. Naprawa: tylko agent, który **aktualnie ma zielone**, zgłasza
żądanie zmiany (agent na czerwonym nic nie robi). **Nauka:** te same reguły
gry (np. "anulowanie przy konflikcie") trzeba przemyśleć dla **każdej**
polityki, która z nimi wchodzi w interakcję - nie tylko dla agenta RL.

---

## CZĘŚĆ 7: Technologie - co, gdzie i dlaczego

### P25. Co to jest PettingZoo i do czego służy w projekcie?
**O:** To biblioteka-standard dla środowisk **wieloagentowych** RL (taki
"Gymnasium, ale dla wielu agentów"). Środowisko `TrafficLightEnv` jest
napisane jako `ParallelEnv` z PettingZoo - co oznacza, że w każdym kroku
**wszyscy agenci** podejmują decyzje **jednocześnie** (równolegle), a nie po
kolei. To odpowiada rzeczywistości - obie sygnalizacje "myślą" w tym samym
momencie.

### P26. Co to jest SuperSuit i Stable-Baselines3 (SB3), i dlaczego są potrzebne razem?
**O:** Stable-Baselines3 to biblioteka z gotowymi, dobrze przetestowanymi
implementacjami algorytmów RL (m.in. PPO) - ale jest napisana pod
**jednoagentowe** środowiska Gymnasium (klasa `VecEnv`). SuperSuit to
"konwerter/most" - bierze środowisko PettingZoo (wieloagentowe) i
przekształca je tak, żeby wyglądało dla SB3 jak zwykłe `VecEnv` (każdy agent
= jedno "równoległe środowisko"). Dzięki temu można użyć gotowego, sprawdzonego
PPO z SB3 do treningu wieloagentowego, bez pisania własnego algorytmu od
zera.

### P27. Dlaczego użyto Redis?
**O:** Redis to szybka baza danych "w pamięci" (key-value store). W projekcie
służy do **przekazywania metryk treningu na żywo** - podczas gdy PPO trenuje
(co może trwać minuty), `RedisMetricsCallback` zapisuje do Redis aktualne
wartości (np. bieżącą nagrodę, krok treningu). Dzięki temu inny proces (np.
API albo dashboard) może **w czasie rzeczywistym** odczytać postęp treningu,
bez czekania na jego zakończenie.

### P28. Dlaczego użyto FastAPI?
**O:** FastAPI to framework do budowania API (REST) w Pythonie. Tutaj
udostępnia "z zewnątrz" funkcje projektu jako endpointy HTTP - np. można
wysłać żądanie, żeby uruchomić trening, sprawdzić jego status (czytając
Redis), albo poprosić o ewaluację modelu. Dzięki temu projekt nie jest "tylko
skryptem" - może być używany jako usługa (np. przez stronę WWW albo inny
program).

### P29. Dlaczego użyto Optuna?
**O:** Optuna to biblioteka do **automatycznego dobierania hiperparametrów**
(np. learning_rate, batch_size, alpha). Ręczne sprawdzanie wszystkich
kombinacji byłoby bardzo czasochłonne. Optuna używa algorytmu TPE (Tree-
structured Parzen Estimator) - "inteligentnie" zgaduje, jakie kombinacje
parametrów warto przetestować jako następne, na podstawie wyników
poprzednich prób, żeby szybciej znaleźć dobry zestaw.

### P30. Dlaczego do zarządzania środowiskiem Python finalnie użyto Python 3.12 (a nie 3.13 zgodnie z pyproject.toml)?
**O:** Jedna z zależności projektu (`tinyscaler`, używana przez SuperSuit) nie
ma jeszcze gotowej, "skompilowanej" wersji (wheel) dla Python 3.13 -
zainstalowanie jej wymagałoby kompilatora C++ (Microsoft Visual C++ Build
Tools), którego nie było na komputerze. Dla Python 3.12 taka gotowa wersja
istnieje. Dlatego stworzono wirtualne środowisko (`venv`) z Pythonem 3.12 i
zainstalowano wszystkie zależności tam - bez zmiany deklaracji w
`pyproject.toml` (która pozostaje historycznym wymaganiem projektu).
**Nauka:** kompatybilność wersji Pythona z bibliotekami "natywnymi"
(skompilowanymi z C/C++) bywa ograniczona - nowsza wersja Pythona nie
zawsze ma od razu wsparcie wszystkich pakietów.

---

## CZĘŚĆ 8: Pytania "podsumowujące" (dobre na koniec egzaminu/obrony)

### P31. Gdyby ktoś zapytał "po co w ogóle 2 agentów, a nie 1 agent zarządzający całym skrzyżowaniem?" - co odpowiedzieć?
**O:** Modelowanie jako 2 agentów lepiej odzwierciedla rzeczywistość (każda
sygnalizacja "decyduje" niezależnie, ale w ramach wspólnych reguł) i jest
**skalowalne** - to samo podejście (więcej agentów, ta sama współdzielona
sieć) można rozszerzyć na skrzyżowania z wieloma drogami albo na sieć wielu
skrzyżowań, bez przebudowy architektury.

### P32. Jakie są główne wnioski z projektu?
**O:**
1. Agent RL (PPO) **radzi sobie lepiej** niż sztywny harmonogram - niższa
   średnia "kara" za korki i mniejsza zmienność wyników.
2. **Kooperacja (alpha)** jest kluczowa - bez niej agent uczy się egoistycznej,
   złej strategii.
3. **Projekt nagrody i jej skala** (normalizacja) są równie ważne jak wybór
   samego algorytmu.
4. Cały system (symulacja → trening → ewaluacja → API → wizualizacja) działa
   end-to-end i jest przetestowany (26 testów automatycznych).

### P33. Co można rozwinąć w przyszłości?
**O:** Bardziej złożone skrzyżowania (więcej dróg/faz), prawdziwe dane o
ruchu (zamiast symulowanego Poissona), koordynacja wielu skrzyżowań naraz,
uwzględnienie pieszych, oraz automatyczny dobór `alpha` przez Optunę.

### P34. "Czy to jest realistyczny model?" - jak odpowiedzieć krytycznie?
**O:** Nie w pełni - to **uproszczenie**: jedno skrzyżowanie, dwie drogi,
brak pieszych, uproszczony model ruchu (Poisson). Jednak **kluczowe elementy
realizmu** są zachowane: losowość przyjazdów (proces stochastyczny),
ograniczenia czasowe sygnalizacji (minimalny czas zielonego, faza żółta) oraz
wzajemne wykluczanie się świateł. To dobry "poligon" do testowania idei RL,
zanim zastosuje się je w bardziej złożonym, realistycznym symulatorze.

---

## Jak się tego uczyć?
1. Przeczytaj raz całość, żeby złapać "duży obraz" (Część 1-2).
2. Skup się na Części 3-4 (PPO, kooperacja `alpha`) - to "serce" projektu i
   najczęściej padające pytania.
3. Część 6 (bugi) jest świetna na pytania typu "jakie trudności napotkaliście"
   - pokazuje realną pracę inżynierską.
4. Część 7 (technologie) - umiej powiedzieć JEDNYM zdaniem, do czego służy
   każde narzędzie (PettingZoo, SuperSuit, SB3, Redis, FastAPI, Optuna).
5. Część 8 - przygotuj się na pytania "a co dalej / a czy to realistyczne".
