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
  
ŘAZENÍ
---
  
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
  ---
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
  ---
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
  ---
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
  
<b>KOMBINACE FILTRŮ</b>
  ---
  
 Pořadí:
- 1)Předběžná filtrace (WHERE),
- 2)seskupení (GROUP BY),
- 3)následná filtrace (HAVING).
  
Dotaz sečte prodané kusy pro obě skupiny výrobků, omezí výběr
na leden 2018 a ve výsledku zobrazí jen řádky, u kterých je
celkový součet 10 a více:
  
```
SELECT skupina, SUM(POCET) AS celkem FROM
prodeje WHERE datum BETWEEN '2018-01-01' AND
'2018-01-31' GROUP BY skupina HAVING SUM(pocet)
>= 10;  
```
<b>VÝPOČET PŘED AGREGACÍ</b>
  ---
  
  
Chceme znát cenu vyrobených součástek za
každý den.
  
Během jednoho dne se však na lince vyrábí
různé typy souč. s rozdílnou cenou.
  
Nejprve spočítáme cenu, poté sečteme:
```
SELECT datum, SUM(kusu*cena) AS
celkem FROM vyroba GROUP BY datum;
```
<b>VÝPOČET PO AGREGACÍ</b>
  ---
  
  
- Potřebujeme zjistit průmernou dobu potřebnou k výrobě
jedné součástky.
- Pro jednotlivé dny zjistíme celkový počet vyrobených
součástek a celkovou provozní dobu
- Obě hodnoty nakonec vydělíme a vynásobíme 60 pro
obdržení počtu sekund potřebných pro výrobu jedné
součástky:
  
```
SELECT datum, SUM(cas)/SUM(kusu) * 60 AS
podil FROM vyroba GROUP BY datum;
```

<b>VZOREC V GROUP BY</b>
  ---
  
  
Sečtení vyrobených kusů po měsících:
```
SELECT MONTH(datum) AS mesic, SUM(kusu) AS celkem
FROM vyroba GROUP BY MONTH(datum);  
```
Sečtení počtu vyrobených součástek podle skupin (skupina = první dva
znaky v kódu souč.):
```
SELECT LEFT(kod, 2) AS skupina, SUM(kusu) AS celkem
FROM vyroba GROUP BY LEFT(kod, 2);
```
Zjištění počtu obratů do 1000Kč a nad:
```
 SELECT CASE WHEN obrat<1000 THEN '<1000' ELSE
'>1000' END AS vyse_obratu, COUNT(obrat) AS pocet
FROM obraty GROUP BY CASE WHEN obrat<1000 THEN
'<1000' ELSE '>1000' END; 
```
  
<b>SOUHRNY Z VÍCE TABULEK</b>
  ---
  
  
