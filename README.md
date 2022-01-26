# MySQL


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
