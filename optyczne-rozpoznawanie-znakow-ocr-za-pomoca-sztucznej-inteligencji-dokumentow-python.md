---
description: >-
  Informacje o tym ćwiczeniu (w Codelabs) tematOstatnia aktualizacja: cze 20,
  2023 koło_kontaAutorzy: Holt Skinner
---

# Optyczne rozpoznawanie znaków (OCR) za pomocą sztucznej inteligencji dokumentów (Python)

### [1. Przegląd](https://codelabs.developers.google.com/codelabs/docai-ocr-python#0)

### **Co to jest dokumentowa sztuczna inteligencja?** <a href="#what-is-document-ai" id="what-is-document-ai"></a>

[Document AI](https://cloud.google.com/document-ai/docs) to rozwiązanie do rozumienia dokumentów, które pobiera nieustrukturyzowane dane (np. dokumenty, e-maile, faktury, formularze itp.) i ułatwia ich zrozumienie, analizowanie i wykorzystanie . Interfejs API zapewnia strukturę poprzez klasyfikację treści, wyodrębnianie jednostek, zaawansowane wyszukiwanie i nie tylko.

W tym laboratorium dowiesz się, jak wykonać optyczne rozpoznawanie znaków przy użyciu interfejsu API Document AI w języku Python.

Wykorzystamy plik PDF klasycznej powieści „Kubuś Puchatek” przez A.A. Milne, która niedawno stała się częścią [domeny publicznej](https://en.wikipedia.org/wiki/Public\_domain) w Stanach Zjednoczonych. Ten plik został zeskanowany i zdigitalizowany w [Książkach Google](https://www.google.com/books/edition/Winnie\_the\_Pooh/XB7hAAAAMAAJ).

### **Czego się nauczysz** <a href="#what-youll-learn" id="what-youll-learn"></a>

* Jak włączyć interfejs API Document AI
* Jak uwierzytelniać żądania API
* Jak zainstalować bibliotekę kliencką dla Pythona
* Jak korzystać z interfejsów API przetwarzania online i wsadowego
* Jak analizować tekst z pliku PDF

### **Czego potrzebujesz** <a href="#what-youll-need" id="what-youll-need"></a>

* Projekt Google Cloud
* Przeglądarka, taka jak [Chrome](https://www.google.com/chrome/) lub [Firefox](https://www.mozilla.org/en-US/firefox/)
* Znajomość Pythona (3.9+)



### [2. Konfiguracja i wymagania](https://codelabs.developers.google.com/codelabs/docai-ocr-python#1)



### **Konfiguracja środowiska we własnym tempie** <a href="#self-paced-environment-setup" id="self-paced-environment-setup"></a>

1. Zaloguj się do [Cloud Console](http://console.cloud.google.com/) i utwórz nowy projekt lub wykorzystaj ponownie istniejący. (Jeśli nie masz jeszcze konta Gmail lub Google Workspace, musisz [utworzyć je](https://accounts.google.com/SignUp)).

**Uwaga:** Dostęp do Cloud Console możesz łatwo uzyskać, zapamiętując jej adres URL, czyli console.cloud.google.com.

![Wybierz Projekt](https://codelabs.developers.google.com/static/codelabs/docai-ocr-python/img/SelectProject.png)

![Nowy projekt](https://codelabs.developers.google.com/static/codelabs/docai-ocr-python/img/NewProject.png)

![Uzyskaj identyfikator projektu](https://codelabs.developers.google.com/static/codelabs/docai-ocr-python/img/GetProjectID.png)

Zapamiętaj identyfikator projektu – unikalną nazwę wszystkich projektów Google Cloud. (Powyższy identyfikator projektu został już zajęty i nie będzie dla Ciebie odpowiedni, przepraszam!). Musisz podać ten identyfikator później jako `PROJECT_ID`.

**Uwaga:** Jeśli korzystasz z konta Gmail, możesz pozostawić domyślną lokalizację ustawioną na **Brak organizacji**. Jeśli korzystasz z konta Google Workspace, wybierz lokalizację odpowiednią dla Twojej organizacji.

2. Następnie musisz [włączyć rozliczenia](https://console.cloud.google.com/billing) w Cloud Console, aby móc korzystać z zasobów Google Cloud.

Pamiętaj, aby postępować zgodnie ze wszystkimi instrukcjami zawartymi w sekcji „Czyszczenie” Sekcja. W tej sekcji dowiesz się, jak zamknąć zasoby, aby nie naliczać opłat wykraczających poza ten samouczek. Nowi użytkownicy Google Cloud kwalifikują się do programu [bezpłatna wersja próbna o wartości 300 USD](http://cloud.google.com/free).

### **Uruchom Cloud Shell** <a href="#start-cloud-shell" id="start-cloud-shell"></a>

Chociaż Google Cloud możesz obsługiwać Google Cloud zdalnie ze swojego laptopa, to ćwiczenie z programowania wykorzystuje [Google Cloud Shell](https://cloud.google.com/cloud-shell/), środowisko wiersza poleceń działające w Chmura.

#### Aktywuj Cloud Shell <a href="#activate-cloud-shell" id="activate-cloud-shell"></a>

1. W konsoli Cloud kliknij **Aktywuj Cloud Shell** ![Aktywuj Cloud Shell](https://lh3.googleusercontent.com/H7JlbhKGHITmsxhQIcLwoe5HXZMhDlYue4K-SPszMxUxDjIeWfOHBfxDHYpmLQTzUmQ7Xx8o6OJUlANnQF0iBuUyfp1RzVad\_4nCa0Zz5LtwBlUZFXFCWFrmrWZLqg1MkZz2LdgUDQ)

![Aktywuj Cloud Shell](https://lh6.googleusercontent.com/zlNW0HehB\_AFW1qZ4AyebSQUdWm95n7TbnOr7UVm3j9dFcg6oWApJRlC0jnU1Mvb-IQp-trP1Px8xKNwt6o3pP6fyih947sEhOFI4IRF0W7WZk6hFqZDUGXQQXrw21GuMm2ecHrbzQ)

Jeśli nigdy wcześniej nie uruchamiałeś Cloud Shell, zostanie wyświetlony ekran pośredni (poniżej części ekranu) opisujący, co to jest. W takim przypadku kliknij **Kontynuuj** (i nigdy więcej tego nie zobaczysz). Oto jak wygląda ten jednorazowy ekran:

![Wprowadzenie do Cloud Shell](https://lh6.googleusercontent.com/kEPbNAo\_w5C\_pi9QvhFwWwky1cX8hr\_xEMGWySNIoMCdi-Djx9AQRqWn-\_\_DmEpC7vKgUtl-feTcv-wBxJ8NwzzAp7mY65-fi2LJo4twUoewT1SUjd6Y3h81RG3rKIkqhoVlFR-G7w)

Udostępnienie i połączenie z Cloud Shell powinno zająć tylko kilka chwil.![Chmura Shell](https://lh4.googleusercontent.com/pTv5mEKzWMWp5VBrg2eGcuRPv9dLInPToS-mohlrqDASyYGWnZ\_SwE-MzOWHe76ZdCSmw0kgWogSJv27lrQE8pvA5OD6P1I47nz8vrAdK7yR1NseZKJvcxAZrPb8wRxoqyTpD-gbhA)

Cloud Shell zapewnia dostęp terminalowy do maszyny wirtualnej hostowanej w chmurze. Maszyna wirtualna zawiera wszystkie potrzebne narzędzia programistyczne. Oferuje trwały katalog domowy o pojemności 5 GB i działa w Google Cloud, znacznie zwiększając wydajność sieci i uwierzytelnianie. Wiele, jeśli nie całość, pracy w tym laboratorium z kodowania można wykonać za pomocą zwykłej przeglądarki.

Po połączeniu z Cloud Shell powinieneś zobaczyć, że jesteś już uwierzytelniony i że projekt jest już ustawiony na Twój identyfikator projektu.

2. Uruchom następujące polecenie w Cloud Shell, aby potwierdzić uwierzytelnienie:

```sh
gcloud auth list
```

**Dane wyjściowe polecenia**

```
 Credentialed Accounts
ACTIVE  ACCOUNT
*      <my_account>@<my_domain.com>

To set the active account, run:
    $ gcloud config set account `ACCOUNT`
```

**Uwaga:** `gcloud` narzędzie wiersza poleceń to potężne i ujednolicone narzędzie wiersza poleceń w Google Cloud. Jest preinstalowany w Cloud Shell. Wśród jego funkcji znajduje się `gcloud` uzupełnianie kart w powłoce. Więcej informacji znajdziesz w [omówieniu narzędzia wiersza poleceń gcloud](https://cloud.google.com/sdk/gcloud/) .

```
gcloud config list project
```

**Dane wyjściowe polecenia**

```
[core]
project = <PROJECT_ID>
```

Jeśli tak nie jest, możesz to ustawić za pomocą tego polecenia:

```sh
gcloud config set project <PROJECT_ID>
```

**Dane wyjściowe polecenia**

```
Updated property [core/project].
```



### [3. Włącz interfejs API Document AI](https://codelabs.developers.google.com/codelabs/docai-ocr-python#2)



Zanim zaczniesz korzystać z Document AI, musisz włączyć API. Możesz to zrobić za pomocą [`gcloud` interfejsu wiersza poleceń](https://cloud.google.com/sdk/gcloud) lub konsoli Cloud.

### Użyj `gcloud` interfejsu wiersza polecenia <a href="#use-the-gcloud-cli" id="use-the-gcloud-cli"></a>

1. Jeśli nie używasz [Cloud Shell](https://cloud.google.com/shell), wykonaj czynności opisane w [Instalacja `gcloud` CLI](https://cloud.google.com/sdk/docs/install) na komputerze lokalnym.
2. Interfejsy API można włączyć za pomocą następujących poleceń `gcloud`.

```sh
gcloud services enable documentai.googleapis.com storage.googleapis.com
```

Powinieneś zobaczyć coś takiego:

```
Operation "operations/..." finished successfully.
```

### Skorzystaj z konsoli Cloud <a href="#use-the-cloud-console" id="use-the-cloud-console"></a>

Otwórz [konsolę Cloud](http://console.cloud.google.com/) w swojej przeglądarce.

1. Korzystając z paska wyszukiwania u góry konsoli, wyszukaj „Document AI API”, a następnie kliknij **Włącz**, aby użyć interfejsu API w Twoim projekcie Google Cloud

![Wyszukaj API](https://codelabs.developers.google.com/static/codelabs/docai-ocr-python/img/enable\_api.png)

1. Powtórz poprzedni krok dla interfejsu API Google Cloud Storage.

Teraz możesz korzystać z Document AI!



### [4. Utwórz i przetestuj procesor](https://codelabs.developers.google.com/codelabs/docai-ocr-python#3)

\


Najpierw musisz utworzyć instancję procesora OCR dokumentu, która przeprowadzi ekstrakcję. Można to zrobić za pomocą konsoli Cloud lub [interfejsu API do zarządzania procesorami](https://cloud.google.com/python/docs/reference/documentai/latest/google.cloud.documentai\_v1beta3.services.document\_processor\_service.DocumentProcessorServiceClient#google\_cloud\_documentai\_v1beta3\_services\_document\_processor\_service\_DocumentProcessorServiceClient\_create\_processor).

### Konsola chmurowa <a href="#cloud-console" id="cloud-console"></a>

1. W konsoli przejdź do [Omówienie platformy Document AI](https://console.cloud.google.com/ai/document-ai)![Konsola przeglądu AI dokumentu](https://codelabs.developers.google.com/static/codelabs/docai-ocr-python/img/DocAIOverviewConsole.png)
2. Kliknij **Przeglądaj procesory** i wybierz **OCR dokumentu**![Procesory](https://codelabs.developers.google.com/static/codelabs/docai-ocr-python/img/ProcessorsConsole.png)
3. Nadaj mu nazwę `codelab-ocr` (lub inną, którą zapamiętasz) i wybierz najbliższy region z listy.
4. Kliknij **Utwórz**, aby utworzyć procesor
5. Skopiuj identyfikator procesora. Musisz później użyć tego w swoim kodzie.![Identyfikator procesora](https://codelabs.developers.google.com/static/codelabs/docai-ocr-python/img/ProcessorID.png)

Możesz przetestować swój procesor w konsoli, przesyłając dokument. Kliknij **Prześlij dokument testowy** i wybierz dokument do analizy.

Poniżej możesz pobrać plik PDF, który zawiera pierwsze 3 strony naszej powieści.

![Strona tytułowa](https://codelabs.developers.google.com/static/codelabs/docai-ocr-python/img/Winnie\_the\_Pooh\_Titlepage.jpg)

[file\_downloadściągnij PDF](https://storage.googleapis.com/cloud-samples-data/documentai/codelabs/ocr/Winnie\_the\_Pooh\_3\_Pages.pdf)

Twoje dane wyjściowe powinny wyglądać tak:![Przeanalizowana książka](https://codelabs.developers.google.com/static/codelabs/docai-ocr-python/img/ParsedBook.png)

### Biblioteka klienta Pythona <a href="#python-client-library" id="python-client-library"></a>

Wykonaj te ćwiczenia z programowania, aby dowiedzieć się, jak zarządzać procesorami Document AI za pomocą biblioteki klienta Python:

[Zarządzanie procesorami Document AI za pomocą języka Python — Codelab](https://codelabs.developers.google.com/codelabs/docai-manage-processors-python)