Výpočet celkových dodaných množství pro jednotlivé výroby:
```
SELECT vyrobky.nazev, SUM(dodavky.mnozstvi)
AS soucet FROM vyrobky INNER JOIN dodavky ON
dodavky.cislo_vyrobku=vyrobky.cislo GROUP BY
vyrobky.nazev;
```
Dotaz, zjišťující, kolikrát byly dané výrobky prodány:
```
SELECT vyrobky.nazev, COUNT(dodavky.mnozstvi)
AS pocet1, COUNT(*) AS pocet2 FROM vyrobky
INNER JOIN dodavky ON
dodavky.cislo_vyrobku=vyrobky.cislo GROUP BY
vyrobky.nazev;
```
Zjištění data poslední dodávky:
```
SELECT vyrobky.nazev, MAX(dodavky.datum) AS
posledni FROM vyrobky INNER JOIN dodavky ON
dodavky.cislo_vyrobku = vyrobky.cislo GROUP
BY vyrobky.nazev;
```
Zobrazení úhrnných tržeb:
```
SELECT vyrobky.nazev,
SUM(vyrobky.cena*dodavky.mnozstvi) AS trzba
FROM vyrobky INNER JOIN dodavky ON
dodavky.cislo_vyrobku = vyrobky.cislo GROUP
BY vyrobky.nazev;
```
Úhrnná dodaná množství za rok 2017:
```
SELECT vyrobky.nazev, SUM(dodavky.mnozstvi) AS
celkem FROM vyrobky INNER JOIN dodavky ON
dodavky.cislo_vyrobku = vyrobky.cislo WHERE
dodavky.datum BETWEEN '2017-01-01' AND '2017-12-31'
GROUP BY vyrobky.nazev HAVING
SUM(dodavky.mnozstvi)>200;
```
Pět výrobků s nejvyššími tržbami:
```
SELECT vyrobky.nazev, ROUND(SUM(vyrobky.cena *
dodavky.mnozstvi),0) AS trzba FROM vyrobky INNER
JOIN dodavky ON dodavky.cislo_vyrobku =
vyrobky.cislo GROUP BY vyrobky.nazev ORDER BY trzba
DESC LIMIT 5;
```
Zobraz tržby včetně DPH pro jednotlivé zákazníky:
```
SELECT zakaznici.jmeno, SUM(vyrobky.cena + vyrobky.cena
* vyrobky.sazba_dph/100 * polozky.mnozstvi) AS trzba
FROM zakaznici
INNER JOIN faktury ON faktury.odberatel = zakaznici.id
INNER JOIN polozky ON polozky.faktura = faktury.cislo
INNER JOIN vyrobky on vyrobky.cislo =
polozky.cislo_vyrobku
GROUP BY zakaznici.jmeno
```

<b>VNĚJŠÍ SPOJENÍ TABULEK</b>
  ---
  
  
- Dotaz, počítající pro všechny učitele celkové úvazky:
- SELECT ucitele.jmeno, SUM(predmety.hodin) AS
celkem FROM ucitele LEFT OUTER JOIN predmety
ON ucitele.id = predmety.ucitel GROUP BY
ucitele.jmeno;
- Vylepšete, aby dotazmísto NULL ukazoval u
Ladislava 0.
- Zobrazte pomocí COUNT(), kolik předmětů jaký učitel
vyučuje.
  
<b>SPOJENÍ DOTAZŮ SE SOUHRNEM</b>
  ---
  
  
- Představme si tabulku prodejů, obsahující číslo měsíce, jméno
prodejce, výši tržby a další údaje.
- Spočítejme měsíční průměry prodejců zvlášť pro muže a zvlášť pro
ženy, kdy tabulka neobsahuje informaci o pohlaví prodejce.
```
SELECT mesic, AVG(trzba) AS prumer , 'muzi' AS typ
FROM data WHERE zastupce in ('Jaroslav', 'Petr',
'Marek') GROUp BY mesic
UNION
SELECT mesic, AVG(trzba) AS prumer , 'zeny' AS typ
FROM data WHERE zastupce in ('Jitka', 'Pavla',
'Marie') GROUp BY mesic
```
  
<b>Procvičení na konec</b>
  ---
  
  <details>
  <summary><b>1. SLIDE</b></summary>
  
- 1)Z tabulky obratů zjistěte počet řádků, součet a
průměr obratů a nejvyšší a nejmenší obrat. 
```
SELECT COUNT(*) as pocet, SUM(obrat) as soucet, AVG(obrat) as prumer, MAX(obrat), MIN(obrat) FROM obraty
```
- 2)Z tabulky faktur zjistěte počet zaplacených faktur
(vyplněno datum zaplacení).
```
SELECT COUNT(faktury.zaplaceno) FROM faktury WHERE faktury.zaplaceno IS NOT NULL
```
- 3)Z tabulky obratů vypočtěte součet obratů a počet
řádků pro jednotlivá čísla protiúčtů (sloupec „ucet“)
```
SELECT SUM(obrat), COUNT(obraty.ucet) from obraty GROUP by ucet
```
- 4)Z tabulky obratů zjistěte počet jednotlivých
protiúčtů.
```
SELECT COUNT(DISTINCT obraty.ucet) from obraty
  ```
