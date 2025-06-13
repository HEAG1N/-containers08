Integrare continuă cu Github Actions

Scopul lucrării

În cadrul acestei lucrări vom învăța să configuram integrarea continuă cu ajutorul Github Actions.
Sarcina

Crearea unei aplicații Web, scrierea testelor pentru aceasta și configurarea integrării continue cu ajutorul Github Actions pe baza containerelor.

Execuție

Cream un repozitoriu containers08 și il copiam pe computer.

Crearea aplicației Web

În directorul containers08 cream directorul ./site. În directorul ./site va fi plasată aplicația Web pe baza PHP.

Cream în directorul ./site aplicația Web pe baza PHP cu următoarea structură:

![433015772-49c09ff8-ed04-4f1c-ab22-1d5c2328de27](https://github.com/user-attachments/assets/4c189d46-9381-4fa5-b27c-a4081ce9f30d)

Fișierul modules/database.php conține clasa Database pentru lucru cu baza de date. Pentru lucru cu baza de date folosim SQLite. Fișierul modules/page.php conține clasa Page pentru lucru cu paginile.:

![433015856-77dce227-78dd-4f67-b83a-e9c2f19c806b](https://github.com/user-attachments/assets/7d30f885-46d6-41c2-8207-36f3fca7c17b)

Fișierul templates/index.tpl conține șablonul paginii.

![433015926-56600b67-473b-4882-8f0b-51cbb3fae81e](https://github.com/user-attachments/assets/146f200e-42fd-4b97-a484-24f5b0912636)

Fișierul styles/style.css conține stilurile pentru pagina.

![433015938-87db4fac-1fe8-4747-a77b-9ad7337adfec](https://github.com/user-attachments/assets/ff41595e-c04f-43ea-b718-83471da1ddeb)

Fișierul index.php conține codul pentru afișarea paginii.

![433016164-7b41b3ab-4642-4329-88a8-709d9427f90b](https://github.com/user-attachments/assets/0e82217d-39ea-4fe3-8b36-f79b61c3d503)

Fișierul config.php conține setările pentru conectarea la baza de date.

![433016291-8773b63d-6afe-4d94-8bc0-450e5a61c0ed](https://github.com/user-attachments/assets/2e61f933-8974-4478-9d68-e96b4d6147ed)

Pregătirea fișierului SQL pentru baza de date

Cream în directorul ./site directorul ./sql. În directorul creat cream fișierul schema.sql cu următorul conținut:

![433016322-e0ed446f-d788-4edd-b862-9ea224563d85](https://github.com/user-attachments/assets/1bc24608-93a1-42dc-aac4-fe100211a7d1)

Crearea testelor

Cream în rădăcina directorului containers08 directorul ./tests. În directorul creat cream fișierul testframework.php cu următorul conținut:

![433016420-d55f694e-456c-4c1f-b3fe-21aef4fcb324](https://github.com/user-attachments/assets/7b327c77-dd68-4c26-8277-82688b47a1b3)

Cream în directorul ./tests fișierul tests.php.

Adăugam în fișierul ./tests/tests.php teste pentru toate metodele clasei Database, precum și pentru metodele clasei Page.
```
<?php

require_once __DIR__ . '/testframework.php';

$config = [
    "db" => [
        "path" => "/var/www/db/db.sqlite"
    ]
];
require_once '/var/www/html/modules/database.php';
require_once '/var/www/html/modules/page.php';

$tests = new TestFramework();

// test 1: check database connection
function testDbConnection() {
    global $config;
    $db = new Database($config["db"]["path"]);
    return assertExpression(true, "Database connection successful", "Failed to connect to database");
}

// test 2: test count method
function testDbCount() {
    global $config;
    $db = new Database($config["db"]["path"]);
    $count = $db->Count("page");
    return assertExpression($count == 3, "Count method returned correct value: $count", "Count method failed, expected 3 but got $count");
}

// test 3: test create method
function testDbCreate() {
    global $config;
    $db = new Database($config["db"]["path"]);
    $initialCount = $db->Count("page");
    
    $newId = $db->Create("page", [
        "title" => "Test Page",
        "content" => "Test Content"
    ]);
    
    $newCount = $db->Count("page");
    $record = $db->Read("page", $newId);
    
    $db->Delete("page", $newId); // Clean up
    
    return assertExpression(
        $newCount == $initialCount + 1 && $record["title"] == "Test Page" && $record["content"] == "Test Content",
        "Create method created a record successfully",
        "Create method failed"
    );
}

// test 4: test read method
function testDbRead() {
    global $config;
    $db = new Database($config["db"]["path"]);
    
    $record = $db->Read("page", 1);
    
    return assertExpression(
        $record && $record["id"] == 1 && $record["title"] == "Page 1" && $record["content"] == "Content 1",
        "Read method returned correct record",
        "Read method failed"
    );
}

// test 5: test update method
function testDbUpdate() {
    global $config;
    $db = new Database($config["db"]["path"]);
    
    $newId = $db->Create("page", [
        "title" => "Update Test",
        "content" => "Before Update"
    ]);
    
    $db->Update("page", $newId, [
        "title" => "Updated Title",
        "content" => "After Update"
    ]);
    
    $record = $db->Read("page", $newId);
    
    $db->Delete("page", $newId); // Clean up
    
    return assertExpression(
        $record["title"] == "Updated Title" && $record["content"] == "After Update",
        "Update method updated the record successfully",
        "Update method failed"
    );
}

// test 6: test delete method
function testDbDelete() {
    global $config;
    $db = new Database($config["db"]["path"]);
    
    $newId = $db->Create("page", [
        "title" => "Delete Test",
        "content" => "To Be Deleted"
    ]);
    
    $initialCount = $db->Count("page");
    $db->Delete("page", $newId);
    $newCount = $db->Count("page");
    
    return assertExpression(
        $newCount == $initialCount - 1,
        "Delete method deleted the record successfully",
        "Delete method failed"
    );
}

// test 7: test fetch method
function testDbFetch() {
    global $config;
    $db = new Database($config["db"]["path"]);
    
    $results = $db->Fetch("SELECT * FROM page WHERE id <= 3 ORDER BY id");
    
    return assertExpression(
        count($results) == 3 && 
        $results[0]["id"] == 1 && 
        $results[1]["id"] == 2 && 
        $results[2]["id"] == 3,
        "Fetch method returned correct results",
        "Fetch method failed"
    );
}

// test 8: test page rendering
function testPageRender() {
    $page = new Page('/var/www/html/templates/index.tpl');
    
    $data = [
        "title" => "Test Title",
        "content" => "Test Content"
    ];
    
    $rendered = $page->Render($data);
    
    return assertExpression(
        strpos($rendered, "Test Title") !== false && 
        strpos($rendered, "Test Content") !== false,
        "Page rendering works correctly",
        "Page rendering failed"
    );
}

// add tests
$tests->add('Database connection', 'testDbConnection');
$tests->add('Table count', 'testDbCount');
$tests->add('Data create', 'testDbCreate');
$tests->add('Data read', 'testDbRead');
$tests->add('Data update', 'testDbUpdate');
$tests->add('Data delete', 'testDbDelete');
$tests->add('Data fetch', 'testDbFetch');
$tests->add('Page render', 'testPageRender');

// run tests
$tests->run();

echo "Test results: " . $tests->getResult();
```
Cream Dockerfile

Cream în directorul rădăcină al proiectului fișierul Dockerfile cu următorul conținut:

![433017066-c3e70d60-7e4b-4468-9e0b-e670cb0c1263](https://github.com/user-attachments/assets/6a8eb490-c98b-4efd-afb6-7544da9c594b)

Configurarea Github Actions

Cream în directorul rădăcină al proiectului fișierul .github/workflows/main.yml cu următorul conținut:
```
name: CI

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Build the Docker image
        run: docker build -t containers08 .
      - name: Create `container`
        run: docker create --name container --volume database:/var/www/db containers08
      - name: Copy tests to the container
        run: docker cp ./tests container:/var/www/html
      - name: Up the container
        run: docker start container
      - name: Run tests
        run: docker exec container php /var/www/html/tests/tests.php
      - name: Stop the container
        run: docker stop container
      - name: Remove the container
        run: docker rm container
```
Pornire și testare

Trimitem modificările în repozitoriu și ne asiguram că testele trec cu succes. Pentru aceasta, trecem la fila Actions în repozitoriu și așteptam finalizarea sarcinii.


Răspunsuri la întrebări:
Ce este integrarea continuă?

Integrarea continuă (Continuous Integration, CI) este o practică DevOps care presupune integrarea frecventă a modificărilor de cod într-un depozit partajat. De fiecare dată când un dezvoltator face o modificare și o urcă în repository (de exemplu, pe ramura main), sunt executate automat procese precum:

rularea testelor unitare,

construirea aplicației,

verificarea stilului codului.

Scopul CI este de a detecta rapid erorile și de a îmbunătăți calitatea codului, asigurând o integrare rapidă și sigură a modificărilor în codul sursă.
Pentru ce sunt necesare testele unitare? Cât de des trebuie să fie executate?

Testele unitare sunt folosite pentru a verifica funcționarea corectă a componentelor individuale (unităților) ale unei aplicații, cum ar fi metode sau clase.

Acestea sunt necesare pentru:

a crește încrederea în calitatea codului,

a permite refactorizarea codului fără teamă de a introduce erori,

a facilita dezvoltarea în echipă.
Care modificări trebuie făcute în fișierul .github/workflows/main.yml pentru a rula testele la fiecare solicitare de trage (Pull Request)?

Pentru a rula CI și testele la fiecare pull request, trebuie să adăugăm în secțiunea on: a fișierului .github/workflows/main.yml următoarea linie:
```
on:
push:
  branches:
    - main
pull_request:
  branches:
    - main
```
Aceasta va declanșa workflow-ul și pentru solicitările de trage (pull requests) către ramura main.
Ce trebuie adăugat în fișierul .github/workflows/main.yml pentru a șterge imaginile create după testare?

Pentru a șterge imaginea Docker creată după rularea testelor, putem adăuga următorul pas la sfârșitul workflow-ului, după ștergerea containerului:
```
  - name: Remove the Docker image
    run: docker rmi containers08 || true
```
docker rmi containers08 șterge imaginea.

|| true asigură că procesul continuă chiar dacă ștergerea imaginii eșuează (de exemplu, dacă imaginea a fost deja ștearsă).
Concluzii

În urma realizării lucrării de laborator №8, am învățat să:

construim o aplicație web simplă în PHP,

folosim o bază de date SQLite și să accesăm date printr-o clasă dedicată (Database),

utilizăm o clasă de tip Page pentru a încărca șabloane dinamice,

scriem un cadru simplu de testare pentru testele unitare,

configurăm un Dockerfile pentru a rula aplicația și testele într-un container,

configurăm un workflow GitHub Actions pentru a automatiza procesul de integrare continuă.

Această lucrare a demonstrat utilitatea testării automate și a CI în dezvoltarea modernă, oferind un mediu controlat și predictibil pentru validarea codului nou adăugat. Automatizarea testelor prin GitHub Actions contribuie la un proces de dezvoltare mai sigur și eficient.
