---
description: >-
  https://codelabs.developers.google.com/codelabs/cloud-documentai-manage-processors-python#0
---

# Zarządzanie procesorami Document AI za pomocą Pythona



### Informacje o tym ćwiczeniu (w Codelabs)

_temat_Ostatnia aktualizacja: cze 20, 2023_koło\_konta_Autorzy: Laurent Picard



### [1. Przegląd](https://codelabs.developers.google.com/codelabs/cloud-documentai-manage-processors-python#0)

![c6d2ea69b1ba0eff.png](https://codelabs.developers.google.com/static/codelabs/cloud-documentai-manage-processors-python/img/c6d2ea69b1ba0eff.png)

#### **Co to jest dokumentowa sztuczna inteligencja?** <a href="#what-is-document-ai" id="what-is-document-ai"></a>

[Document AI](https://cloud.google.com/document-ai/docs/overview) to platforma, która pozwala wyciągać wnioski z dokumentów. W istocie oferuje rosnącą listę procesorów dokumentów (zwanych także parserami lub rozdzielaczami, w zależności od ich funkcjonalności).

Istnieją dwa sposoby zarządzania procesorami Document AI:

* ręcznie, z konsoli internetowej;
* programowo, przy użyciu interfejsu API Document AI.

Oto przykładowy zrzut ekranu przedstawiający listę procesorów, zarówno z konsoli internetowej, jak i z kodu Pythona:

![312f0e9b3a8b8291.png](https://codelabs.developers.google.com/static/codelabs/cloud-documentai-manage-processors-python/img/312f0e9b3a8b8291.png)

W tym laboratorium skoncentrujesz się na programowym zarządzaniu procesorami Document AI za pomocą biblioteki klienta Python.

#### **Co zobaczysz** <a href="#what-youll-see" id="what-youll-see"></a>

* Jak skonfigurować środowisko
* Jak pobrać typy procesorów
* Jak tworzyć procesory
* Jak wyświetlić listę procesorów projektu
* Jak korzystać z procesorów
* Jak włączyć/wyłączyć procesory
* Jak zarządzać wersjami procesorów
* Jak usunąć procesory

#### **Czego potrzebujesz** <a href="#what-youll-need" id="what-youll-need"></a>

* Projekt Google Cloud
* Przeglądarka, taka jak [Chrome](https://www.google.com/chrome) lub [Firefox](https://www.mozilla.org/firefox/)
* Znajomość obsługi Pythona

### [3. Konfiguracja środowiska](https://codelabs.developers.google.com/codelabs/cloud-documentai-manage-processors-python#2)

Zanim zaczniesz korzystać z Document AI, uruchom następujące polecenie w Cloud Shell, aby włączyć interfejs API Document AI:

```
gcloud services enable documentai.googleapis.com
```

Powinieneś zobaczyć coś takiego:

```
Operation "operations/..." finished successfully.
```

Teraz możesz korzystać z Document AI!

Przejdź do swojego katalogu domowego:

```
cd ~
```

Utwórz środowisko wirtualne Pythona, aby wyizolować zależności:

```
virtualenv venv-docai
```

Aktywuj środowisko wirtualne:

```
source venv-docai/bin/activate
```

Aby zakończyć korzystanie ze środowiska wirtualnego i wrócić do systemowej wersji Pythona, możesz użyć polecenia `deactivate`.

Zainstaluj IPython, bibliotekę klienta Document AI i python-tabulate (którego użyjesz do ładnego wydrukowania wyników żądania):

```
pip install ipython google-cloud-documentai tabulate
```

Powinieneś zobaczyć coś takiego:

```
...
Installing collected packages: ..., tabulate, ipython, google-cloud-documentai
Successfully installed ... google-cloud-documentai-2.15.0 ...
```

Teraz możesz już korzystać z biblioteki klienta Document AI!

Jeśli konfigurujesz własne środowisko programistyczne w języku Python poza Cloud Shell, postępuj zgodnie z tymi [wytycznymi](https://cloud.google.com/python/setup).

Ustaw następujące zmienne środowiskowe:

```
export PROJECT_ID=$(gcloud config get-value core/project)
```

```
# Choose "us" or "eu"
export API_LOCATION="us"
```

* Obie zmienne środowiskowe zostaną użyte w Twojej aplikacji.
* `API_LOCATION` pozwala wybrać, gdzie będą realizowane wnioski (`"us"` w przypadku Stanów Zjednoczonych, `"eu"` w przypadku Unii Europejskiej). Możesz używać procesorów w wielu lokalizacjach, ale w całym laboratorium będziesz używać jednej lokalizacji, aby było to proste.

Od tego momentu wszystkie kroki powinny być wykonywane w tej samej sesji.

**Jeśli rozpoczniesz nową sesję**, po przywróceniu środowiska za pomocą następujących konkretnych części z tej sekcji konieczne będzie ponowne uruchomienie od tego miejsca:

* Aktywuj swoje środowisko wirtualne
* Ustaw i sprawdź zmienne środowiskowe

Upewnij się, że zmienne środowiskowe są poprawnie zdefiniowane:

```
echo $PROJECT_ID
```

```
echo $API_LOCATION
```

W kolejnych krokach będziesz używać interaktywnego interpretera języka Python o nazwie [IPython](https://ipython.org/), który właśnie zainstalowałeś. Rozpocznij sesję, uruchamiając `ipython` w Cloud Shell:

```
ipython
```

Powinieneś zobaczyć coś takiego:

```
Python 3.9.2 (default, Feb 28 2021, 17:03:44)
Type 'copyright', 'credits' or 'license' for more information
IPython 8.14.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]:
```

W razie potrzeby możesz zakończyć sesję IPython za pomocą polecenia `exit`.

Skopiuj następujący kod do sesji IPython:

```
import os
from typing import Iterator, MutableSequence, Optional, Sequence, Tuple

import google.cloud.documentai_v1 as docai
from tabulate import tabulate

PROJECT_ID = os.getenv("PROJECT_ID", "")
API_LOCATION = os.getenv("API_LOCATION", "")

assert PROJECT_ID, "PROJECT_ID is undefined"
assert API_LOCATION in ("us", "eu"), "API_LOCATION is incorrect"

# Test processors
document_ocr_display_name = "document-ocr"
form_parser_display_name = "form-parser"

test_processor_display_names_and_types = (
    (document_ocr_display_name, "OCR_PROCESSOR"),
    (form_parser_display_name, "FORM_PARSER_PROCESSOR"),
)

def get_client() -> docai.DocumentProcessorServiceClient:
    client_options = {"api_endpoint": f"{API_LOCATION}-documentai.googleapis.com"}
    return docai.DocumentProcessorServiceClient(client_options=client_options)

def get_parent(client: docai.DocumentProcessorServiceClient) -> str:
    return client.common_location_path(PROJECT_ID, API_LOCATION)

def get_client_and_parent() -> Tuple[docai.DocumentProcessorServiceClient, str]:
    client = get_client()
    parent = get_parent(client)
    return client, parent
    
```

* To wykorzystuje zdefiniowane wcześniej zmienne środowiskowe `PROJECT_ID` i `API_LOCATION`.
* Funkcje narzędziowe `get_*()` będą używane w kolejnych krokach w przypadku żądań procesora.

Możesz już wysłać pierwsze żądanie i pobrać typy procesorów



BARD AI TLUMACZENIE KODU @@@@@



Ten fragment kodu definiuje funkcje do zarządzania procesorami dokumentów za pomocą interfejsu API Google Cloud Document AI (DocumentAI). Obejmuje funkcje do pobierania klienta, uzyskiwania ścieżki zasobu nadrzędnego i uzyskiwania klienta i nadrzędnego razem.

**Importowanie bibliotek**

Kod zaczyna się od zaimportowania niezbędnych bibliotek: `os`, `typing`, `google.cloud.documentai_v1 as docai` i `tabulate`.

`os` jest używany do uzyskiwania dostępu do zmiennych środowiskowych, takich jak `PROJECT_ID` i `API_LOCATION`.

`typing` zapewnia adnotacje typu dla funkcji i zmiennych, co ułatwia czytelność i utrzymanie kodu.

`docai` to biblioteka Google Cloud Document AI, która umożliwia kodowi interakcję z interfejsem API DocumentAI.

`tabulate` to biblioteka do generowania tabel za pomocą danych opartych na tekście, która jest używana do wyświetlania listy procesorów.

**Zmienne środowiskowe**

Kod pobiera zmienne środowiskowe `PROJECT_ID` i `API_LOCATION`. Te zmienne są używane do identyfikacji projektu i lokalizacji punktu końcowego interfejsu API DocumentAI.

**Sprawdzanie spełnienia warunków**

Kod sprawdza, czy `PROJECT_ID` jest zdefiniowany i czy `API_LOCATION` jest równe `us` lub `eu`. Te kontrole zapewniają, że kod ma niezbędne informacje do połączenia się z interfejsem API DocumentAI.

**Testowe procesory**

Kod definiuje dwa testowe procesory: `document_ocr_display_name` do ekstrakcji tekstu i `form_parser_display_name` do analizy formularzy.

**Nazwy wyświetlania i typy procesorów**

Kod definiuje krotkę o nazwie `test_processor_display_names_and_types`, która zawiera pary nazw wyświetlania i typów procesorów. Ta krotka będzie używana do tworzenia i zarządzania procesorami.

**Pobieranie klienta**

Funkcja `get_client()` tworzy klienta DocumentAI za pomocą podanej `API_LOCATION`. Słownik `client_options` ustawia punkt końcowy API na określony `API_LOCATION`.

**Pobieranie nadrzędnego**

Funkcja `get_parent()` zwraca ścieżkę zasobu nadrzędnego do tworzenia procesorów, która składa się z identyfikatora projektu i lokalizacji API.

**Pobieranie klienta i nadrzędnego**

Funkcja `get_client_and_parent()` łączy `get_client()` i `get_parent()`, zwracając zarówno klienta, jak i ścieżkę zasobu nadrzędnego.

Ten kod zapewnia podstawowe bloki budulcowe do zarządzania procesorami dokumentów w interfejsie API Document AI.

**Tłumaczenie poszczególnych funkcji**

Python

```
def get_client() -> docai.DocumentProcessorServiceClient:
    client_options = {"api_endpoint": f"{API_LOCATION}-documentai.googleapis.com"}
    return docai.DocumentProcessorServiceClient(client_options=client_options)
```

Zachowaj ostrożność podczas korzystania z kodu. [Więcej informacji](https://bard.google.com/faq#coding)content\_copy

Funkcja `get_client()` tworzy klienta DocumentAI. Argument `API_LOCATION` określa lokalizację punktu końcowego API. Argument `client_options` określa opcje klienta, takie jak punkt końcowy API.

Python

```
def get_parent(client: docai.DocumentProcessorServiceClient) -> str:
    return client.common_location_path(PROJECT_ID, API_LOCATION)
```

Zachowaj ostrożność podczas korzystania z kodu. [Więcej informacji](https://bard.google.com/faq#coding)content\_copy

Funkcja `get_parent()` zwraca ścieżkę zasobu nadrzędnego do tworzenia procesorów. Argument `client` jest klientem DocumentAI. Argumenty `PROJECT_ID` i `API_LOCATION` są identyfikatorem projektu i lokalizacją punktu końcowego API.

Python

```
def get_client_and_parent() -> Tuple[docai.DocumentProcessorServiceClient, str]:
    client = get_client()
    parent = get_parent(client)
    return client, parent
```

Zachowaj ostrożność podczas korzystania z kodu. [Więcej informacji](https://bard.google.com/faq#coding)content\_copy

Funkcja `get_client_and_parent()` łączy `get_client()` i `get_parent()`, zwracając zarówno klienta, jak i ścieżkę zasobu nadrzędnego.