- 5)Z tabulky obratů vypočítejte průměrný obrat pro
jednotlivá čísla protiúčtů a celkový průměr.
```
SELECT obraty.ucet, AVG(obraty.obrat) FROM obraty GROUP BY obraty.ucet WITH ROLLUP
```
  
  </details>

<details>
  <summary><b>2. SLIDE</b></summary>
  
1) Z tabulky položek vypočítejte součty částek pro jednotlivé
faktury a čísla výrobků, celkové součty částek pro každou
fakturu a součet všech částek v celé tabulce
```  
SELECT IFNULL(polozky.faktura, 'celkem'), polozky.cislo_vyrobku, SUM(polozky.castka) FROM polozky GROUP BY polozky.faktura, polozky.cislo_vyrobku WITH ROLLUP

SELECT IFNULL(polozky.faktura, 'celkem'), SUM(polozky.castka) FROM polozky GROUP BY polozky.faktura WITH ROLLUP
```
  
2) Z tabulky faktur vypočítejte součet částek podle čísla zákazníka.
Výpočet bude omezen jen na zaplacené faktury
```
SELECT faktury.odberatel, SUM(faktury.castka) FROM faktury WHERE faktury.zaplaceno IS NOT null GROUP BY faktury.odberatel
```
3) Z tabulky faktur vypočítejte součet částek podle čísla zákazníka.
Dotaz zobrazí pouze řádky se součtem nad 40 000Kč
```
SELECT faktury.odberatel, SUM(faktury.castka) AS soucet FROM faktury 
GROUP BY faktury.odberatel HAVING soucet > 40000 
```
4) Z tabulky faktur vypočítejte součet částek podle čísla zákazníka
za rok 2018. Dotaz zobrazí pouze řádky se součtem nad 20
000Kč
```
SELECT faktury.odberatel, SUM(faktury.castka) FROM faktury WHERE year(vystaveno) = 2018 GROUP BY faktury.odberatel HAVING SUM(faktury.castka) > 20000
```
5) Z tabulky položek spočítejte pro jednotlivé faktury součet podílů
částka/množství
```
SELECT polozky.faktura, SUM(polozky.castka /polozky.mnozstvi) FROM polozky GROUP BY polozky.faktura
```
  </details>

28.02. 2022  
---
  
 <details>
  <summary><b>1. SLIDE</b></summary>
  
1) Z tabulky obraty4 po jednotlivých protiúčtech
(sloupec účet) zjistěte podíl součtu poplatků a
součet obratů v procentech.
```
SELECT obraty4.ucet, SUM(obraty4.poplatek) / SUM(obraty4.obrat) * 100 FROM obraty4 GROUP BY ucet
```
2) Z tabulky faktur určete počet neproplacených faktur.
```
SELECT COUNT(*) FROM faktury WHERE faktury.zaplaceno IS NULL
```
3) Z tabulky obratů vypočtěte součet obratů pro
jednotlivé banky.
```
SELECT RIGHT(obraty.ucet, 4) as banka, SUM(obraty.obrat) FROM obraty GROUP BY banka
```
4) Z tabulky obratů vypočtěte součet obratů po rocích a
měsících.
```
SELECT YEAR(obraty.datum), MONTH(obraty.datum), SUM(obraty.obrat) FROM obraty GROUP BY YEAR(obraty.datum)
  ```
5) Z tabulky obratů vypočtěte součet a počet obratů do
10 000Kč a nad tuto částku.
```
SELECT CASE WHEN obraty.obrat > 10000 THEN ">10000" ELSE "<10000" END AS LIM, SUM(obraty.obrat) FROM obraty GROUP BY LIM
```
 </details>
  
<details>
<summary><b>2. SLIDE</b></summary>
  
1) Zobrazte názvy skupin, počet výrobků v každé skupině a
průměrnou cenu výrobků ve skupině.
  
