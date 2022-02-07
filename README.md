# MySQL

<a href="https://vydb1.spsmb.cz/phpmyadmin/index.php">login</a>

Seskupování a souhrny
---

<a href="https://github.com/KRBNJSF/MySQL/blob/main/23_seskupovaniASouhrny.odp">Download: 23</a><br>
```SELECT skupina, datum, SUM(pocet) AS celkem FROM prodeje GROUP BY skupina, datum```<br>
```SELECT DISTINCT datum FROM prodeje GROUP BY datum```

```SELECT DISTINCT datum FROM prodeje```

Výběr sloupců v dotazu se seskupováním
---
```SELECT datum, SUM(pocet) AS celkem FROM prodeje GROUP BY datum, skupina```<br>

Agregační funkce
---
- Součet – SUM()
- Průměr – AVG()
- Minimum – MIN()
- Maximum – MAX()
- Počet řádků – COUNT()

<b>Example: <br></b>
```SELECT COUNT(*) AS celkem_radku FROM prodeje```

```SELECT COUNT(*), COUNT(obec), COUNT(krajske_mesto), COUNT(DISTINCT krajske_mesto) FROM obce```
*  * - všechny řádky
* obec - všechny řádky, které nejsou NULL

``` ```
Rozdíl mezi count(*) a count(sloupec)
---
`SELECT count(*), count(id) FROM obce GROUP BY krajske_mesto`<br>
<b>Pokud je v sloupci hodnota NULL, COUNT(sloupec) jí nezapočítává, COUNT(*) ano.</b>

Příklady
---
`SELECT DISTINCT os_cislo FROM hodiny`

`SELECT vyrobky.nazev, SUM(polozky.mnozstvi) FROM polozky INNER JOIN vyrobky ON vyrobky.cislo=polozky.cislo_vyrobku GROUP BY vyrobky.nazev WITH ROLLUP`

7.2. 2022
---
[23_seskupovaniASouhrny.pdf](https://github.com/KRBNJSF/MySQL/files/8015574/23_seskupovaniASouhrny.pdf)

slide 10

![image](https://user-images.githubusercontent.com/90755554/152794412-b2a2eed5-942b-454f-a334-911f0ece9e0b.png)

<b>ZOBRAZENÍ CELKOVÉHO SOUHRNU</b><b>
Seskupení podle jednoho sloupce:
```
SELECT vyrobky.nazev, SUM(polozky.mnozstvi)
FROM polozky INNER JOIN vyrobky ON
vyrobky.cislo=polozky.cislo_vyrobku GROUP BY
vyrobky.nazev WITH ROLLUP
```
Vylepšení o zobrazení slova celkem namísto NULL:
```
SELECT IFNULL(vyrobky.nazev, 'celkem'),
SUM(polozky.mnozstvi) FROM polozky INNER JOIN
vyrobky ON
vyrobky.cislo=polozky.cislo_vyrobku GROUP BY
vyrobky.nazev WITH ROLLUP
```
<b>ŘAZENÍ</b><br>
Při seskupování dle více sloupců se řazení provede
zleva doprava:
  ```
  SELECT skupina, datum FROM prodeje
GROUP BY skupina, datum
  ```
  Daný sloupec lze upřednostnit pomocí ORDER BY
na konci dotazu:
  ```
  SELECT skupina, datum FROM prodeje
GROUP BY skupina, datum ORDER BY datum
  ```
  <b>PŘEDBĚŽNÁ FILTRACE</b><br>
  Klauzule WHERE v dotaze se souhrnem znamená, že se
filtrace provede ještě dříve, než se začnou řádky seskupovat.
  
  Součet prodaných kusů a omezení výběru na leden 2018:
  ```
  SELECT skupina, SUM(pocet) AS celkem FROM
prodeje WHERE datum BETWEEN '2018-01-01' AND
'2018-01-31' GROUP BY skupina
  ```
  Součet řádků s hodnotou NULL:
  ```
  SELECT COUNT(*) FROM obce WHERE krajske_mesto
IS NULL;
  ```
  <b>NÁSLEDNÁ FILTRACE</b>
  Klauzuli HAVING používáme pro možnost
výběru řádků až po seskupení.
  
  Z tabulky prodejů provedeme sumaci po
datumech, ale ve výsledku zobrazíme jen dny,
kde se prodalo 10 a více ks:
```
SELECT datum, SUM(pocet) AS celkem
FROM prodeje GROUP BY datum HAVING
SUM(pocet) >=10;
 ```
  <b>WHERE VS HAVING</b>
  Předběžná filtrace (WHERE):
```
SELECT druh, SUM(pocet) AS celkem
FROM drubez WHERE pocet > 10 GROUP
BY druh;
```
Následná filtrace (HAVING):
  ```
SELECT druh, SUM(POCET) AS celkem
FROM drubez GROUP BY druh HAVING
SUM(pocet) > 10;
  ```
  
