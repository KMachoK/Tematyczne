======================
Partycjonowanie danych
======================

+---------+--------------------------+
| Autorzy:| 1. Olaf Chomicki         |
|         | 2. Konrad Machowski      |
|         | 3. Wiktor Wydrzyński     |
+---------+--------------------------+

Wstęp
=====

Partycjonowanie danych to technika architektoniczna polegająca na dzieleniu dużych tabel lub indeksów bazy danych na mniejsze, łatwiejsze w zarządzaniu fragmenty, zwane partycjami. Z punktu widzenia aplikacji, dane zazwyczaj nadal wyglądają jak jedna logiczna całość, ale fizycznie są oddzielone [1]_. Głównym celem stosowania tej techniki jest poprawa wydajności, skalowalności oraz ułatwienie zarządzania ogromnymi zbiorami danych w nowoczesnych systemach informatycznych.

W systemach bazodanowych szczególnie ważne jest to, że wraz ze wzrostem wolumenu danych operacje takie jak odczyt, zapis czy tworzenie kopii zapasowych stają się coraz wolniejsze. Dzięki partycjonowaniu, zapytania mogą skanować tylko te fragmenty tabeli, które są rzeczywiście potrzebne, co znacząco optymalizuje procesy dyskowe i zużycie pamięci operacyjnej oraz zasobów procesora.


Korzyści i podstawowe podziały
==============================

Jednym z podstawowych celów wdrażania partycjonowania jest mechanizm *partition pruning* (odcinanie partycji). Pozwala on silnikowi bazy danych ignorować podczas wykonywania zapytania te partycje, które na pewno nie zawierają wyników, o które prosi użytkownik lub aplikacja. Ponadto ułatwione jest zarządzanie cyklem życia informacji – archiwalne dane można szybko odłączyć lub usunąć operując wyłącznie na metadanych, zamiast obciążać system długotrwałymi operacjami kasowania pojedynczych wierszy.

Wyróżnia się dwa główne typy partycjonowania: poziome oraz pionowe. Partycjonowanie poziome polega na podziale wierszy – różne grupy rekordów trafiają do różnych partycji na podstawie zdefiniowanego klucza. Z kolei partycjonowanie pionowe dzieli tabelę na kolumny, separując rzadko używane lub bardzo duże dane (np. pliki binarne, długie teksty) od tych odpytywanych najczęściej, co zmniejsza rozmiar bloku danych odczytywanego z dysku [2]_.


Strategie podziału horyzontalnego
=================================

W praktyce inżynierskiej najczęściej stosuje się partycjonowanie poziome, które realizuje się za pomocą kilku podstawowych strategii. Partycjonowanie zakresowe (Range Partitioning) dzieli dane na podstawie ciągłych przedziałów wartości. Jest to najczęstszy wybór w przypadku danych powiązanych z czasem, gdzie każda fizyczna partycja reprezentuje określony miesiąc lub rok. 

Kolejną strategią jest partycjonowanie listowe (List Partitioning), w którym wiersze są przypisywane do partycji na podstawie przynależności do zdefiniowanej listy wartości (np. kodów regionów lub statusów dokumentu). Często stosuje się również partycjonowanie skrótu (Hash Partitioning), które wykorzystuje funkcję matematyczną do równomiernego rozłożenia danych, co świetnie sprawdza się w sytuacjach braku naturalnego klucza biznesowego [3]_. Czasem stosuje się podejścia złożone, łączące np. podział po dacie z hashowaniem.


Partycjonowanie a architektura rozproszona
==========================================

Kluczowym aspektem przy projektowaniu większych systemów jest odróżnienie tradycyjnego partycjonowania od shardingu. Partycjonowanie w klasycznym ujęciu ma miejsce wewnątrz pojedynczej instancji bazy danych (np. na jednym serwerze w PostgreSQL czy Oracle). Wszystkie partycje, choć oddzielone fizycznie na dysku, współdzielą moc obliczeniową i pamięć tego samego serwera.

Sharding to natomiast specyficzna forma partycjonowania horyzontalnego, ale w architekturze rozproszonej (Shared-Nothing). Każdy fragment danych ("shard") staje się autonomiczną bazą danych rezydującą na oddzielnym węźle. Takie podejście wymuszone jest w momencie, w którym ograniczenia sprzętowe pojedynczej maszyny ulegają wyczerpaniu, zmuszając do równoległego podziału obciążenia na klaster serwerów [4]_.


Wyzwania, problemy i ograniczenia
=================================

Mimo znacznych korzyści wydajnościowych, partycjonowanie rodzi specyficzne wyzwania projektowe. Niewłaściwy dobór klucza podziału może doprowadzić do powstania tzw. gorących punktów (hotspots), gdzie większość ruchu i tak trafia tylko do jednej partycji, niwelując korzyści ze skali. Ponadto, zapytania wymuszające złączenia (Cross-partition joins) między danymi rozrzuconymi po różnych partycjach bywają mocno obciążające dla optymalizatora bazy danych [5]_.

Administratorzy i programiści muszą również przemyśleć strategię indeksowania. Indeksy lokalne (tworzone oddzielnie dla każdej partycji) są bardzo łatwe w utrzymaniu, ale mogą spowalniać globalne wyszukiwanie. Z kolei indeksy globalne (pokrywające całą tabelę niezależnie od podziału) znacząco przyspieszają odczyt, lecz ich utrzymanie i przebudowa bywają bardzo kosztowne podczas modyfikowania struktury partycji.


Podsumowanie
============

Partycjonowanie danych to standardowe narzędzie pozwalające radzić sobie z ogromnymi przyrostami danych. Pozwala na utrzymanie stabilnej wydajności zapytań i upraszcza procesy administracyjne. Decyzja o jego zastosowaniu musi jednak opierać się na dokładnej analizie wzorców dostępu do danych (query patterns), aby zagwarantować, że narzut związany z zarządzaniem wieloma partycjami nie przerośnie korzyści wynikających z przyspieszonego działania systemu.

.. [1] "Architektura systemów bazodanowych" - Podstawy i fizyczny podział danych.
.. [2] "Optymalizacja I/O w relacyjnych bazach danych".
.. [3] Metody zapewniania równomiernego rozkładu danych przy użyciu algorytmów hashujących.
.. [4] Różnice między partycjonowaniem wertykalnym, horyzontalnym a architekturą Shared-Nothing.
.. [5] Problem optymalizacji złożonych zapytań SQL w rozproszonych modelach danych.
