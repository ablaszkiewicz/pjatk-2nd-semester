# 1.15.1

Opiszę przykłady zapytań, które dane zainteresowane strony mogłyby chcieć wykonać. Podziele to na poziomy:

- klienta,
- pracownika,
- zarządzającego.

## Poziom klienta

### Klient chce widzieć dostępne produkty i ich ceny

Brak potrzeby widoku

### Klient chce wiedzieć, kiedy kupił jakie produkty

```sql
CREATE VIEW Historia_Zakupow_Klienta AS
SELECT S.Data, P.Nazwa AS Nazwa_produktu, S.Ilość, P.Cena * S.Ilość AS Cena
FROM Sprzedaż S
JOIN Produkty P ON S.Id_produktu = P.Id_produktu
```

## Poziom pracownika

### Chce zobaczyć, kiedy i komu sprzedał jaki produkt

```sql
CREATE VIEW Historia_Sprzedaży_Pracownika AS
SELECT S.Data, K.Imie AS Imie_klienta, K.Nazwisko AS Nazwisko_klienta,
       P.Nazwa AS Nazwa_produktu, S.Ilość
FROM Sprzedaż S
JOIN Klienci K ON S.Id_klienta = K.Id_klienta
JOIN Produkty P ON S.Id_produktu = P.Id_produktu
```

### Chce zobaczyć, z jakimi pracownikami miał kontakt (np. w przypadku covid)

```sql
CREATE VIEW Kontakt_Klient_Pracownik AS
SELECT DISTINCT C.Id_prac, C.Imie AS Imie_klienta, C.Nazwisko AS Nazwisko_klienta,
       P.Imię AS Imię_pracownika, P.Nazwisko AS Nazwisko_pracownika
FROM Sprzedaż S
JOIN Klienci C ON S.Id_klienta = C.Id_klienta
JOIN Pracownicy P ON S.Id_prac = P.Id_prac;
```

## Poziom zarządu

### Chce zobaczyć, jakie produkty są najczęściej kupowane

```sql
CREATE VIEW Najczesciej_Kupowane_Produkty AS
SELECT P.Id_produktu, P.Nazwa AS Nazwa_produktu, COUNT(*) AS Liczba_Zakupow
FROM Sprzedaż S
JOIN Produkty P ON S.Id_produktu = P.Id_produktu
GROUP BY P.Id_produktu, P.Nazwa
ORDER BY COUNT(*) DESC;
```

### Chce zobaczyć, jaki pracownik ile sprzedał produktów w danych miesiącach

```sql
CREATE VIEW Sprzedane_Produkty_Pracownika_Miesiac AS
SELECT EXTRACT(MONTH FROM S.Data) AS Miesiac, EXTRACT(YEAR FROM S.Data) AS Rok, S.Id_prac
       P.Id_produktu, P.Nazwa AS Nazwa_produktu, SUM(S.Ilość) AS Ilość_Sprzedanych_Produktów
FROM Sprzedaż S
JOIN Produkty P ON S.Id_produktu = P.Id_produktu
GROUP BY EXTRACT(MONTH FROM S.Data), EXTRACT(YEAR FROM S.Data), P.Id_produktu, P.Nazwa;
```

### Chce zobaczyć, jak długo pracują dani pracownicy

```sql
CREATE VIEW Dni_Pracy_Pracowników AS
SELECT P.Id_prac, P.Imię AS Imię_pracownika, P.Nazwisko AS Nazwisko_pracownika,
       SUM(NVL(DATEDIFF(DAY, Z.Data_pocz, NVL(Z.Data_koniec, SYSDATE)), 0)) AS Dni_Pracy
FROM Pracownicy P
LEFT JOIN Zatrudnienie Z ON P.Id_prac = Z.Id_prac
GROUP BY P.Id_prac, P.Imię, P.Nazwisko;
```

# 1.15.2

Jako że w wykładzie opisane jest, że klaster łączy się poprzed powiązanie tabel wspólną kolumną, to widzę tutaj 2 klastry:

- Klienci-Sprzedaż - powiązane Id_klienta,
- Pracownicy-Zatrudnienie - powiązane Id_prac.
