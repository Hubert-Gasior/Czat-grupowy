# Chat Grupowy

# Autorzy: Hubert Gąsior, Konrad J, Eryk W

## Spis treści
1. [Wstęp](#wstęp)
2. [Funkcjonalność](#funkcjonalność)
    1. [Logowanie do serwera](#logowanie-do-serwera)
    2. [Komunikacja między użytkownikami](#komunikacja-między-użytkownikami)
    3. [Przechowywanie oraz wyświetlanie historii chatu](#przechowywanie-oraz-wyświetlanie-historii-chatu)
    4. [Proces demona](#proces-demona)

## Wstęp
Niniejszy dokument opisuje funkcjonalność usługi grupowego komunikatora tekstowego, który umożliwia przechowywanie kont użytkowników oraz historii rozmów, działając w formie demona.

## Funkcjonalność

### Logowanie do serwera
Serwer przechowuje hasło, które umożliwiają użytkownikowi dołączenie do wybranej grupy. Proces dołączenia klienta odbywa się za pośrednictwem protokołu TCP. W pierwszym kroku klient podaje swój login, a następnie wprowadza i przesyła hasło do serwera. Serwer, korzystając z funkcji `select()`, odbiera hasło przesłane przez gniazdo TCP. Jeśli wprowadzone hasło jest poprawne, klient zostaje zapytany o numer grupy do której chce dołączyć po wybraniu odpowiedniego pokoju serwer przesyła odpowiedni adres grupy multicastowej, do której użytkownik zostaje dołączony.

### Komunikacja między użytkownikami
Serwer umożliwia wymianę wiadomości między użytkownikami w danym pokoju za pośrednictwem protokołu UDP oraz adresowania multicastowego. Wiadomości od użytkowników są odbierane przez serwer za pomocą połączenia unicast, a następnie rozsyłane do pozostałych użytkowników w pokoju za pomocą mechanizmu multicast.

### Przechowywanie oraz wyświetlanie historii chatu
Na serwerze zaimplementowano dwie funkcje umożliwiające zapisywanie i odczytywanie pliku tekstowego. Podczas wymiany wiadomości między użytkownikami serwer tworzy nowy plik `historia.txt` lub zapisuje do już istniejącego każdą otrzymaną wiadomość. W momencie dołączenia użytkownika do danej grupy serwer odczytuje odpowiedni plik i przesyła jego zawartość do zalogowanego użytkownika.

### Proces demona
Serwer posiada zaimplementowaną funkcję umożliwiającą uruchomienie go jako demona, co pozwala na odłączenie serwera od terminala i jego pracę w tle podczas działania systemu.
