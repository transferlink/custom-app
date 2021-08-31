### Integracja własnej aplikacji z Transferlink
Jeśli oprogramowanie, którego używasz, nie jest obsługiwane przez Transferlink, możesz zaprogramować własną integrację. Do poprawnego działania integracji wymagane jest stworzenie trzech funkcji API po stronie Twojego oprogramowania - zwracanie listy zasobów (np. faktur lub zamówień), oznaczanie zasobu jako opłacony, oznaczanie zasobu jako nieopłacony.
#### Podstawowa konfiguracja i założenia
Zaloguj się na swoje konto Transferlink i przejdź do zakładki _Integracje / Aplikacje_, a następnie wybierz pozycję _Własna integracja_ z listy.

Transferlink będzie komunikował się z Twoim API tylko zapytaniami **GET**. Komunikacja może następować z różnych adresów IP, dlatego nie ograniczaj dostępu do API w tym zakresie. Jako parametr autoryzacjny możesz wykorzystać nagłówek **X-Secret-Token**, który dołączymy do każdego zapytania. Wartość tego parametru będzie taka sama jak zawartość pola _Hasło do przesłania w nagłówku_, które skonfigurujesz w ustawieniach aplikacji. Odpowiedź Twojego API powinna być zawsze generowana w formacie JSON.

Zapoznaj się z definicją poniższych funkcji, utwórz odpowiednie funkcje w swojej aplikacji, a następnie w ustawieniach aplikacji wprowadź pełne adresy URL pod którymi funkcje mogą zostać wywołane. 
#### Pobieranie listy zasobów
Do tego zapytania Transferlink dołączy 3 parametry w URL:
- **page** - numer strony z wynikami (integer)
- **limit** - ilość wyników na jedną stronę (min. 10, max. 50) (integer)
- **from** - data (unixtime) od której mają zostać pobrane zasoby (np. zamówienia - tylko nowsze niż wskazana data) (integer)

W odpowiedzi Twoje API powinno zwrócić strukturę JSON:
- **resources** - tablica z zasobami
  - **id** - unikalny identyfikator zasobu dla całej aplikacji z Twojego systemu (string, max. 255 znaków)
  - **resource_external_1** - identyfikator, który będzie wyszukiwany w tytule przelewu, np. numer faktury (string, max. 255 znaków)
  - **resource_external_2** - identyfikator, który będzie wyszukiwany w tytule przelewu, np. numer zamówienia (string, max. 255 znaków)
  - **resource_external_3** - identyfikator, który będzie wyszukiwany w tytule przelewu, np. numer Klienta (string, max. 255 znaków)
  - **amount** - kwota zasobu, np. zamówienia lub faktury, wartość brutto w najmniejszej jednostce danej waluty, np. groszach (integer)
  - **currency** - 3 znakowy kod waluty, np. PLN (string, max. 3 znaki)
  - **status** - wartość **PAID** (płatność zaksięgowana) lub **UNPAID** (płatność niezaksięgowana) (string enum)
- **pages** - ilość stron z wynikami (ceil(ilość wszystkich wyników / limit z querystring)) (integer)
```
Request: 

GET https://example.com/api/resources?page=1&limit=2&from=1630419109
X-Secret-Token: secret-password-from-settings
Content-Type: application/json
Accept: application/json

Response:

200 OK
{
  "resources": [
    {
      "id": "unique-resource-identifier-1",
      "resource_external_1": "keyword-for-detecting-payment-1",
      "resource_external_2": "keyword-for-detecting-payment-2",
      "resource_external_3": "keyword-for-detecting-payment-3",
      "amount": 1000,
      "currency": "PLN",
      "status": "PAID",
    },
    {
      "id": "unique-resource-identifier-2",
      "resource_external_1": "keyword-for-detecting-payment-1",
      "resource_external_2": "keyword-for-detecting-payment-2",
      "resource_external_3": "keyword-for-detecting-payment-3",
      "amount": "5500",
      "currency": "PLN",
      "status": "UNPAID",
    }
  ],
  "pages": 7
}
```

#### Oznaczanie zasobu jako opłacony
Do tego zapytania Transferlink dołączy 1 parametr w URL:
- **id** - unikalny identyfikator zasobu z Twojej aplikacji (string)

W odpowiedzi Twoje API powinno zwrócić kod 200 (HTTP STATUS OK). Treść odpowiedzi nie jest brana pod uwagę.
```
Request: 

GET https://example.com/api/mark_as_paid?id=unique-identifier-1
X-Secret-Token: secret-password-from-settings
Content-Type: application/json
Accept: application/json

Response:

200 OK
```

#### Oznaczanie zasobu jako nieopłacony
Do tego zapytania Transferlink dołączy 1 parametr w URL:
- **id** - unikalny identyfikator zasobu z Twojej aplikacji (string)

W odpowiedzi Twoje API powinno zwrócić kod 200 (HTTP STATUS OK). Treść odpowiedzi nie jest brana pod uwagę.
```
Request: 

GET https://example.com/api/mark_as_unpaid?id=unique-identifier-1
X-Secret-Token: secret-password-from-settings
Content-Type: application/json
Accept: application/json

Response:

200 OK
```
