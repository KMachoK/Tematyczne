==================================
Wydajność, skalowanie i replikacja
==================================

:Autorzy:
    1. Olaf Chomicki
    2. Konrad Machowski
    3. Wiktor Wydrzyński

Wydajność, skalowanie i replikacja
==================================

Współczesne systemy bazodanowe muszą sprostać wymaganiom związanym z wykładniczym przyrostem wolumenu danych oraz stale rosnącą liczbą jednoczesnych użytkowników. W tym kontekście kluczowe staje się zrozumienie trzech fundamentalnych pojęć: wydajności, skalowania i replikacji. Wydajność odnosi się do szybkości przetwarzania pojedynczych transakcji i zapytań przy optymalnym wykorzystaniu dostępnych zasobów. 

Gdy optymalizacja kodu przestaje wystarczać, konieczne staje się skalowanie – pionowe (zwiększanie mocy pojedynczego serwera) lub poziome (rozpraszanie obciążenia na wiele maszyn). Replikacja z kolei stanowi most łączący skalowanie odczytów z zapewnieniem wysokiej dostępności i odporności systemu na awarie sprzętowe.

Kontrola i buforowanie połączeń z bazą danych
=============================================

Nawiązywanie nowego połączenia z bazą danych jest operacją niezwykle kosztowną procesorowo i czasowo, ponieważ wymaga uwierzytelnienia użytkownika, alokacji pamięci oraz ustanowienia sesji sieciowej. W systemach o dużym natężeniu ruchu brak kontroli nad liczbą połączeń prowadzi do szybkiego wyczerpania zasobów serwera. 

Rozwiązaniem tego problemu jest buforowanie połączeń (Connection Pooling). Mechanizm ten polega na utrzymywaniu stałej puli otwartych, gotowych do użycia połączeń, które są wielokrotnie współdzielone przez aplikację. Narzędzia takie jak PgBouncer (dla PostgreSQL) czy wbudowane pule w serwerach aplikacji drastycznie zmniejszają narzut overhead, stabilizując czas reakcji bazy danych pod dużym obciążeniem.

INDEX i CLUSTER
===============

Indeksowanie to podstawowa technika optymalizacji zapytań o charakterze odczytowym. Instrukcja ``INDEX`` tworzy pomocniczą strukturę danych (najczęściej w formie drzewa B-drzewa lub tabeli mieszającej), która pozwala silnikowi bazy danych na błyskawiczne odnalezienie żądanych wierszy bez konieczności kosztownego przeszukiwania całej tabeli (Full Table Scan). 

Z kolei polecenie ``CLUSTER`` idzie o krok dalej – fizycznie reorganizuje strukturę danych na dysku w taki sposób, aby kolejność wierszy w tabeli dokładnie odpowiadała kolejności indeksu. Powoduje to, że powiązane rekordy są składowane obok siebie w tych samych blokach pamięci, co radykalnie przyspiesza zapytania zakresowe (Range Queries). Warto jednak pamiętać, że operacja ``CLUSTER`` jest kosztowna i nie jest automatycznie utrzymywana przy dodawaniu nowych danych.

Rola i zastosowanie replikacji
==============================

Replikacja polega na permanentnym kopiowaniu i synchronizowaniu danych z głównego węzła bazy danych (Primary/Master) na węzły pomocnicze (Replica/Slave). Jej rola w architekturze systemów IT jest wieloaspektowa. Po pierwsze, gwarantuje wysoką dostępność (High Availability) – w przypadku fizycznej awarii Mastera, jedna z replik może automatycznie przejąć jego funkcje (proces Failover). 

Po drugie, umożliwia skalowanie odczytów (Read Scalability). Przekierowanie operacji modyfikujących (INSERT, UPDATE) do węzła głównego, a operacji odczytu oraz generowania ciężkich raportów na repliki pozwala na znaczne odciążenie głównego serwera. Po trzecie, repliki stanowią doskonałą bazę do wykonywania kopii zapasowych bez wpływu na wydajność produkcyjnego środowiska.

Oprogramowanie i zaimplementowane mechanizmy replikacji
======================================================

Mechanizmy replikacji mogą być realizowane na poziomie silnika bazy danych lub za pomocą zewnętrznego oprogramowania. Wyróżnia się dwa główne podejścia pod kątem transmisji danych: replikację opartą na logu (WAL - Write-Ahead Logging w PostgreSQL lub Binlog w MySQL) oraz replikację logiczną. Ze względu na synchronizację, proces ten może przebiegać synchronicznie (klient otrzymuje potwierdzenie zapisu dopiero, gdy dane dotrą do repliki – gwarancja spójności, ale wyższe opóźnienia) lub asynchronicznie (zapis na replice odbywa się w tle).

Popularne oprogramowanie wspierające i automatyzujące replikację oraz zarządzanie klastrami to:

* **Patroni / Bucardo:** Narzędzia dedykowane dla ekosystemu PostgreSQL, służące do automatycznego zarządzania wysoką dostępnością i wykrywania awarii.
* **Orchestrator / Galera Cluster:** Zaawansowane rozwiązania dla bazy MySQL umożliwiające replikację wielomistrzowską (Multi-Master) lub topologie rozproszone.

Limity systemu oraz ograniczanie dostępu użytkowników
======================================================

Bezpieczeństwo i stabilność bazy danych wymagają rygorystycznego zarządzania limitami systemowymi oraz uprawnieniami. Zbyt duża swoboda przyznana użytkownikom lub procesom aplikacyjnym może doprowadzić do celowego bądź przypadkowego unieruchomienia systemu (np. poprzez uruchomienie zapytania alokującego całą pamięć RAM).

Systemy bazodanowe pozwalają na definiowanie ścisłych limitów (Resource Limits) takich jak: maksymalna liczba jednoczesnych połączeń dla danego użytkownika, limit czasu wykonywania pojedynczego zapytania (Statement Timeout) oraz maksymalna przestrzeń dyskowa. Ponadto, za pomocą mechanizmów kontroli dostępu (zarządzanie rolami, nadawanie uprawnień poprzez instrukcję ``GRANT`` oraz polityki zabezpieczeń na poziomie wierszy - RLS), ogranicza się widoczność danych wyłącznie do obszaru niezbędnego dla danej roli biznesowej.

Testy wydajności sprzętu na poziomie systemu operacyjnego
=========================================================

Przed wdrożeniem produkcyjnym bazy danych kluczowe jest przeprowadzenie niezależnych testów wydajnościowych podzespołów sprzętowych (Benchmarków) bezpośrednio na poziomie systemu operacyjnego, aby wykluczyć błędy konfiguracyjne środowiska:

* **Pamięć RAM:** Testowana za pomocą narzędzi takich jak ``sysbench`` lub ``memtester`` w celu weryfikacji przepustowości szyny pamięci oraz stabilności alokacji dużych bloków.
* **Procesor (CPU):** Wykorzystanie narzędzi do generowania obciążeń obliczeniowych (np. ``stress-ng`` lub ``sysbench cpu``) pozwala ocenić zachowanie systemu pod kątem przełączania kontekstu i zarządzania wątkami.
* **Dyski (I/O):** Najważniejszy test z punktu widzenia bazy danych. Narzędzie ``fio`` (Flexible I/O Tester) pozwala zasymulować specyficzny dla baz danych profil losowego odczytu i zapisu małych bloków (Random R/W), mierząc kluczowe parametry: liczbę operacji wejścia/wyjścia na sekundę (IOPS) oraz opóźnienia dyskowe (Latency).
