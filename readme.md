# Okay Selenium Testy

Aplikace určená k testování produkčních webů společnosti OKAY s.r.o.

<br>

## Závislosti

K běhu aplikace je potřeba mít v počítači nainstalovaný Python 3.8 a vyšší. Také je potřeba splnit závislosti definované v souboru `requirements.txt`. Testy běží v prohlížeči Google Chrome, je tedy potřeba mít nainstalovaný nejen tento prohlížeč, ale také aplikaci Chromedriver - dostupná [zde](https://chromedriver.chromium.org/).

<br>

## Instalace

Aplikaci stáhněte z tohoto repozitáře, případně vytvořte kopii repozitáře na svém počítači / serveru. Před prvním spuštěním je potřeba nainstalovat potřebné balíčky:

```
pip install -r requirements.txt
```

Následně je potřeba vytvořit kopii souboru `config_sample.json` a změnit nastavení.

```
cp config_sample.json config.json
```

```json
{
    "defaults": {
        "delay": 10,            // čas mezi úkony (sec)
        "is_headless": 0,       // render v paměti (bool)
        "is_email": 0,          // notifikace email (bool)
        "is_slack": 0,          // notifikace slack (bool)
    },
    "secrets": {
        "mail_to": "",          // mail pro notifikaci
        "mail_from": "",        // google mail pro odesílání
        "mail_password": "",    // heslo k tomuto mailu
        "slack_token": "",      // API token do slacku
        "slack_channel": ""     // ID kanálu ve slacku
    }
}
```

Pokud chcete testy spouštět automaticky, je potřeba nastavit CRON.

<br>

## Jak psát testy

Pro testování jsou k dispozici dvě třídy, a sice `OkayTest` a `JenaTest` - zvolte si tu, která odpovídá testované stránce. Testy píšete v Pythonu pomocí jednoduchých metod.

**Příklad:**
```python
from okay_tests import OkayTest

test = OkayTest(name="okaysk_menu")
test.open_url(url="https://www.okay.sk/")
test.open_random_menu_items(3)
test.open_random_footer_items(3)
test.abort()
```

Při inicializaci testu je možno použít několik volitelných argumentů, které mohou změnit chování testu. Zde jsou nejdůležitější z nich:

```python
test = OkayTest(
    name="okaysk_menu",     # vlastní název testu
    theme="120943050794",   # ID šablony v Shopify
    is_mobile=True,         # aktivuje emulátor mobilu
    delay=5                 # změní výchozí čas mezi úkony
)
```

<br>

## Použitelné metody (abecedně)

Všechny níže uvedené metody jsou použitelné jak pro třídu `OkayTest` i `JenaTest`.

<br>

### **abort**

Zavře prohlížeč a ukončí probíhající test.

**Příklad:**
```python
test.abort()
```

<br>

### **add_to_cart**

Přidá aktuální produkt do košíku. Aby metoda fungovala, musí se test zrovna nacházet na detailu nějakého produktu.

**Příklad:**
```python
test.add_to_cart()
```

<br>

### **check_services**

Pokusí se zaškrtnout nábytkové služby v košíku a následně vytvoří printscreen košíku. Je potřeba definovat služby jako `list` (pole) obsahující `variant ID` těchto služeb.

**Příklad:**
```python
test.check_services(services=["40968686796951", "40968686829719"])
test.check_services(services=["40968686928023"])
```

Argument `services` je povinný.

<br>

### **choose_delivery**

Zvolí druh dopravy definovaný argumentem `delivery`. Je potřeba, aby se test zrovna nacházel ve fázi volby dopravy.

**Příklad:**
```python
test.choose_delivery(delivery='na moju adresu', proceed=True)
```

Argument `delivery` je povinný a musí odpovídat způsobu dopravy na daném webu. Argument `proceed` je nepovinný a pokud jej nastavíte `True`, bude test pokračovat k volbě platby (výchozí hodnota je `False`).

<br>

### **choose_payment**

Zvolí druh platby definovaný argumentem `payment`. Je potřeba, aby se test zrovna nacházel ve fázi volby platby.

**Příklad:**
```python
test.choose_payment(payment='na moju adresu', proceed=True)
```

Argument `payment` je povinný a musí odpovídat způsobu platby na daném webu. Argument `proceed` je nepovinný a pokud jej nastavíte `True`, bude test pokračovat a dokončí objednávku (výchozí hodnota je `False`).

<br>

### **confirm_order**

Slouží k ověření, zda byla objednávka úspěšně dokončena. Ověření proběhne tím, že se test pokusí kliknout na prvek na thank-you stránce.

**Příklad:**
```python
test.confirm_order()
```

<br>

### **empty_cart**

Otevře aktuální košík a smaže všechny položky v něm. Tato metoda nevyvolá žádnou chybu v případě, kdy bude košík prázdný.

**Příklad:**
```python
test.empty_cart()
```

<br>

### **goto_checkout**

Pokračovat z košíku do checkoutu. Pokud se zrovna nenacházíte v košíku, tato metoda jej otevře za vás. Zároveň vyplní všechny potřebné zákaznické detaily (pokud jsou potřeba) a pokračuje k volbě dopravy.

**Příklad:**
```python
test.goto_checkout()
```

<br>

### **handle_gopay**

Projde platební bránou gopay až po zadání čísla karty a potom se vrátí zpět do eshopu, čímž stornuje objednávku.

**Příklad:**
```python
test.handle_gopay()
```

<br>

### **log_results**

Za vstup vezme `list` (pole), které se skládá z libovolného počtu `dictionary` a uloží je jako výstup do souboru.

**Příklad:**
```python
test.log_results(
    name='(2) Do 50 kg', 
    url='https://www.okay.sk/collections/mikrovlnne-rury-a-mini-rury',
    logs=[
        {'Zásielkovňa': '1,00 €', 'Doručiť na moju adresu': '2,00& €'},
        {'Bankový prevod': '-', 'Dobierka': '0 €', 'Platba na výdajni': '-'}
    ]
)
```

Všechny argumenty, tedy `name`, `url` a `logs` jsou povinné.

<br>

### **new_test**

Tuto metodu je vhodné používat ve všech `for` a `while` smyčkách na začátku každé iterace. Nastaví výchozí hodnoty testu během jednotlivých iterací, vyčistí cache a cookies.

**Příklad:**
```python
test.new_test()
```

<br>

### **open_product**

Najde na stránce první nejprodávanější produkt skladem. Pokud takový produkt neexistuje, vybere první produkt v kolekci při aktuálním řazení.

**Příklad:**
```python
test.open_product()
```

<br>

### **open_random_menu_items**

Vezme seznam všech položek v hlavním menu, náhodně klikne na tolik, kolik je definováno argumentem `items` a pořídí printscreeny.

**Příklad:**
```python
test.click_random_mainmenu_items(items=3)
```

Argument `items` je povinný.

<br>

### **open_random_footer_items**

Vezme seznam všech položek v patičkovém menu, náhodně klikne na tolik, kolik je definováno argumentem `items` a pořídí printscreeny.

**Příklad:**
```python
test.click_random_footer_items(items=3)
```

Argument `items` je povinný.

<br>

### **open_specific_menu_item**

Otevře položku v hlavním menu, která odpovídá řetězci zadanému argumentem `text`.

**Příklad:**
```python
test.click_specific_mainmenu_item(text='Televízory')
```

Argument `text` je povinný.

<br>

### **open_url**

Otevře webovou stránku definovanou argumentem `url`.

**Příklad"**
```python
test.open_url(url='https://www.okay.sk/')
```

Argument `url` je povinný.

<br>

### **parse_delivery**

Vyčte seznam všech způsobů dopravy a vrátí jej jako `dictionary`.

**Příklad:**
```python
delivery = test.parse_delivery()
```

<br>

### **parse_payment**

Vyčte seznam všech platebních metod a vrátí jej jako `dictionary`.

**Příklad:**
```python
payment = test.parse_payment()
```

<br>

### **search_for**

Vyhledá frázi definovanou argumentem `text`.

**Příklad:**
```python
test.search_for(text='mobilný telefón')
```

Argument `text` je povinný.

<br>

### **set_filter**

Nastaví filtr v kolekci podle jeho jména `name` a hodnoty `value`.

**Příklad:**
```python
test.set_filter(name='výrobcovia', value='lg')
```

Oba argumenty jsou povinné.

<br>

## Poznámky

Každá metoda, kterou můžete v testu použít, má navíc možnost zadání argumentu `screenshots`. Pokud je tento argument nastaven `False`, v průběhu této metody nebudou pořízeny žádné printscreeny (výchozí hodnota je `True`).

**Příklad:**
```python
test.click_random_mainmenu_items(items=3, screenshots=False)
```

<br>

## Další příklady

Součástí repozitáře jsou také spubory `okaysk__samples.py` a `jena__samples.py`, které obsahují základní baterii testů. Můžete jej použít jako referenční příklady při psaní vlastních testů.

<br>

*(C) 2021 OKAY s.r.o.*