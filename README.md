# programacio

RA 8-9 | ORM de programació

## Què aprendrem?

* Concepte ORM.
* c# Microsoft Entity Framework Core

## Introducció

### Què és un ORM?

Un **ORM** (Object-Relational Mapping) és una tècnica que permet transformar dades entre un sistema orientat a objectes (com C#) i una base de dades relacional (com SQL Server). L’objectiu principal és evitar escriure manualment codi SQL per accedir, inserir o modificar dades, substituint-lo per codi en el nostre llenguatge de programació que es tradueix automàticament a SQL.

### Què és Entity Framework Core?

**Entity Framework Core (EF Core)** és un ORM desenvolupat per Microsoft per a .NET. Permet treballar amb bases de dades mitjançant objectes de C#. És lleuger, multiplataforma i compatible amb diferents sistemes de bases de dades com SQL Server, SQLite, PostgreSQL, etc.

### Què és CRUD?

**CRUD** és un acrònim que fa referència a les quatre operacions bàsiques que es poden fer amb dades en una aplicació:

- **C**reate (Crear): inserir noves dades a la base de dades.  
- **R**ead (Llegir): recuperar dades existents.  
- **U**pdate (Actualitzar): modificar dades existents.  
- **D**elete (Eliminar): suprimir dades.

Aquestes operacions són fonamentals en gairebé qualsevol aplicació que treballi amb una base de dades. Un **ORM** com Entity Framework Core facilita la implementació d’aquestes operacions mitjançant codi orientat a objectes, sense necessitat d’escriure instruccions SQL directament. Per exemple, en lloc d’escriure una consulta `SELECT` per llegir dades, podem utilitzar mètodes de LINQ sobre col·leccions d’objectes en C#.

En resum, el CRUD és el conjunt d’operacions que es fan habitualment amb les dades, i l’ORM ens ofereix una forma més senzilla, segura i mantenible de dur-les a terme.


### Avantatges i inconvenients d'usar un ORM

| Avantatges                        | Inconvenients                        |
|----------------------------------|--------------------------------------|
| Estalvia codi SQL manual         | Pot ser més lent que SQL optimitzat  |
| Millor mantenibilitat del codi   | Aprenentatge de l’ORM                |
| Validació i restriccions amb codi| Menys control sobre consultes        |
| Portabilitat entre bases de dades| Depèn del rendiment de l’ORM         |

## Entity Framework Core

### Minimal Reproducible Example

Treballarem sobre un exemple mínim però que treballi amb `DbContext`.

En una aplicació es mapegen a taules multitut de classes, nosaltres només en mapejarem una que serveixi d'exemple:

```c#
public class Llibre {
    public string ISBN { get; set; }
    public int nombre_pagines { get; set; }
}
```

Veurem com Entity Framework Core s'encarrega de crear la taula on persistir instàncies de `Llibre`.

### Conceptes necessaris

#### LINQ

**LINQ** (Language Integrated Query) és una manera d’escriure consultes dins de C# per manipular col·leccions de dades com ara llistes, arrays o consultes de bases de dades.

Per exemple, si tenim una `List<Llibre>` i volem obtenir només els llibres amb més de 15 pàgines:

```c#
var llibresFiltrats = 
    llibres
    .Where(llibre => llibre.nombre_pagines > 15 )
    .ToList();
```

Aquest tipus de consulta és molt similar a les que farem quan llegim dades de la base de dades amb EF Core.

#### CRUD

Les operacions bàsiques que fem amb una base de dades es coneixen com a **CRUD**:

* **Create**: Inserir nous registres.
* **Read**: Llegir dades.
* **Update**: Modificar dades existents.
* **Delete**: Esborrar dades.

Entity Framework ens permet fer aquestes operacions usant objectes de C# i mètodes com `.Add()`, `.Find()`, `.Update()`, `.Remove()` i `.SaveChanges()`.

### La cadena de connexió

Una **cadena de connexió** (connection string) és un text que conté tota la informació necessària per connectar-se a una base de dades: servidor, nom de la base de dades, usuari, contrasenya, etc.

Exemple que farem servir:

```
"Server=localhost;Database=LlibresDb;User Id=sa;Password=PrimerDAW2025!;TrustServerCertificate=True;"
```


### La base de dades

Cal tenir engegat un **SQL Server**. Es pot fer de dues maneres:

1. Instal·lant-lo a la màquina.
2. Utilitzant un **contenidor Docker**, amb aquesta comanda:

```bash
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=PrimerDAW2025!" -p 1433:1433 --name sql1 --hostname sql1 -d mcr.microsoft.com/mssql/server:2022-latest
```

Ens podem connectar a la base de dades amb eines com:

* Microsoft SQL Server Management Studio (SSMS)
* DBeaver
* Directament des del nostre codi

### Instal·lar paquet EF Core

Per poder fer servir EF Core, cal instal·lar el paquet `Microsoft.EntityFrameworkCore.SqlServer`.

Es pot fer de dues maneres:

* **Des de consola**:
    ```bash
    dotnet add package Microsoft.EntityFrameworkCore.SqlServer
    ```

* **Des de Visual Studio**:
    - Obrir el gestor de paquets NuGet
    - Buscar `Microsoft.EntityFrameworkCore.SqlServer`
    - Clicar "Instal·lar"

### El primer programa amb Entity Framework Core

Aquest és el primer exemple complet:

```c#
using System.ComponentModel.DataAnnotations;
using System.Linq;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Logging;

// Classe que representa la taula 'Llibres' a la base de dades
public class Llibre {
    [Key] // Indica que ISBN és la clau primària
    [MaxLength(30)] // Limita la longitud màxima del camp
    public string ISBN { get; set; }
    public int nombre_pagines { get; set; }
}

// Classe que hereta de DbContext i configura la connexió i les taules
public class LlibresDbContext : DbContext
{
    public DbSet<Llibre> Llibres { get; set; } // Representa la taula de llibres

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        // Configura la cadena de connexió i activa els logs de consulta
        optionsBuilder
        .UseSqlServer("Server=localhost;Database=LlibresDb;User Id=sa;Password=x1ng1x1ng1!;TrustServerCertificate=True;")
        .LogTo(System.Console.WriteLine, LogLevel.Information)
        .EnableSensitiveDataLogging(true);
    }
}

class Program
{
    static void Main(string[] args)
    {
        // Crea una llista de llibres
        var llibres = new List<Llibre>
        {
            new Llibre { ISBN = "0", nombre_pagines = 10 },
            new Llibre { ISBN = "1", nombre_pagines = 20 },
            new Llibre { ISBN = "2", nombre_pagines = 5 },
            new Llibre { ISBN = "3", nombre_pagines = 25 }
        };

        using var ctx = new LlibresDbContext();

        // Crea la base de dades si no existeix
        ctx.Database.EnsureCreated();

        // Afegeix els llibres i guarda els canvis a la base de dades
        ctx.Llibres.AddRange(llibres);
        ctx.SaveChanges();
    }   
}
```

### Què són les Data Annotations?

Les **Data Annotations** són atributs que s’afegeixen a les propietats de les classes per afegir metadades. EF Core les utilitza per definir restriccions, claus primàries, longituds màximes, etc.

Exemples:

* `[Key]` → Clau primària
* `[MaxLength(30)]` → Longitud màxima d’un camp de text
* `[Required]` → No permet valors nuls

### Com configurem DbContext?

La classe `DbContext` és el cor d’EF Core. Conté:

* La definició de les taules (com `DbSet<Llibre>`)
* La configuració de la connexió (`OnConfiguring`)
* Mecanismes per accedir, modificar i persistir dades

La configuració de la connexió es fa a `OnConfiguring`, indicant la cadena de connexió i opcions com el log de consultes SQL.

### Com configurem DbContext?

A continuació expliquem les dues opcions de configuració que hem utilitzat dins del mètode `OnConfiguring` de `DbContext`.

```c#
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer("Server=localhost;Database=LlibresDb;User Id=sa;Password=x1ng1x1ng1!;TrustServerCertificate=True;")
        .LogTo(System.Console.WriteLine, LogLevel.Information)
        .EnableSensitiveDataLogging(true);
}
```

#### 1. `.UseSqlServer(...)`

Aquesta línia especifica que volem fer servir **SQL Server** com a sistema de base de dades, i proporciona la **cadena de connexió** amb tota la informació necessària per connectar-nos-hi.

Paràmetres més destacats:
- `Server=localhost`: indica que el servidor és local.
- `Database=LlibresDb`: nom de la base de dades que utilitzarem.
- `User Id=sa; Password=...`: credencials d'accés.
- `TrustServerCertificate=True`: evita errors de certificat en entorns locals.

Sense aquesta línia, Entity Framework no sabria on connectar-se.

#### 2. `.LogTo(...)` i `.EnableSensitiveDataLogging(true)`

Aquestes dues línies activen **el registre de les operacions** que fa Entity Framework. És molt útil mentre desenvolupem o estem aprenent com funciona EF Core.

- `.LogTo(System.Console.WriteLine, LogLevel.Information)`: imprimeix totes les consultes i informació rellevant a la consola.
- `.EnableSensitiveDataLogging(true)`: mostra també els valors dels paràmetres (per exemple, les dades que afegim o consultem).

> ⚠️ Aquesta opció mostra dades sensibles, per tant **només s’ha d’utilitzar en entorns de desenvolupament o aprenentatge**, mai en producció.

Aquesta configuració ens permet:
- Connectar correctament amb la base de dades SQL Server.
- Veure com EF Core genera i executa consultes SQL, cosa molt útil per entendre el funcionament intern.


### Ús de transaccions amb Entity Framework Core

En alguns casos, especialment quan fem **múltiples operacions** sobre la base de dades que han d’executar-se com un **bloc indivisible**, és convenient fer servir una **transacció**.

Una **transacció** garanteix que totes les operacions dins seu s'executen correctament o, en cas d’error, **cap d’elles es persisteix**. Això evita que la base de dades quedi en un estat inconsistent.

Entity Framework Core ens permet controlar transaccions manualment amb `BeginTransaction(...)`.

Exemple d’ús:

```c#
using var transaction = ctx.Database.BeginTransaction(isolationLevel: System.Data.IsolationLevel.ReadCommitted);
try
{
    // Guardem els canvis
    ctx.SaveChanges();

    // Si tot ha anat bé, confirmem la transacció
    transaction.Commit();
}
catch (Exception ex)
{
    // Si hi ha algun error, desfer tots els canvis
    Console.WriteLine($"Error: {ex.Message}");
    transaction.Rollback();
}
```

#### Explicació:

- **`BeginTransaction(...)`**: Inicia una nova transacció. Podem especificar el nivell d'aïllament (en aquest cas, `ReadCommitted`, que és el valor per defecte en SQL Server).
- **`SaveChanges()`**: Guarda els canvis pendents a la base de dades.
- **`Commit()`**: Si tot ha funcionat, confirma la transacció.
- **`Rollback()`**: Si hi ha una excepció, es cancel·la tot el que s'hagi fet dins la transacció.

Investiga quins nivells d'isolació trobem dins `System.Data.IsolationLevel`.

### Operacions CRUD amb Entity Framework Core

Entity Framework Core ens permet fer les quatre operacions bàsiques sobre una base de dades: **Create, Read, Update, Delete** (CRUD). A continuació veiem com es fan cadascuna d’aquestes operacions utilitzant la classe `DbContext` i el nostre exemple amb `Llibre`.

---

#### 🟢 Create (Crear)

Per afegir una nova instància (registre) a la base de dades:

```c#
var nouLlibre = new Llibre { ISBN = "4", nombre_pagines = 100 };
ctx.Llibres.Add(nouLlibre);  // Afegim l’objecte al context
ctx.SaveChanges();           // Persistim els canvis a la base de dades
```

També es pot fer amb diversos llibres alhora:

```c#
var llibres = new List<Llibre>
{
    new Llibre { ISBN = "5", nombre_pagines = 80 },
    new Llibre { ISBN = "6", nombre_pagines = 120 }
};

ctx.Llibres.AddRange(llibres);
ctx.SaveChanges();
```

---

#### 🔵 Read (Llegir)

Per consultar dades de la base de dades, podem utilitzar **LINQ**:

Tots els llibres:

```c#
var totsElsLlibres = ctx.Llibres.ToList();
```

Llibres amb més de 50 pàgines:

```c#
var llibresGrans = ctx.Llibres
    .Where(llibre => llibre.nombre_pagines > 50)
    .ToList();
```

Cercar un llibre pel seu ISBN (clau primària):

```c#
var llibre = ctx.Llibres.Find("4");  // Retorna null si no existeix
```

---

#### 🟡 Update (Actualitzar)

Per modificar dades d’un registre, primer cal recuperar-lo, modificar-lo i després guardar els canvis:

```c#
var llibre = ctx.Llibres.Find("4");
if (llibre != null)
{
    llibre.nombre_pagines = 150;
    ctx.SaveChanges();  // EF detecta els canvis i genera la consulta UPDATE
}
```

També es pot fer amb diverses entitats modificades dins la mateixa sessió.

---

#### 🔴 Delete (Eliminar)

Per eliminar un registre, primer cal obtenir-lo i després usar `.Remove()`:

```c#
var llibre = ctx.Llibres.Find("4");
if (llibre != null)
{
    ctx.Llibres.Remove(llibre);
    ctx.SaveChanges();
}
```

També podem eliminar múltiples registres amb `.RemoveRange(...)`.

---

Aquestes operacions són automàticament traduïdes per EF Core a consultes SQL adequades (INSERT, SELECT, UPDATE, DELETE) segons el context i l’estat de les entitats. Això ens permet treballar sempre amb objectes i delegar a EF la comunicació amb la base de dades.


### Migracions amb Entity Framework Core

Entity Framework Core permet controlar els canvis a l’estructura de la base de dades mitjançant **migracions**. Això és molt útil per mantenir sincronitzat el model del nostre codi amb la base de dades, especialment en projectes que evolucionen amb el temps.

Amb les migracions podem:

- Crear la base de dades inicial a partir del model de C#.
- Actualitzar l'estructura de la base de dades quan afegim o canviem propietats.
- Compartir canvis entre diferents desenvolupadors de manera ordenada.

---

### Requisits previs

Per poder fer servir migracions, calen dues coses:

#### 📦 Paquet de migració

Instal·la el paquet necessari dins del projecte:

```bash
dotnet add package Microsoft.EntityFrameworkCore.Design
```

#### 🔧 Eina de línia de comandes

Instal·la l’eina `dotnet-ef` globalment (només cal fer-ho un cop):

```bash
dotnet tool install --global dotnet-ef
```

Un cop fet, pots verificar-ho amb:

```bash
dotnet ef --version
```

---

### Crear una migració

Quan tens definit un `DbContext` i unes classes de model, pots generar la primera migració així:

```bash
dotnet ef migrations add Inicial
```

Això crea:

- Una carpeta `Migrations/`
- Un fitxer de migració amb el codi per crear la base de dades
- Un fitxer `ModelSnapshot` que guarda l’estat actual del model

---

### Abans d'aplicar la migració

* Esborra la base de dades perquè la tornarem a crear.
* Esborra la línia `ctx.Database.EnsureCreated();`, ara no crearem la base de dades en executar el programa sino que ho farem nosaltres a voluntad quan volguem.


---

### Aplicar la migració

Per crear la base de dades o aplicar els canvis definits a la migració, executa:

```bash
dotnet ef database update
```

---

### Afegir nous canvis al model

Si afegim una nova propietat, per exemple:

```c#
public string Titol { get; set; }
```

Cal fer una nova migració:

```bash
dotnet ef migrations add AfegirTitol
dotnet ef database update
```

---

### Bona pràctica

Evita fer servir `EnsureCreated()` quan utilitzes migracions, ja que són mecanismes incompatibles. **Amb migracions, EF gestiona tota la creació i evolució de l'esquema de forma controlada.**

---

### Què hi trobarem dins la carpeta `Migrations/`?

Quan generem una migració amb `dotnet ef migrations add ...`, EF Core crea automàticament una carpeta anomenada **`Migrations/`** (si no existia abans). Aquesta carpeta conté tot el que EF necessita per mantenir un historial de l’estructura de la base de dades.

A dins hi trobarem:

---

#### 🗂️ Fitxer de migració

Exemple de nom:  
`20250512101532_Inicial.cs`

Aquest fitxer conté dues funcions importants:

- **`Up()`**: defineix les operacions per aplicar la migració (crear taules, afegir columnes, etc.).
- **`Down()`**: defineix com desfer els canvis (eliminar taules, revertir columnes, etc.).

Exemple (simplificat):

```c#
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.CreateTable(
        name: "Llibres",
        columns: table => new
        {
            ISBN = table.Column<string>(type: "nvarchar(30)", nullable: false),
            nombre_pagines = table.Column<int>(type: "int", nullable: false),
            Titol = table.Column<string>(type: "nvarchar(max)", nullable: true)
        },
        constraints: table =>
        {
            table.PrimaryKey("PK_Llibres", x => x.ISBN);
        });
}
```

---

#### 🗂️ Fitxer `ModelSnapshot`

Exemple:  
`LlibresDbContextModelSnapshot.cs`

Aquest fitxer mostra **l’estat actual del model** de dades. EF Core el fa servir per comparar l’estat anterior amb el nou quan generes una nova migració. No cal modificar-lo manualment.

---

#### 📌 Altres migracions

Cada vegada que facis una nova migració, EF afegirà un nou fitxer de migració, amb un nom basat en la data/hora i el nom que li hagis donat (ex: `AfegirTitol.cs`). Això crea un **historial complet de tots els canvis** a la base de dades.

---

Aquest sistema és molt útil perquè et permet:

- Fer un **seguiment dels canvis** que fas al model.
- **Aplicar, revertir o repetir migracions** segons calgui.
- Desplegar els canvis a altres entorns (producció, servidors, altres màquines).

> En resum: la carpeta `Migrations/` és com el **"git" de l'estructura de la base de dades**.


### La taula `__EFMigrationsHistory`

Quan utilitzem migracions amb Entity Framework Core, EF crea automàticament una **taula especial** a la base de dades anomenada:

```
__EFMigrationsHistory
```


Aquesta taula és **essencial per al funcionament de les migracions**. No la creem nosaltres manualment; EF Core la genera i l’actualitza cada vegada que fem:

```bash
dotnet ef database update
```

---

### 📋 Què conté aquesta taula?

Aquesta taula conté un registre de **totes les migracions que s’han aplicat correctament** a la base de dades. Cada fila representa una migració aplicada.

Columnes habituals:

| MigrationId                  | ProductVersion |
|-----------------------------|----------------|
| 20250512101532_Inicial      | 8.0.0          |
| 20250512112100_AfegirTitol  | 8.0.0          |

- **MigrationId**: coincideix amb el nom del fitxer de migració (sense extensió).
- **ProductVersion**: versió de EF Core utilitzada quan es va aplicar la migració.

---

### 🧠 Per què serveix?

EF Core consulta aquesta taula per saber:

- **Quines migracions ja han estat aplicades** a la base de dades.
- **Quines migracions falten per aplicar** (en cas que n’hi hagi de noves al projecte).
- Evitar aplicar una migració més d’un cop.
- Permetre revertir migracions si cal.

Això és molt útil quan diversos desenvolupadors treballen amb el mateix projecte, o quan despleguem l’aplicació a diferents entorns (dev, test, prod...).

---

### 🔒 Importància de no modificar-la

**No s’ha de modificar manualment** aquesta taula. Si s’elimina o manipula, EF Core **perdrà el seguiment** de quines migracions s’han aplicat, i podries corrompre l’estructura o provocar errors a `update`.

---

> En resum: `__EFMigrationsHistory` és com l’historial intern d’EF Core per controlar l’estat de la base de dades en relació al model definit al codi.


## Bibliografia

* [Entity Framework Core](https://learn.microsoft.com/en-us/ef/core/)
* [Getting Started with EF Core](https://learn.microsoft.com/en-us/ef/core/get-started/overview/first-app?tabs=netcore-cli)
* [Linq (Method Syntax)](https://canro91.github.io/2021/01/18/LinqGuide/)