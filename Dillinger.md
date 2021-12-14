# OpenAPI 3 és a Swagger.

## OpenAPI vs Swagger, mik is a különbségek?
2017-ben az OpenAPI 3.0 megjelenésé elég nagy mérföldkőnek számított. Ez volt az első hivatalos kiadás 2015 óta, amikor is a SmartBear Software az OpenAPI Intiative-nek ajándékozta a jogokat, illetve megtörtént a Swagger Specification -> OpenAPI Specification név váltás.

A különbség legegyszerűbben így érthető meg: 
- **OpenAPI** = specifikáció
- **Swagger** = eszközök a specifikáció megvalósításához

Az OpenAPI a specifikáció hivatalos neve. A specifikáció fejlesztését az OpenAPI Initiative segíti, amelyben több mint 30 szervezet vesz részt az IT világ különböző területeiről - köztük a Microsoft, a Google, az IBM és a CapitalOne. A Swagger eszközök fejlesztését vezető Smartbear Software szintén tagja az OpenAPI Initiative-nek, és segíti a specifikáció fejlődését.A Swagger eszköztár nyílt forráskódú, ingyenes és kereskedelmi eszközök keverékét tartalmazza, amelyek az API életciklus különböző szakaszaiban használhatók:

 - **Swagger Editor**: A Swagger Editor lehetővé teszi az OpenAPI-specifikációk YAML-ben történő szerkesztését a böngészőben, valamint a dokumentációk valós idejű előnézetét.
 - **Swagger UI**: A Swagger UI egy HTML, Javascript és CSS eszközökből álló gyűjtemény, amely dinamikusan gyönyörű dokumentációt generál egy OAS-kompatibilis API-ból.
 - **Swagger Codegen**: Lehetővé teszi az API klienskönyvtárak (SDK generálása), szerver csonkok és dokumentáció automatikus generálását egy OpenAPI Spec alapján.
 - **Swagger Parser**: Önálló könyvtár az OpenAPI definíciók Java-ból történő elemzésére.
 - **Swagger Core**: Java-alapú könyvtárak az OpenAPI-definíciók létrehozásához, fogyasztásához és az azokkal való munkához.
 - **Swagger Inspector** (ingyenes): API-tesztelő eszköz, amely lehetővé teszi az API-k validálását és OpenAPI-definíciók generálását egy meglévő API-ból.
 - **SwaggerHub** (ingyenes és kereskedelmi): API-tervezés és dokumentáció, az OpenAPI-val dolgozó csapatok számára készült.
 
Mélyedjünk el egy kicsit az OpenAPI 3.0 használatában.

 ## Swagger 2.0
 ### _Meta információk:_
 A metainformációk szakaszban adhatsz meg információkat az általános API-ról. Ebben a szakaszban olyan információkat lehet megadni, mint például, hogy mit csinál az API, mi az API alap URL címe és milyen webes protokollt követ.
```yaml
swagger: '2.0'
info:
  version: 1.0.0
  title: Simple Artist API
  description: A simple API to understand the Swagger Specification in greater detail

schemes:
    - https
host: example.io
basePath: /v1
/**/
```

 ### _Útvonal elemek (Path items)_