```
SELECT skupiny.nazev, COUNT(*), AVG(cena) FROM vyrobky INNER JOIN skupiny ON skupiny.cislo = vyrobky.skupina GROUP BY skupiny.nazev
```
  
2) Pro jednotlivá jména zákazníků zobrazte celkové
nafakturované částky.
```
----
```
3) Pro jednotlivá jména zákazníků zobrazte datum poslední
fakturace.
```
SELECT zakaznici.jmeno, SUM(faktury.castka),MAX(faktury.zaplaceno) FROM faktury INNER JOIN zakaznici ON zakaznici.id = faktury.odberatel GROUP BY zakaznici.jmeno
```
4) Pro jednotlivé názvy výrobků zobrazte počet jejich prodejů,
které jsou vyšší, než 200 jednotek.
```
SELECT vyrobky.nazev, COUNT(*) FROM vyrobky JOIN polozky ON polozky.cislo_vyrobku = vyrobky.cislo WHERE polozky.mnozstvi > 200 GROUP BY vyrobky.nazev
```
 
5) Zobrazte názvy výrobků, pro které je celková fakturovaná
částka v tabulce položek vyšší, než 10 000 Kč.
```
SELECT vyrobky.nazev, SUM(polozky.castka) FROM vyrobky INNER JOIN polozky ON polozky.cislo_vyrobku = vyrobky.cislo GROUP BY vyrobky.nazev HAVING SUM(polozky.castka) > 10000
```
6) Zobrazte názvy skupin a rozdíl mezi nejvyšší a nejnižší cenou
u výrobků ve skupině.
```
SELECT skupiny.nazev, MAX(cena) - MIN(cena) FROM vyrobky INNER JOIN skupiny ON skupiny.cislo = vyrobky.skupina GROUP BY skupiny.nazev
```
  
  </details>
  
  <details>
  <summary><b>3. SLIDE</b></summary>
  
  1) Zobrazte jména jednotlivých zákazníků, názvy výrobků a
uhrnná prodaná množství pro jednotlivé zákazníky a výrobky.
  ```
  SELECT zakaznici.jmeno, vyrobky.nazev, SUM(polozky.mnozstvi)
  ```
  2) Určete celkový počet předpisů, pro které neexistuje příslušná
platba.
  ```
  SELECT COUNT(*) FROM predpisy LEFT JOIN platby ON predpisy.vs = platby.vs WHERE platby.id IS NULL;
  ```
   3) Zobrazte jména všech výrobků a jejich úhrnná prodaná
množství.
  ```
  SELECT vyrobky.
  ```
  4)
  ```
  
  ```
  5)
  ```
  
  ```
  6) Pro jednotlivé zákazníky zobrazte celkové nafakturované
částky, zvlášť pro neproplacené a zvlášť pro proplacené
faktury.
  ```
SELECT zakaznici.jmeno, CASE WHEN faktury.zaplaceno IS NULL THEN 'Nezaplaceno' ELSE 'Zaplaceno' END AS zap_nezap, SUM(faktury.castka) FROM zakaznici INNER JOIN faktury ON faktury.odberatel = zakaznici.id GROUP BY zakaznici.jmeno, zap_nezap
  ```
  
  </details>

  
  7.03. 2022
  ---
  
  <a href="https://github.com/KRBNJSF/MySQL/raw/main/25_poddotazy.odp">Download 25</a>
  
### PODDOTAZY
- Příkazy SELECT lze vnořovat do sebe a vytvářet tak jejich hierarchii. Vnořené příkazy SELECT označujame jako poddotazy.
- Příklad: Zobrazte čísla dodaných výrobků, kde je vyplněno datum dodání a dodané množství je vyšší, než 200:
1. krok: ```SELECT * FROM dodavky WHERE NOT datum IS NULL```
2. krok: ```SELECT cislo_vyrobku FROM (SELECT * FROM dodavky WHERE NOT datum IS NULL) AS poddotaz WHERE mnozstvi>200```
  