Az általunk készített API végpontjai. Az alap URL-hez képes relatív elérési úton érhetjük el őket, jelen esetben `/artist` útvonalon. Specifikálhatunk bennük HTTP igéket melyeket a végpontok használnak, ilyen például a `get`
```yaml
/**/
paths:
  /artists:
    get:
     description: Returns a list of artists 
/**/
```

 ### _Válaszok_
 A végpontokban definiálnunk kell a válaszokat amiket a kliens kaphat, ezeket a HTTP igék leírása alatt a `responses` tulajdonsághoz kell írnunk. Ehhez szükségünk van egy HTTP státusz kódra, illetve egy leírásra, mely a válasz sémáját ismerteti. 
 ```yaml
/**/
paths:
    /**/
    responses:
        200:
          description: Successfully returned a list of artists 
          schema:
            type: array
            items:
              type: object
              required:
               - username
              properties:
                artist_name:
                  type: string
                artist_genre:
                  type: string
                albums_recorded:
                  type: integer
                username:
                  type: string
        400:
          description: Invalid request 
          schema:
            type: object
            properties:   
              message:
                type: string
/**/
 ```
 
 ## _Paraméterek_
 Lehetőségünk van számtalan paraméter megadására, mellyel a kliensek és a szerver közötti kommunikáció specifikusságát tudjuk növelni. Ilyen paraméter például a lekérdezés paraméter (query parameter). Ahhoz, hogy egy ilyen :`GET http://example.com/v1/artists?limit=20&offset=3` kérést ki tudjunk szolgálni a fenti kódot az alábbi sorokkal kell bővítenünk:
 ```yaml
 /**/
    get:
    /**/
    parameters:
        - name: limit
          in: query
          type: integer
          description: Limits the number of items on a page
        - name: offset
          in: query
          type: integer
          description: Specifies the page number of the artists to be displayed 
/**/
 ```
 
 A lekérdező paramétereken túl még fontos a felhasználói jogkörök paraméterezése. Az alábbi példában egy olyan végpontot készítünk melyben az artist visszakapja a felhasználó nevéhez kapcsolódó adatokat : `/artists/{username}`
 ```yaml
 /artists/{username}:
    get:
      description: Obtain information about an artist from his or her unique username
      parameters:
        - name: username
          in: path 
          type: string
          required: true 
      responses:
        200:
          description: Successfully returned an artist
          schema:
            type: object
            properties:
              artist_name:
                type: string
              artist_genre:
                type: string
              albums_recorded:
                type: integer
        400:
          description: Invalid request
          schema:
            type: object 
            properties:           
              message:
                type: string
 ```

Példakódjaink egy alap API kezdetleges változatát állították elő. Jól látható, hogy ez a kód egy komolyabb API leírásnál hatalmas menyiséget is elérhet. Fontos tisztázni azt, hogy az API leírásához egy már logikailag összerakott adatbázist és az azt reprezentáló modellt kell alkotnunk így megkönnyítve a dolgunk. 

A Swagger számtalan lehetőséget nyújt a fejlesztők számára, a szabványos OpenAPI leírásokat figyelembe véve nagyjából egyezik is velük. Ezeket a leírásokat általában JSON vagy YAML nyelveken készítjük. A hivatalos források mind a YAML nyelvet ajánlják, ugyanis könnyebben olvasható, és gyorsabban megérthető. 

 ## Polimorfizmus és öröklés OpenAPI 3.0 alatt
Az OpenAPI specifikációja lehetőséget biztosít arra, hogy a közös tulajdonságok ne alkossanak sormintát, ezzel csúnyává téve az amúgy is elég nagy kódot. Nézzünk erre egy példát: 

```yaml
    components:
      schemas:
        BasicErrorModel:
          type: object
          required:
            - message
            - code
          properties:
            message:
              type: string
            code:
              type: integer
              minimum: 100
              maximum: 600
        ExtendedErrorModel:
          allOf:     # Combines the BasicErrorModel and the inline model
            - $ref: '#/components/schemas/BasicErrorModel'
            - type: object
              required:
                - rootCause
              properties:
                rootCause:
                  type: string
```

Itt a legfontosabb elem az `allOf` kulcsszó. Az `allOf` segít abban, hogy egy relatív elérési út megadásával egyesítsük az általunk készített leszármazott attribútumait az elérési út túloldalán álló model attribútumaival.  Van egy szabály mellyet ajánlott követnünk az `allOf` parancsnál, ez pedig nem más mint,hogy ne használjunk azonos attribútum neveket különböző adattípusokkal. Ha ilyen előfordul a kódban az számtalan hibához vezethet. 

A polimorfizmus megoldásához az `allOf`-hoz hasonló `oneOf` és `anyOf` kulcsszavakat tudjuk használni. 
```yaml
/**/
    components:
      responses:
        sampleObjectResponse:
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/simpleObject'
                  - $ref: '#/components/schemas/complexObject'
     /**/
    components:
      schemas:
        simpleObject:
          /**/
        complexObject:
          /**/
```
Használatát tekintve, a `oneOf` parancs lehetővé teszi, hogy a válaszban érkezett csomagok tartalmazhatnak `simpleObject` és `complexObject` objektumokat sémát is, de csak egyet. Az `anyOf` parancs ettől eltérően több objektum séma detektálását is bizotsítja.

A szakirodalom véleménye megoszlik a a polimorfizmusról mint lehetséges `edge-case`-ről. Az OpenAPI 3.0 kiadásával megjelentek a már fent említett parancsok, viszont még 2019-ben is számtalan olyan szolgáltatás volt mely nem támogatta ezek használatát. Ha tehetjük ne használjunk polimorfizmust az ilyen problémák elkerülése végett. 