```
SELECT cislo_vyrobku FROM (SELECT * FROM dodavky WHERE datum IS NOT NULL) AS poddotaz WHERE mnozstvi < 200  
```
  
Vnořený příkaz SELECT se vždy zapisuje do závorky a většinou i označuje pomocí aliasu
  
Vnoření příkazů SELECT do sebe může být několikanásobné.
  
Zpracování probíhá „zdola nahoru“ tj. Nejprve se provede příkaz SELECT vnořený nejhlouběji.
  
  <b>Typy poddotazů:</b>
- Skalární (vrací jedinou hodnotu)
- Sloupcový (vrací sloupec hodnot)
- Tabulkový (vrací tabulku hodnot)
  
### SKALÁRNÍ PODDOTAZY
  
Příklad: napište dotaz, který zobrazí datum, jméno prodejce, tržbu a pro srovnání také průměrnou hodnotu tržby za celou tabulku:
  
```
SELECT datum, prodejce, trzba, (SELECT AVG(trzba) FROM trzby) AS prumer FROM trzby
```
Místo průměru použijte procentuální podíl jednotlivých tržeb na jejich celkovém součtu:

```
SELECT datum, prodejce, trzba, trzba/(SELECT SUM(trzba) FROM trzby)*100 AS podil FROM trzby 
```
Pro jednotlivé prodejce:
```
SELECT prodejce, SUM(trzba), SUM(trzba)/(SELECT SUM(trzba) FROM trzby)*100 AS podil FROM trzby GROUP BY prodejce;
```
  
Poddotaz nemusí být v GROUP BY, vyrábíme tak agregační funkci:
```
SELECT prodejce, SUM(trzba) AS soucet, (SELECT SUM(trzba) FROM trzby) AS celkem FROM trzby GROUP BY prodejce;
```
  
### SKALÁRNÍ PODDOTAZ V KLAUZULI WEHERE

Zobrazení nadprůměrných tržeb:
```
SELECT datum, prodejce, trzba FROM trzby WHERE trzba > (SELECT AVG(trzba) FROM trzby);  
```
Zobrazení tržeb, které dosahují alespoň 10% celkového objemu tržeb:
```
SELECT datum, prodejce, trzba FROM trzby WHERE trzba >= (SELECT SUM(trzba) FROM trzby)*0.1
```

### MIN A MAX VE SKALÁRNÍM PODDOTAZU
  
V případě stejných maxim bychom potřebovali vypsat všechna. Takto to nelze:
```
SELECT * FROM trzby ORDER BY trzba DESC LIMIT 1;
```
Takto ano:
```
SELECT * FROM trzby WHERE trzba = (SELECT MAX(trzba) FROM trzby);
```

### SKALÁRNÍ PODDOTAZ V KLAUZULI HAVING
  
Zobrazení součtů tržeb po dnech tam, kde je tržba větší, než celkový průměr tržeb:
```
SELECT datum, SUM(trzba) AS soucet FROM trzby GROUP BY datum HAVING SUM(trzba) > (SELECT AVG(trzba) FROM trzby;
```
Zobrazení průměrných tržeb za únor, pokud jsou vyšší, než je průměr za leden:
```
SELECT prodejce, AVG(trzba) AS prumer FROM trzby WHERE MONTH(datum) = 2  GROUP BY prodejce HAVING AVG(trzba) > (SELECT AVG(trzba) FROM trzby WHERE MONTH(datum)=1);
```
### DÍLČÍ SOUHRN VZHLEDEM K HODNOTĚ VYBR. SLOUPCE
  
Zobrazte prodejce, datum, tržbu a pro každého prodejce součet jeho tržeb:
```
SELECT prodejce, datum, trzba, (SELECT SUM(trzba) FROM trzby AS trzby1 WHERE trzby1.prodejce = trzby.prodejce) AS celkem 
FROM trzby
```
Totéž s INNER JOIN:
```
SELECT trzby.prodejce, trzby.datum, trzby.trzba, souhrn_trzeb.soucet
FROM trzby
INNER JOIN (SELECT prodejce, SUM(trzba) AS soucet FROM trzby GROUP BY prodejce) AS souhrn_trzeb ON trzby.prodejce=souhrn_trzeb.prodejce;
```
  
### PODDOTAZ V OPERÁTORU IN

Z tabulky tržeb zobrazte řádky u kterých se datum nachází v tabulce datumy:

- Vnitřní spojení způsobí duplicitu:
```
SELECT trzby.datum, trzby.prodejce, trzby.trzba FROM trzby INNER JOIN datumy ON trzby.datum = datumy.datum;
```
- Kromě použití DISTINCT ještě můžeme udělat:
```
SELECT trzby.datum, trzby.prodejce, trzby.trzba FROM trzby WHERE datum IN (SELECT datum FROM datumy);
``` 
Zobrazte úhrnné prodeje pro jednotlivé prodejce a součty omezte na dny, zapsané v tabulce datumů:
```
SELECT prodejce, SUM(trzba) AS celkem FROM trzby WHERE datum IN (SELECT datum FROM datumy) GROUP BY prodejce;
```
Totéž s omezením na leden:
```
SELECT prodejce, SUM(trzba) AS celkem FROM trzby WHERE datum IN (SELECT datum FROM datumy WHERE MONTH(datum) = 1) GROUP BY prodejce;
```
  
### PODDOTAZ IN JAKO VNĚJŠÍ SPOJENÍ

Z tabulky tržeb zobrazte řádky u kterých se datum nenachází v tabulce datumy:
 
- Pomocí LEFT OUTER JOIN:
```
SELECT trzby.datum, trzby.prodejce, trzby.trzba FROM trzby LEFT OUTER JOIN datumy ON trzby.datum = datumy.datum WHERE datumy.datum IS NULL;
```
- Pomocí poddotazu:
```
SELECT trzby.datum, trzby.prodejce, trzby.trzba FROM trzby WHERE NOT datum IN (SELECT datum FROM datumy);
```
  
### PODDOTAZ V OPERÁTORU IN

- Poddotazy jsou efektivní, pokud vnořený SELECT dává výrayně menší počet záznamů oproti v
- Vnější tabulce.

Seskupování v operátoru IN – zobrazte z tabulky datumů ty, kde je v tabulce tržeb součet jednotlivých tržeb větší, než 10000Kč:
```
SELECT datum FROM datumy WHERE datum IN(SELECT datum FROM trzby GROUP BY datum HAVING SUM(trzba)>10000);
```
  
### PODDOTAZ V SEZNAMU ALL/ANY
  
IN – hodnota ve sloupci se musí rovnat alespoň jedné hodnotě v seznamu
  
ALL – hodnota ve sloupci musí být větší/menší, než všechny hodnoty v seznamu:
```
SELECT castka FROM vydani WHERE castka > ALL(SELECT trzba FROM trzby);
```
ANY(SOME) -  hodnota ve sloupci musí být větší/menší, než alespoň jedna hodnota v seznamu:
```
SELECT castka FROM vydani WHERE castka > ANY(SELECT trzba FROM trzby);
```
Výběr hodnot, které leží uprostřed intervalu, definovaného nejmenší a největší hodnotou v seznamu:
```
SELECT * FROM vydani WHERE castka >ANY(SELECT trzba FROM trzby) AND castka <ANY(SELECT trzba FROM trzby);
```
Zobrazení řádků z tabulky tržeb pro Jitku, u kterých je tržba větší, než alespoň jedna tržba ostatních prodejců:
```
SELECT datum, prodejce, trzba FROM trzby WHERE prodejce = 'Jitka' AND trzba > ANY(SELECT trzba FROM trzby WHERE prodejce <> 'Jitka');
```
Zobrazení řádků pro Karla, pro které platí, že tržba je větší, než alespoň jedna průměrná tržba u ostatních prodejců:
```
SELECT datum, prodejce, trzba FROM trzby WHERE prodejce = 'Karel' AND trzba > ANY(SELECT AVG(trzba) FROM trzby WHERE prodejce <> 'Karel' GROUP BY prodejce);
```
Pokud by to mělo být větší, než průměrná tržba u všech ostatních prodejců, místo ANY dáme ALL.

### TABULKOVÉ PODDOTAZY

Tabulkové poddotazy reprezentují další tabulku. Ta se dá běžným způsobem (výběr dat spojení) dále používat.
  
Zobrazení naposledy provedené akce každé obce:
```
SELECT akce2.obec, akce2.cislo, akce2.dokonceno FROM akce2 INNER JOIN (SELECT obec, MAX(dokonceno) AS datum FROM akce2 GROUP BY obec) AS vyber ON akce2.obec=vyber.obec AND akce2.dokonceno=vyber.datum;  
```
Zobrazení výrobků s nejvyšší cenou každé skupiny. Poddotaz vybere z tabulky výrobků nejvyšší cenu pro každou skupinu a tento výběr se spojí s tabulkou skupin a tabulkou výrobků:
```
SELECT skupiny.nazev, vyrobky.nazev, vyber.maximum FROM skupiny INNER JOIN vyrobky ON skupiny.cislo = vyrobky.skupina INNER JOIN (SELECT skupina, MAX(cena) AS maximum FROM vyrobky GROUP BY skupina) AS vyber ON vyrobky.skupina = vyber.skupina AND vyrobky.cena = vyber.maximum;
```
Zobrazení součtů dodávek jednotlivých výrobků, kde dodané množství je NULL:
```
SELECT vyrobky.nazev, IFNULL(podvyber.pocet, 0) AS pocet FROM vyrobky LEFT OUTER JOIN (SELECT polozky.cislo_vyrobku, COUNT(*) AS pocet FROM polozky WHERE polozky.mnozstvi IS NULL GROUP BY cislo_vyrobku) AS podvyber ON vyrobky.cislo=podvyber.cislo_vyrobku;
```
Zobrazení dvou řádků z tabulky výrobků s nejvyšší cenou u výrobků ze skladu č. 1 a stejně tak u výrobků ze skladu č. 2:
```
SELECT * FROM (SELECT * FROM vyrobky WHERE sklad = 1 LIMIT 2) AS sklad1 UNION SELECT * FROM (SELECT * FROM vyrobky WHERE sklad = 2 LIMIT 2) AS sklad2 ORDER BY cena DESC
```
  
### OPERÁTOR EXISTS
  
Operátor EXISTS testuje, zdali se v jedné tabulce vyskytují řádky, které hodnotami v některém sloupci odpovídají řádkům v druhé tabulce. Obecná syntaxe:
```
SELECT <seznam_polí> FROM Tabulka1 WHERE EXISTS (SELECT <seznam_polí> FROM Tabulka 2 WHERE <spojovací_podmínka>).
```
Zobrazení názvů výrobků, jejichž čísla se vyskytují v tabulce dodávek:
```
SELECT vyrobky.nazev FROM vyrobky WHERE EXISTS (SELECT * FROM dodavky WHERE dodavky.cislo_vyrobku = vyrobky.cislo);
```
Zobrazení názvů výrobků, jejichž čísla se vyskytují v tabulce dodávek a současně alespoň jedno jejich prodané množství bylo přes 200ks:
```
SELECT vyrobky.nazev FROM vyrobky WHERE EXISTS (SELECT * FROM dodavky WHERE dodavky.cislo_vyrobku = vyrobky.cislo AND dodavky.mnozstvi>200);
```
Operátor EXISTS lze zpravidla nahradit jinými prostředky SQL. Jeho nevýhoda je v nízké efektivitě. Vnitřní příkaz SELECT se musí opakovaně vyhodnocovat pro každý řádek z první tabulky.
  
  
<a href="https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html">MySql operators</a>
